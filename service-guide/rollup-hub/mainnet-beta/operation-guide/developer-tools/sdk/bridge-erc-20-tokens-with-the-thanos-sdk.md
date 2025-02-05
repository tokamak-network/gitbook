# Bridge ERC-20 tokens with the Thanos SDK

This tutorial describes how the [Thanos SDK](https://www.npmjs.com/package/@tokamak-network/thanos-sdk) can bridge ERC-20 tokens from L1 to L2. You can look at our example: [Github](https://github.com/tokamak-network/tokamak-thanos-tutorial/tree/main/cross-dom-bridge-erc20)

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

`CrossChainMessenger` in Thanos SDK is designed to facilitate cross-chain communication between Ethereum and the L2 built by Thanos Stack. It allows sending messages and executes transactions between L1 and L2, simplifying moving assets between them.

The CrossChainMessenger constructor is

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

To deposit ERC-20 tokens, we can initialize the CrossChainMessenger instance:

```tsx
const messenger = new CrossChainMessenger({
    l1ChainId: (await l1Provider.getNetwork()).chainId,
    l2ChainId: (await l2Provider.getNetwork()).chainId,
    l1SignerOrProvider: l1Wallet,
    l2SignerOrProvider: l2Wallet,
    bedrock: true
});
```

We already defined the known network on the CrossChainMessenger class. But if your network is custom, you can initialize the CrossChainMessenger instance with the custom `opts.contracts`

#### **5. Set the L1 and L2 ERC-20 addresses**

The l1token must be deployed on the L1 network. And the l2Token must be deployed on the L2 network.

```tsx
const l1Erc20Addr = process.env.L1_TOKEN
const l2Erc20Addr = process.env.L2_TOKEN
```

#### **6. Setup the token contracts**

```tsx
const erc20ABI = [{"inputs":[{"internalType":"address","name":"_spender","type":"address"},{"internalType":"uint256","name":"_value","type":"uint256"}],"name":"approve","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"type":"function"},{"inputs":[{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"faucet","outputs":[],"stateMutability":"nonpayable","type":"function"}]

const l1ERC20 = new ethers.Contract(l1Erc20Addr, erc20ABI, l1Wallet);
const l2ERC20 = new ethers.Contract(l2Erc20Addr, erc20ABI, l2Wallet);
```

#### **7. Deposit tokens**

You can use the L1 standard bridge contract to bridge the ERC-20 tokens from L1 to L2. You will receive the same amount in the L2 after successfully depositing it.

**7.1. Approve the StandardBridge use of your tokens**

```tsx
const approveTx = await crossChainMessenger.approveERC20(
    l1Erc20Addr,
    l2Erc20Addr,
    depositAmount
);
await approveTx.wait();
```

**7.2. Deposit tokens**

Predefine your deposit amount.

```tsx
const depositAmount = 1e6;
```

Execute your deposit transaction by using the `bridgeERC20` function.

```tsx
const depositTx = await crossChainMessenger.bridgeERC20(
    l1Erc20Addr,
    l2Erc20Addr,
    depositAmount
);
await depositTx.wait();
console.log(`Deposit transaction hash (on L1): ${depositTx.hash}`);
```

**7.3. Wait for the deposit to be relayed**

We can wait for the deposit transaction until it’s relayed on the L2 chain by using the `waitForMessageStatus` function.

```tsx
await crossChainMessenger.waitForMessageStatus(
  depositTx.hash,
  thanosSDK.MessageStatus.RELAYED
);
```

**7.4. Verify your balances**

```tsx
 const walletAddr = l1Wallet.address;
 const l1Balance = await l1ERC20.balanceOf(walletAddr);
 const l2Balance = await l2ERC20.balanceOf(walletAddr);
 console.log(`Token on L1:${l1Balance}     Token on L2:${l2Balance}`);
```

#### **8. Withdraw tokens**

To withdraw the tokens from L2 to L1, you can use the L2 standard bridge contract.

Predefine your withdrawal amount.

```tsx
const withdrawAmount = 1e6;
```

**8.1. Approve if your token is USDC**

Because the USDC isn’t supported natively on the L2 network, so we follow [the Circle standard](https://www.circle.com/blog/bridged-usdc-standard) to bring USDC to our L2 network.

Because of that, the USDC bridged token on the L2 network isn’t implemented by `IOptimismMintableERC20` interfaces. So, you must approve the USDC bridged to our L2 standard bridge contract before executing the withdrawal transaction.

```tsx
const L2USDCAddr = '0x4200000000000000000000000000000000000778'

if (l2Erc20Addr === L2USDCAddr) {
  const tx = await l2ERC20.approve(L2USDCBridge, withdrawAmount);
  const receipt = await tx.wait();
}
```

**8.2. Initialize your withdrawal transaction**

To withdraw your tokens on L2, you can use the `withdrawaERC20` function.

```tsx
const withdrawalResponse = await crossChainMessenger.withdrawERC20(l1Erc20Addr, l2Erc20Addr, withdrawAmount)
const withdrawalTx = await withdrawalResponse.wait()
```

**8.3. Wait for your withdrawal transaction until it is ready to prove**

You must prove the withdrawal transaction on L1. So, you need to wait until the withdrawal is ready to prove.

```tsx
await crossChainMessenger.waitForMessageStatus(
  withdrawalTx.transactionHash,
  thanosSDK.MessageStatus.READY_TO_PROVE
)
```

**8.4. Prove the withdrawal transaction on L1**

When your withdrawal transaction is ready to be proven, you must send the proven transaction on L1 to prove that the transaction happened on L2.

```tsx
const proveTx = await crossChainMessenger.proveMessage(withdrawalTx.transactionHash)
const proveReceipt = await proveTx.wait(3)
```

**8.5. Wait for your withdrawal transaction until it is ready to finalize**

The final step is to finalize your withdrawal transaction on L1. This can only happen after the fault-proof period has elapsed. On Thanos Sepolia, the period is 24 mins.

```tsx
 await crossChainMessenger.waitForMessageStatus(
  withdrawalTx,
  thanosSDK.MessageStatus.READY_FOR_RELAY
)
```

**8.6. Finalize your withdrawal transaction**

Once the withdrawal is ready to be relayed, you can finally complete the withdrawal process.

```tsx
const finalizeTxResponse = await crossChainMessenger.finalizeMessage(withdrawalTx.transactionHash)
const finalizeTxReceipt = await finalizeTxResponse.wait()
```

**8.7. Wait until your withdrawal is relayed**

```tsx
await crossChainMessenger.waitForMessageStatus(
  withdrawalResponse,
  thanosSDK.MessageStatus.RELAYED
)
```

**8.8. Verify your balances**

```tsx
 const walletAddr = l1Wallet.address;
 const l1Balance = await l1ERC20.balanceOf(walletAddr);
 const l2Balance = await l2ERC20.balanceOf(walletAddr);
 console.log(`Token on L1:${l1Balance}     Token on L2:${l2Balance}`);
```
