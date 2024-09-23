# Bridging L1 and L2 (Portal)

In Thanos, the Portal supports asset transfers between networks by using the `OptimismPortal` contract on L1 and the `L2ToL1MessagePasser` contract on L2. However, assets that can be transferred through the Portal are limited to `NativeToken`, and it handles transactions with lower gas fees compared to using `CrossChainMessenger`.

This document explores the functions of the Portal used for deposits and withdrawals and provides a step-by-step guide on how to import and use the Portal to transfer `NativeToken` between L1 and L2.

You can find detailed code on the Portal [here](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/sdk/src/portals.ts). Examples of how to use the Thanos SDK’s Portal for depositing and withdrawing NativeTokens between L1 and L2 can be found [here](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/sdk/tasks/portals.ts).

## Constructor

The constructor sets up the necessary information for initializing a `CrossChainProvider` instance. Each constructor is composed of essential elements to facilitate smooth Cross-Chain interactions between L1 and L2.

* `l1SignerOrProvider`: Signer or Provider for the L1 chain, or a JSON-RPC url
* `l2SignerOrProvider`: Signer or Provider for the L2 chain, or a JSON-RPC url
* `l1ChainId`: Chain ID for the L1 chain
* `l2ChainId`: Chain ID for the L2 chain
* `depositConfirmationBlocks`: Optional number of blocks before a deposit is confirmed
* `l1BlockTimeSeconds`: Optional estimated block time in seconds for the L1 chain
* `contracts`: Optional contract address overrides

```typescript
/**
   * Creates a new CrossChainProvider instance.
   *
   * @param opts Options for the provider.
   * @param opts.l1SignerOrProvider Signer or Provider for the L1 chain, or a JSON-RPC url.
   * @param opts.l2SignerOrProvider Signer or Provider for the L2 chain, or a JSON-RPC url.
   * @param opts.l1ChainId Chain ID for the L1 chain.
   * @param opts.l2ChainId Chain ID for the L2 chain.
   * @param opts.depositConfirmationBlocks Optional number of blocks before a deposit is confirmed.
   * @param opts.l1BlockTimeSeconds Optional estimated block time in seconds for the L1 chain.
   * @param opts.contracts Optional contract address overrides.
   */
  constructor(opts: {
    l1SignerOrProvider: SignerOrProviderLike
    l2SignerOrProvider: SignerOrProviderLike
    l1ChainId: NumberLike
    l2ChainId: NumberLike
    depositConfirmationBlocks?: NumberLike
    l1BlockTimeSeconds?: NumberLike
    contracts?: DeepPartial<OEContractsLike>
  })
```

## Set Up

Import the `ethers` library for interacting with the Ethereum blockchain and the `@tokamak-network/thanos-sdk` package. Additionally, create an instance for L1 and L2 interactions using the Portal's constructor, and prepare to use the various functions provided by the Portal.

```typescript
import { ethers } from 'ethers'
import { Portals } from '@tokamak-network/thanos-sdk'

const l1Provider = new ethers.providers.StaticJsonRpcProvider(
  process.env.L1_URL
)
const l2Provider = new ethers.providers.StaticJsonRpcProvider(
  process.env.L2_URL
)

const l1ChainId = (await l1Provider.getNetwork()).chainId
const l2ChainId = (await l2Provider.getNetwork()).chainId

const l1Wallet = new ethers.Wallet(privateKey, l1Provider)
const l2Wallet = new ethers.Wallet(privateKey, l2Provider)

const l1Contracts = {
  AddressManager: addressManager,
  OptimismPortal: optimismPortal,
  L2OutputOracle: l2OutputOracle,
}

const portals = new Portals({
  contracts: {
    l1: l1Contracts,
  },
  l1ChainId,
  l2ChainId,
  l1SignerOrProvider: l1Wallet,
  l2SignerOrProvider: l2Wallet,
})
```

## Deposit

1. **NativeToken Creation:** Create an instance to interface with the `NativeToken` contract on L1. This instance will be used for subsequent token approval and transfer operations. By using the contract's address and the ERC20 standard ABI, you generate a `NativeToken` contract object, which enables interaction with the `OptimismPortal` contract.

```typescript
// NativeTokenContract in L1 (like TON)
const l2NativeTokenContract = new ethers.Contract(
  l2NativeToken,
  erc20ABI,
  l1Wallet
)
```

2. **Approve**: Before depositing NativeToken into L1, you need to approve the `OptimismPortal` contract to use the tokens. This is done by calling the `approve` function to allow the `OptimismPortal` to handle a specified amount of tokens. This step is essential for the deposit process, and once approval is granted, the deposit transaction can proceed successfully.

```typescript
// approve
const approveTx = await l2NativeTokenContract.approve(optimismPortal, amount)
await approveTx.wait()
console.log('approveTx:', approveTx.hash)
```

