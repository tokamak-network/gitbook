# About USDCBridge

### Intro

This page explains the process of depositing and withdrawing USDC between L1 and L2 using the Thanos Stack USDCBridge. The key topics include the functionality and transfer process of the USDC Bridge, information about the USDC token, and the deposit and withdrawal logic demonstrated through test code.

### Overview

In Thanos Stack, the USDCBridge is predeployed on L2 to facilitate the seamless transfer of USDC tokens between L1 and L2. This allows users to trade USDC more easily and manage asset transfers between L1 and L2 securely and efficiently. The USDCBridge supports USDC transfers between L1 and L2, which is crucial in preventing asset loss and ensuring safe cross-chain transactions. While StandardBridge supports general token transfers, it may not fully comply with [Circle's policies and logic](https://www.circle.com/blog/bridged-usdc-standard) for USDC transactions. On the other hand, USDCBridge adheres to Circle's policies and logic, offering better compatibility and stability. Therefore, in Thanos stack, USDCBridge is provided, and using it significantly reduces the risk of asset loss and helps prevent unexpected issues.

### Thanos Stack: USDC Token Information

* L1 USDC Address:
  * [Mainnet](https://etherscan.io/token/0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48):`0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`
* L2
  * USDC Bridge Address: `0x4200000000000000000000000000000000000775`
  * USDC Token Address: `0x4200000000000000000000000000000000000778`
  * USDC Token Name: Bridged USDC (Tokamak Network)
  * USDC Token Symbol: USDC.e
  * USDC Token Decimals: 6

### Example

This section uses test code examples to demonstrate the **USDC transfer process**.

* **Approve**: It is necessary to grant permission to the USDC Bridge for the transfer of USDC tokens between L1 and L2.
* **Deposit**: USDC can be deposited from L1 to L2 via the L1UsdcBridge. The OptimismPortal monitors the deposit event on L2, and the tokens are transferred to L2.
* **Withdrawal**: USDC can be withdrawn from L2 to L1 via the L2UsdcBridge. The OptimismPortal verifies the withdrawal event, and the tokens are withdrawn on L1.

The deposit and withdrawal logic can be verified through the flow of the test code by using [Thanos SDK](https://tokamak-network.github.io/thanos-sdk-docs/):

1. Setup

```typescript
const l1RpcProvider = new ethers.providers.JsonRpcProvider(process.env.L1_RPC);
const l2RpcProvider = new ethers.providers.JsonRpcProvider(process.env.L2_RPC)
const privateKey = process.env.PRIVATE_KEY;
const depositAmount = BigInt(1e6);
const withdrawAmount = BigInt(1e6);
const l1Wallet = new ethers.Wallet(privateKey, l1RpcProvider);
const l2Wallet = new ethers.Wallet(privateKey, l2RpcProvider);
const l1USDCAddr = '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48';
const l2USDCAddr = '0x4200000000000000000000000000000000000778'

const crossChainMessenger = new thanosSDK.CrossChainMessenger({
    l1ChainId: process.env.L1_CHAIN_ID,
    l2ChainId: process.env.L2_CHAIN_ID,
    l1SignerOrProvider: l1Wallet,
    l2SignerOrProvider: l2Wallet,
    bedrock: true,
    nativeTokenAddress: nativeToken
 });
```

2. Deposit USDC

```typescript
const allowanceResponse = await crossChainMessenger.approveERC20(
    '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48',
    '0x4200000000000000000000000000000000000778',
    depositAmount
  );
  await allowanceResponse.wait();
  console.log(`Approval ERC20 token transaction hash (on L1): ${allowanceResponse.hash}`)

  const response = await crossChainMessenger.bridgeERC20(
    l1USDCAddr,
    l2USDCAddr,
    depositAmount
  );
  console.log(`Deposit transaction hash (on L1): ${response.hash}`);
  await response.wait();

  console.log("Waiting for status to change to RELAYED");
  
  await crossChainMessenger.waitForMessageStatus(
    response.hash,
    thanosSDK.MessageStatus.RELAYED
  );
```

3. Withdraw

```typescript
// NOTE: If you want to withdraw USDC.e, you must approve the L2USDCBridge contract with the approval function
const tx = await l2ERC20.approve(L2USDCBridge, withdrawAmount);

console.log("Transaction sent. Waiting for confirmation...");
const receipt = await tx.wait();
console.log("Approval successful:", receipt.transactionHash);

const withdrawalResponse = await crossChainMessenger.withdrawERC20(l1USDCAddr, l2USDCAddr, withdrawAmount)
const withdrawalTx = await withdrawalResponse.wait()

console.log(`Withdraw transaction hash: ${withdrawalTx.transactionHash}`)

console.log(`Wait the message status changed to READY_TO_PROVE`)

await crossChainMessenger.waitForMessageStatus(
  withdrawalTx.transactionHash,
  thanosSDK.MessageStatus.READY_TO_PROVE
)

console.log('Prove the message...')
const proveTx = await crossChainMessenger.proveMessage(withdrawalTx.transactionHash)
const proveReceipt = await proveTx.wait(3)
console.log('Proved transaction hash: ', proveReceipt.transactionHash)

const finalizeInterval = setInterval(async () => {
  const currentStatus = await crossChainMessenger.getMessageStatus(withdrawalTx)
  console.log('Current message status: ', currentStatus)
}, 3000)

try {
  await crossChainMessenger.waitForMessageStatus(
    withdrawalTx,
    thanosSDK.MessageStatus.READY_FOR_RELAY
  )
} finally {
  clearInterval(finalizeInterval)
}

console.log(`Ready for relay, finalizing the message....`)
const finalizeTxResponse = await crossChainMessenger.finalizeMessage(withdrawalTx.transactionHash)
const finalizeTxReceipt = await finalizeTxResponse.wait()
console.log('Finalized message tx', finalizeTxReceipt.transactionHash)

console.log(`Waiting for status to change to RELAYED`)
await crossChainMessenger.waitForMessageStatus(
  withdrawalResponse,
  thanosSDK.MessageStatus.RELAYED
)
```

### Additional Considerations

* Use the specified L1 USDC address (`0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`): To avoid asset loss, it is important to use Circle's Ethereum mainnet USDC address supported by the Thanos stack. Using the wrong address may result in the loss of funds.
* Pre-approve USDC transfers to UsdcBridge: Before initiating token transfers, pre-approve the USDC Bridge contract. This approval is required to authorize the bridge to transfer your tokens on your behalf, ensuring the transaction is executed successfully.

### Reference Links and Github Links

* USDC contracts in Tokamak Thanos repo: [GitHub](https://github.com/tokamak-network/tokamak-thanos/tree/main/packages/tokamak/contracts-bedrock/src/tokamak-contracts/USDC)
* USDC dep/with test code: [GitHub](https://github.com/tokamak-network/tokamak-thanos/blob/main/op-e2e/e2e-dep-with/USDC_test.go)
* Circle USDC on main networks: [circle docs](https://developers.circle.com/stablecoins/usdc-on-main-networks)
* More information about Bridged USDC Standard: [circle docs](https://www.circle.com/bridged-usdc#get-started)
