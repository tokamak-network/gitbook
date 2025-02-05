# Contracts

The [@tokamak-network/thanos-contracts](https://www.npmjs.com/package/@tokamak-network/thanos-contracts) package provides the smart contracts for the Thanos network, focusing on Ethereum scalability and cross-chain communication. It includes contracts for:

* **Cross-Domain Messaging**: Facilitating seamless data and function calls between Ethereum (L1) and Thanos (L2).
* **Token Bridging**: Enabling deposits and withdrawals of ETH and ERC-20 tokens across L1 and L2.
* **Protocol Infrastructure**: Managing transaction batches, state commitments, and fraud-proof mechanisms for Thanos’s rollup architecture.

***

### **1. Cross-Domain Messaging**

**Cross-domain messaging** is a core feature of Thanos, enabling communication between Ethereum(L1) and Thanos(L2). This mechanism is crucial for various use cases, such as transferring data, triggering actions across chains, and enabling interoperability between contracts on different layers.

#### **How Cross-Domain Messaging works**

Thanos employs Cross-Domain Messengers to facilitate messages passing between L1 and L2:

* [**L1CrossDomainMessenger**](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/contracts-bedrock/src/L1/L1CrossDomainMessenger.sol): Handles sending and receiving messages from Ethereum
* [**L2CrossDomainMessenger**](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/contracts-bedrock/src/L2/L2CrossDomainMessenger.sol): Handles sending and receiving messages from Thanos

#### **Process flow:**

a. Send messages from L1 to L2

* Message submission(L1 to L2):
  * A user or contract on L1 sends a message to the `L1CrossDomainMessenger`. The message is sent to the portal contract and stored on it
* Message relay(on L2):
  * The relayer processes the message on (1) and submits it to the `L2CrossDomainMessenger`
  * The message is executed on L2, triggering a function call or data transfer.

b. Send messages from L2 to L1

* Message submission(L2 to L1):
  * A user or contract on L2 sends a message to the `L2CrossDomainMessenger`. The message from L2 to L1 is initially stored in the L2 state.
* Message relay(on L1)
  * When the message is passed on the challenge period, we can prove and finalize this message manually and this message is relayed on `L1CrossDomainMessenger`

#### **Key Methods**

* `sendMessage`: Sends a message to some target address on the other chain
* `relayMessage`: Relays a message that was sent by the other CrossDomainMessenger contract. It can only be executed via cross-chain call from the other messenger OR if the message was received once and is being replayed.

#### **Example**

This example demonstrates triggering a function call on an L2 contract from L1:

* Setup
  * Assume we have an L2 contract ad `0xL2ContractAddress` with the function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyContract {
    uint256 private value;

    event ValueUpdated(uint256 oldValue, uint256 newValue);

    function setValue(uint256 newValue) external {
        uint256 oldValue = value;
        value = newValue;
        emit ValueUpdated(oldValue, newValue);
    }

    function getValue() external view returns (uint256) {
        return value;
    }
}

```

* Execute the transaction on L1

```tsx
contract MyContract {
    address public crossDomainMessenger;

    constructor(address _crossDomainMessenger) {
        crossDomainMessenger = _crossDomainMessenger;
    }

    function increaseValue(address myOptimisticContractAddress, uint256 myFunctionParam) public {
        ICrossDomainMessenger(crossDomainMessenger).sendMessage(
            myOptimisticContractAddress,
            abi.encodeWithSelector(
                IMyOptimisticContract.setValue.selector,
                myFunctionParam
            ),
            1000000 // Specify the gas limit
        );
    }
}
```

* The `setValue` function is called on the target L2 contract with `newValue = 42`.

***

### **2. Token Bridging**

Token bridging is a critical feature of the Thanos ecosystem that enables the transfer of tokens between Layer 1 and Layer 2. This feature allows users and developers to move assets like ETH or ERC-20 tokens to Thanos or withdraw them back to Ethereum.

#### **How Token Bridging Works**

The bridging mechanism uses two main contracts on both L1 and L2:

* [**L1StandardBridge**](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/contracts-bedrock/src/L1/L1StandardBridge.sol): Manages deposits from L1 to L2.
* [**L2StandardBridge**](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/contracts-bedrock/src/L2/L2StandardBridge.sol): Manages withdrawals from L2 to L1.

**Deposit (L1 to L2)**

* Tokens are locked in the **L1StandardBridge** contract on Ethereum.
* A corresponding amount of tokens is minted or unlocked on Optimism by the **L2StandardBridge** contract.

**Withdrawal (L2 to L1)**

* Tokens are burned or locked in the **L2StandardBridge** contract on Optimism.
* A corresponding amount is unlocked or released on Ethereum by the **L1StandardBridge** contract after a challenge period.

#### **Supported Token Types**

* ETH
* Standard ERC-20 Tokens
* Native token(is set when deploying the Thanos network)

#### Key Methods

**On L1StandardBridge contract**

* `bridgeETH`: Deposits ETH from L1 to L2.
* `bridgeERC20`: Deposit standard tokens from L1 to L2
* `bridgeNativeToken`: Deposit the native token from L1 to L2

**On L2StandardBridge contract**

* `withdraw`: withdraw tokens from L2 to L1.

#### Example

**Deposit ETH to Thanos**

You can use [Thanos SDK](https://www.npmjs.com/package/@tokamak-network/thanos-sdk?activeTab=code) to deposit ETH from Ethereum to Thanos

```tsx
const l1RpcProvider = new ethers.providers.JsonRpcProvider(l1Rpc)
const l2RpcProvider = new ethers.providers.JsonRpcProvider(l2Rpc)
const {chainId: l1ChainId} = await l1RpcProvider.getNetwork()
const {chainId: l2ChainId} = await l2RpcProvider.getNetwork()

const crossChainMessenger = new thanosSDK.CrossChainMessenger({
    bedrock: true,
    l1ChainId: l1ChainId,
    l2ChainId: l2ChainId, 
    l1SignerOrProvider: l1Signer,
    l2SignerOrProvider: l2Signer,
})

const response = await crossChainMessenger.bridgeETH(depositAmount)
await response.wait()

await crossChainMessenger.waitForMessageStatus(response, thanosSDK.MessageStatus.RELAYED)

```

**Withdraw ETH to Ethereum**

To withdraw ETH from Thanos to Ethereum, you can use the Thanos SDK

```tsx
const response = await crossChainMessenger.withdrawETH(withdrawAmount)
const withdrawalTx = await response.wait()

```

Withdrawals from L2 to L1 are subject to a waiting period (typically 7 days) to allow for fraud proofs in Thanos’s optimistic rollup architecture. After that, users can prove and finalize the withdrawal transactions on L1 to receive their tokens.

```tsx
await crossChainMessenger.waitForMessageStatus(response, thanosSDK.MessageStatus.READY_TO_PROVE)

const proveTx = await crossChainMessenger.proveMessage(withdrawalTx)
const proveReceipt = await proveTx.wait(3)

const finalizeInterval = setInterval(async () => {
  const currentStatus = await crossChainMessenger.getMessageStatus(withdrawalTx)
  console.log('Message status:', currentStatus)
}, 3000)

try {
  await crossChainMessenger.waitForMessageStatus(
    withdrawalTx,
    thanosSDK.MessageStatus.READY_FOR_RELAY
  )
} finally {
  clearInterval(finalizeInterval)
}

await crossChainMessenger.finalizeMessage(withdrawalTx)

await crossChainMessenger.waitForMessageStatus(response,
  thanosSDK.MessageStatus.RELAYED)
```

***

### **3. Sequencer and Rollups**

Thanos uses an off-chain sequencer to batch transactions and submit them to Ethereum as a single rollup. This improves throughput and reduces costs while maintaining Ethereum's security guarantees.

#### **Key Concepts:**

* **Transaction Batching:** The sequencer groups transactions and creates rollup batches.
* **Fraud Proofs:** Thanos initially relied on fraud proofs but transitioned to an "Optimistic" assumption with fault-proof mechanisms being phased out.

#### **Key Contracts:**

* [**L2OutputOracle**](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/contracts-bedrock/src/L1/L2OutputOracle.sol): The proposer's role is to construct and submit output roots, which are commitments to the L2's state, to the `L2OutputOracle` contract on L1 (the settlement layer)
* [**OptimismPortal**](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/contracts-bedrock/src/L1/OptimismPortal2.sol)**:** This contract is used to prove and finalize the withdrawal transactions
