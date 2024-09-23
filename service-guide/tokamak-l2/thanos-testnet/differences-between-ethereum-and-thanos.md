# Differences Between Ethereum and Thanos

Thanos is designed to be equivalent to the EVM and introduces as few changes to the Ethereum protocol as possible. However, there are some key differences in the way Ethereum and Thanos work that developers should be aware of.

## Opcodes

| Opcode       | Solidity Equivalent | Behavior                                                                                                                                                                                                                     |
| ------------ | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `COINBASE`   | `block.coinbase`    | It returns the fee wallet address of the Sequencer. Although it is essentially the same as Ethereum, the value generally does not change from block to block.                                                                |
| `PREVRANDAO` | `block.prevrandao`  | Unlike the strong guarantees provided by [RANDAO](https://eips.ethereum.org/EIPS/eip-4399) in Ethereum, the Sequencer sets a pseudo-random number for each block.                                                            |
| `ORIGIN`     | `tx.origin`         | If the transaction is an L1 ⇒ L2 transaction initiated by a Smart Contract on L1, it is set to the Address Alias of the address (tx.origin) that initiated the L1 ⇒ L2 transaction. Otherwise, this command works as normal. |
| `CALLER`     | `msg.sender`        | If the transaction is an L1 ⇒ L2 transaction initiated by a Smart Contract on L1, and it is the first Contract called on L2, the Address Alias is applied.                                                                   |

## Address Aliasing

{% hint style="info" %}
Address Aliasing is an important security feature that affects the behaviour of transactions sent from L1 to L2 in Smart Contracts. If you are using cross-chain transactions, please read this section carefully. Note that the CrossChainMessenger handles address aliasing internally.
{% endhint %}

When a transaction is sent from L1 to L2 using EOA, the sender address on L2 is set to the sender address on L1. However, if the transaction is triggered by a smart contract on L1, the sender address on L2 will be different.

Due to the behaviour of the `CREATE` opcode, the same address can be shared on both L1 and L2, but the bytecode of the smart contracts can be different. These contracts share the same address but are essentially two different smart contracts and cannot be treated as the same. As a result, the sender of a transaction sent from L1 to L2 by a smart contract cannot be the address of the L1 smart contract, and the L1 smart contract can behave as if it were the L2 smart contract (because the two contracts share the same address).

To prevent this type of impersonation, when a transaction is sent from L1 to L2 by a smart contract, the sender of the transaction is slightly modified. Instead of appearing to originate from the actual L1 contract address, the L2 transaction appears to be sent from an 'alias' version of the L1 contract address. This alias address is derived by adding a fixed offset to the real L1 contract address, ensuring that it doesn't collide with other addresses on L2, and that the original L1 address can be easily recovered from the alias address.

This change to the sender address only applies to L2 transactions sent by L1 smart contracts. In all other cases, the sender address of the transaction is set according to the same rules used in Ethereum.

| Transaction Source                                      | Sender Address                                                     |
| ------------------------------------------------------- | ------------------------------------------------------------------ |
| L2 user (Externally Owned Account)                      | The user's address (same as in Ethereum)                           |
| L1 user (Externally Owned Account)                      | The user's address (same as in Ethereum)                           |
| L1 contract (using `OptimismPortal.depositTransaction`) | `L1_contract_address + 0x1111000000000000000000000000000000001111` |

## Transactions

#### Transaction Fees

In Thanos, transactions must pay an `L1 Data Fee` in addition to the standard `Execution Gas Fee` expected in Ethereum. For more details, refer to the [Transaction Fee](transaction-fee.md) guide.

#### EIP-1559 Parameters

Thanos calculates its base fee using [EIP-1559](https://notes.ethereum.org/@vbuterin/eip-1559-faq), just like Ethereum. However, the EIP-1559 parameters used in Thanos differ from those used in Ethereum.

<table><thead><tr><th width="266">Parameter</th><th width="204">Thanos value</th><th>Ethereum value (for reference)</th></tr></thead><tbody><tr><td>Block gas limit</td><td>30,000,000 gas</td><td>30,000,000 gas</td></tr><tr><td>Block gas target</td><td>5,000,000 gas</td><td>15,000,000 gas</td></tr><tr><td>EIP-1559 elasticity multiplier</td><td>6</td><td>2</td></tr><tr><td>EIP-1559 denominator</td><td>250</td><td>8</td></tr><tr><td>Maximum base fee increase (per block)</td><td>2%</td><td>12.5%</td></tr><tr><td>Maximum base fee decrease (per block)</td><td>0.4%</td><td>12.5%</td></tr><tr><td>Block time in seconds</td><td>12</td><td>12</td></tr></tbody></table>

## Mempool Rules

Unlike Ethereum, Thanos does not have a public mempool. The Thanos mempool is only visible to the Sequencer, which executes transactions in order of priority fee (highest fee first).
