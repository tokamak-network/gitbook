# Bridge ETH with the Thanos SDK

This tutorial describes how the [Thanos SDK](https://www.npmjs.com/package/@tokamak-network/thanos-sdk) can bridge ETH from L1 to L2. You can look at our example: [Github](https://github.com/tokamak-network/tokamak-thanos-tutorial/tree/main/cross-dom-bridge-eth)

#### 1. Prerequisite

**1.1. Dependencies**

* [node](https://nodejs.org/en)
* [npm](https://www.npmjs.com/)

**1.2. Install packages**

To use Thanos SDK, first, you must install the package from NPM.

```bash
npm install @tokamak-network/thanos-sdk@latest
```

#### 2. Set Environments

```tsx
// .env example

PRIVATE_KEY=change_me
L1_RPC_URL=change_me
L2_RPC_URL=change_me
L1_TOKEN=change_me
L2_TOKEN=change_me
```

#### 3. Create providers and wallets

```tsx
const privateKey = process.env.PRIVATE_KEY;

const l1RpcProvider = new ethers.providers.JsonRpcProvider(process.env.L1_RPC_URL);
const l2RpcProvider = new ethers.providers.JsonRpcProvider(process.env.L2_RPC_URL)
const l1Wallet = new ethers.Wallet(privateKey, l1RpcProvider);
const l2Wallet = new ethers.Wallet(privateKey, l2RpcProvider);

```

#### 4. Use Thanos SDK to interact between two networks

`CrossChainMessenger` in `Thanos SDK` is designed to facilitate cross-chain communication between Ethereum and the L2 built by Thanos Stack. It allows sending messages and executes transactions between L1 and L2, simplifying moving assets between them.

The CrossChainMessenger constructor is:

```tsx
new CrossChainMessenger(opts: {
  l1SignerOrProvider: SignerOrProviderLike
  l2SignerOrProvider: SignerOrProviderLike
  l1ChainId: NumberLike
  l2ChainId: NumberLike
  nativeTokenAddress?: AddressLike
  depositConfirmationBlocks?: NumberLike
  l1BlockTimeSeconds?: NumberLike
  contracts?: DeepPartial<OEContractsLike>
  bridges?: BridgeAdapterData
  bedrock?: boolean
}): CrossChainMessenger
```

To deposit ETH, we can initialize the CrossChainMessenger instance:

```tsx
const messenger = new CrossChainMessenger({
    l1ChainId: (await l1Provider.getNetwork()).chainId,
    l2ChainId: (await l2Provider.getNetwork()).chainId,
    l1SignerOrProvider: l1Wallet,
    l2SignerOrProvider: l2Wallet,
    bedrock: true
});
```

We already defined the known network on the CrossChainMessenger class. But if your network is custom, you can initialize the CrossChainMessenger instance with the custom `opts.contracts` .

#### **5. Setup the token contracts**

First, you must import the `@tokamak-network/core-utils` package.

```tsx
const coreUtils = require('@tokamak-network/core-utils')
```

Then, set up the ETH contract on L2.

```tsx
const erc20ABI = [{"inputs":[{"internalType":"address","name":"_spender","type":"address"},{"internalType":"uint256","name":"_value","type":"uint256"}],"name":"approve","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"type":"function"},{"inputs":[{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"faucet","outputs":[],"stateMutability":"nonpayable","type":"function"}]

const ethContractOnL2 = new ethers.Contract(coreUtils.predeploys.ETH, erc20ABI, l2Signer)
```

#### **6. Deposit tokens**

You can use the L1 standard bridge contract to bridge the ETH from L1 to L2. You will receive the same amount in the L2 after successfully depositing it.

**6.1. Deposit tokens**

Predefine your deposit amount.

```tsx
const depositAmount = 1e6;
```

Execute your deposit transaction by using the `bridgeETH` function.

```tsx
const depositTx = await crossChainMessenger.bridgeETH(
    depositAmount
);
await depositTx.wait();
console.log(`Deposit transaction hash (on L1): ${depositTx.hash}`);
```

**6.2. Wait for the deposit to be relayed**

We can wait for the deposit transaction until it’s relayed on the L2 chain by using the `waitForMessageStatus` function.

```tsx
await crossChainMessenger.waitForMessageStatus(
  depositTx.hash,
  thanosSDK.MessageStatus.RELAYED
);
```

**6.3. Verify your balances**

```tsx
let l1ETHBalance = await l1Signer.getBalance()
let l2ETHBalance = await ethContractOnL2.balanceOf(l2Signer.address)
console.log(`ETH Token on L1:${l1ETHBalance} \\n
            ETH Token on L2: ${l2ETHBalance}`);
```

#### **7. Withdraw tokens**

To withdraw ETH from L2 to L1, you can use the L2 standard bridge contract.

Predefine your withdrawal amount.

```tsx
const withdrawAmount = 1e6;
```

**7.1. Initialize your withdrawal transaction**

To withdraw your tokens on L2, you can use the `withdrawETH` function.

```tsx
const withdrawalResponse = await crossChainMessenger.withdrawETH(withdrawAmount)
const withdrawalTx = await withdrawalResponse.wait()
```

**7.2. Wait for your withdrawal transaction until it is ready to prove**

You must prove the withdrawal transaction on L1. So, you need to wait until the withdrawal is ready to prove.

```tsx
await crossChainMessenger.waitForMessageStatus(
  withdrawalTx.transactionHash,
  thanosSDK.MessageStatus.READY_TO_PROVE
)
```

**7.3. Prove the withdrawal transaction on L1**

When your withdrawal transaction is ready to be proven, you must send the proven transaction on L1 to prove that the transaction happened on L2.

```tsx
const proveTx = await crossChainMessenger.proveMessage(withdrawalTx.transactionHash)
const proveReceipt = await proveTx.wait(3)
```

**7.4. Wait for your withdrawal transaction until it is ready to finalize**

The final step is to finalize your withdrawal transaction on L1. This can only happen after the fault-proof period has elapsed.

```tsx
 await crossChainMessenger.waitForMessageStatus(
  withdrawalTx,
  thanosSDK.MessageStatus.READY_FOR_RELAY
)
```

💡

**7.5. Finalize your withdrawal transaction**

Once the withdrawal is ready to be relayed, you can finally complete the withdrawal process.

```tsx
const finalizeTxResponse = await crossChainMessenger.finalizeMessage(withdrawalTx.transactionHash)
const finalizeTxReceipt = await finalizeTxResponse.wait()
```

**7.6. Wait until your withdrawal is relayed**

```tsx
await crossChainMessenger.waitForMessageStatus(
  withdrawalResponse,
  thanosSDK.MessageStatus.RELAYED
)
```

**7.7. Verify your balances**

```tsx
let l1ETHBalance = await l1Signer.getBalance()
let l2ETHBalance = await ethContractOnL2.balanceOf(l2Signer.address)
console.log(`ETH Token on L1:${l1ETHBalance} \\n
            ETH Token on L2: ${l2ETHBalance}`);
```