3. **Deposit Execution:** Use the `portals.depositTransaction` function to deposit NativeToken into L2. In this process, parameters such as `to`, `value`, `gasLimit`, and `data` are set to execute the transaction. Once the transaction is included in a block, you can verify the transaction hash through `depositReceipt`.
   * **`to`:** The L2 address where the deposit is directed.
   * **`value`:** The amount of tokens to deposit.
   * **`gasLimit`:** The gas limit required to execute the transaction, which should be greater than _`data.length * 16 + 21000`_
   * **`data`:** Additional data to be sent, with a default value of `0x`.

```typescript
// deposit NativeToken in L1
const depositTx = await portals.depositTransaction({
  to: l2Wallet.address,
  value: BigNumber.from(amount),
  gasLimit: BigNumber.from('200000'), // It will be greater than data.length * 16 + 21000
  data: '0x',
})
const depositReceipt = await depositTx.wait()
console.log('depositTx:', depositReceipt.transactionHash)
```

4. **Deposit verification:** Use the `portals.waitingDepositTransactionRelayed` function to verify that the deposit was processed successfully. This function returns the associated transaction hash after verifying that the transaction reached L2 and was finally processed. Verify that the deposit was processed correctly with `getTransactionReceipt`.

<pre class="language-typescript"><code class="lang-typescript"><strong>// get relayedTxHash in L2
</strong>const relayedTxHash = await portals.waitingDepositTransactionRelayed(
  depositReceipt,
  {}
)
console.log('relayedTxHash:', relayedTxHash)
const depositedTxReceipt = await l2Provider.getTransactionReceipt(
  relayedTxHash
)
</code></pre>

## Withdrawal

1. **Initiating Withdrawal:** Use the `portals.initiateWithdrawal` function to initiate a withdrawal from L2 to L1. This function creates a withdrawal transaction and uses parameters such as `target`, `value`, `gasLimit`, and `data` to request the withdrawal. The process waits for the transaction to complete through the `withdrawalReceipt`.
   * **`target`**: L1 address where the withdrawal will be sent
   * **`value`**: Amount to withdraw
   * **`gasLimit`**: Set the gas limit
   * **`data`**: Additional data to be sent, with a default value of `0x`.

```typescript
const withdrawalTx = await portals.initiateWithdrawal({
  target: l1Wallet.address,
  value: BigNumber.from(amount),
  gasLimit: BigNumber.from('200000'),
  data: '0x12345678',
})
const withdrawalReceipt = await withdrawalTx.wait()
```

2. **Waiting for Relay**: Use the `portals.waitForWithdrawalTxReadyForRelayUsingL2Tx` function to wait until the withdrawal transaction, based on the L2 transaction hash, is ready to be relayed to L1.

```typescript
await portals.waitForWithdrawalTxReadyForRelayUsingL2Tx(
  withdrawalReceipt.transactionHash
)
```

3. **Withdrawal Prove:** After confirming that the withdrawal transaction is ready to be relayed from L2 to L1, use the `portals.proveWithdrawalTransactionUsingL2Tx` function to prove the withdrawal transaction based on the L2 transaction hash. At this stage, a proof transaction is submitted to verify the validity of the withdrawal transaction.

```typescript
const proveTransaction = await portals.proveWithdrawalTransactionUsingL2Tx(
  withdrawalReceipt.transactionHash
)
await proveTransaction.wait()
```

4. **Wait for Finalization**: Use the `portals.waitForFinalizationUsingL2Tx` function to wait for the withdrawal transaction to finalize. Once finalization is complete, the withdrawal is processed successfully.

```typescript
await portals.waitForFinalizationUsingL2Tx(withdrawalReceipt.transactionHash)
```

5. **Finalizing Withdrawal:** Use the `portals.finalizeWithdrawalTransactionUsingL2Tx` function to finalize the withdrawal transaction based on the L2 transaction hash. At this stage, the withdrawal transaction is fully processed, and the finalized `finalizedTransactionReceipt` can be confirmed.

```typescript
const finalizedTransaction =
  await portals.finalizeWithdrawalTransactionUsingL2Tx(
    withdrawalReceipt.transactionHash
  )
const finalizedTransactionReceipt = await finalizedTransaction.wait()
console.log('finalized transaction receipt:', finalizedTransactionReceipt)
```

6. **Token Transfer:** After the withdrawal is finalized, use the `OptimismPortal` contract to transfer tokens to the user's wallet address. This process transfers tokens to L1 based on the finalized withdrawal transaction.

```typescript
// Transferfrom (after finalization, user have to transferFrom to get his token)
const transferTx = await l2NativeTokenContract.transferFrom(
  l1Contracts.OptimismPortal,
  l1Wallet.address,
  amount
)
await transferTx.wait()
```

## getMessageStatus

`getMessageStatus` is used to check the status of deposit and withdrawal transactions between L1 and L2 when using `OptimismPortal` and `L2ToL1MessagePasser`. Through `getMessageStatus`, it helps verify which stage the transaction is at within the network.

### OptimismPortal

1. **Transaction Creation**: Use the `portals.depositTransaction` function to create a NativeToken deposit transaction from L1 to L2.

```typescript
const depositTx = await portals.depositTransaction({
    to: l2Wallet.address,
    value: BigNumber.from(amount),
    gasLimit: BigNumber.from('200000'),
    data: '0x',
  })
```

2. **Transaction Waiting**: Call `depositTx.wait()` to wait for the transaction to complete and receive the transaction receipt, `depositReceipt`. This receipt includes important information such as the `transactionHash`.

<pre class="language-typescript"><code class="lang-typescript"><strong>const depositReceipt = await depositTx.wait()
</strong>console.log('depositTx:', depositReceipt.transactionHash)
</code></pre>

3. **Relay Hash**: After the transaction is successfully processed on L1, you need to wait for it to be relayed to L2. Use the `portals.waitingDepositTransactionRelayed` function to verify that the transaction submitted on L1 has been successfully relayed to L2 and obtain the `relayedTxHash`. This hash is used to track the transaction on L2.

```typescript
const relayedTxHash = await portals.waitingDepositTransactionRelayed(
  depositReceipt,
  {}
)
```

4. **Status Check**: Use `portals.getMessageStatus` to check the status of the deposit transaction and verify whether it has been successfully relayed on L2.

```typescript
const status = await portals.getMessageStatus(depositReceipt)
console.log('deposit status relayed:', status === MessageStatus.RELAYED)
```

Through this process, you can deposit `NativeToken` from L1 to L2 and use `getMessageStatus` to monitor the final status of the transaction, ensuring that the entire deposit process has been completed successfully.

### L2ToL1MessagePasser

1. **Withdrawal Transaction Creation and Submission**: Use `portals.initiateWithdrawal` to create a withdrawal transaction from L2 to L1. The result of the created transaction is returned as `withdrawalReceipt`, which is then used to track the status of the withdrawal transaction.

```typescript
const withdrawalTx = await portals.initiateWithdrawal({
    target: l1Wallet.address,
    value: BigNumber.from(amount),
    gasLimit: BigNumber.from('200000'),
    data: '0x12345678',
  })
const withdrawalReceipt = await withdrawalTx.wait()
```

2. **Calculate Withdrawal Message Information**: The `portals.calculateWithdrawalMessage` function calculates information about the withdrawal message. This information is used in the subsequent steps of the withdrawal transaction, including the proving and finalization processes.

```typescript
const withdrawalMessageInfo = await portals.calculateWithdrawalMessage(
  withdrawalReceipt
)
console.log('withdrawalMessageInfo:', withdrawalMessageInfo)
```

3. **Check Withdrawal Transaction Status 1**: Use `portals.getMessageStatus` to check the status of the withdrawal transaction. At this stage, you verify whether the root has been successfully posted on L2.

```typescript
let status = await portals.getMessageStatus(withdrawalReceipt)
console.log('[Withdrawal Status] check publish L2 root:', status)
```

4. **Wait for Relay Readiness**: Use the `portals.waitForWithdrawalTxReadyForRelay` function to wait until the withdrawal transaction is ready to be relayed to L1. This process indicates that the withdrawal message created on L2 is prepared for transmission to L1.

```typescript
await portals.waitForWithdrawalTxReadyForRelay(withdrawalReceipt)
```

5. **Check Withdrawal Transaction Status 2**: Call `getMessageStatus` again to verify if the withdrawal transaction is ready to be proven on L1.

```typescript
status = await portals.getMessageStatus(withdrawalReceipt)
console.log('[Withdrawal Status] check ready for proving:', status)
```

6. **Proof of Withdrawal Transaction**: Use `portals.proveWithdrawalTransaction` to prove the withdrawal transaction on L1. This step involves verifying the withdrawal transaction on L1.

```typescript
const proveTransaction = await portals.proveWithdrawalTransaction(
  withdrawalMessageInfo
)
await proveTransaction.wait()
```

7. **Check Withdrawal Transaction Status**: Use `getMessageStatus` to verify if the transaction is in a challenging state on L1. This status indicates that the transaction is still being processed.

```typescript
status = await portals.getMessageStatus(withdrawalReceipt)
console.log('[Withdrawal Status] check in challenging:', status)
```

8. **Wait for Finalization**: Use `portals.waitForFinalization` to wait until the transaction is finalized. Afterward, you can finalize the transaction.

```typescript
await portals.waitForFinalization(withdrawalMessageInfo)
const finalizedTransaction = await portals.finalizeWithdrawalTransaction(
  withdrawalMessageInfo
```

9. **Finalize and Check Withdrawal Transaction**: Use `portals.finalizeWithdrawalTransaction` to finalize the withdrawal transaction. This process indicates that the transaction has been successfully completed. Finally, call `getMessageStatus` to confirm that the withdrawal transaction was successfully relayed on L1.

```typescript
const finalizedTransactionReceipt = await finalizedTransaction.wait()
console.log('finalized transaction receipt:', finalizedTransactionReceipt)

status = await portals.getMessageStatus(withdrawalReceipt)
console.log('[Withdrawal Status] check relayed:', status)
```
