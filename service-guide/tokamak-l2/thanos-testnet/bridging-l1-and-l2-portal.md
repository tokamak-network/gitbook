# Bridging L1 and L2 (Portal)

&#x20;Thanos에서 Portal은 L1의 `OptimismPortal` 컨트랙트와 L2의 `L2ToL1MessagePasser` 컨트랙트를 사용하여 네트워크 간 자산의 입출금을 지원합니다. 단, Portal을 통해 전송할 수 있는 자산은 `NativeToken`으로 한정되며 `CrossChainMessenger`를 사용하는 것보다 더 낮은 가스비로 거래를 처리할 수 있습니다.

본 문서에서는 입금과 출금 시에 사용되는 Portal의 함수들을 살펴보고 Portal을 import하여 `NativeToken`을 L1과 L2 간에 입출금하는 방법을 예제를 통해 단계별로 설명합니다.

&#x20;[여기](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/sdk/src/portals.ts)를 통해 Portal에 대한 자세한 코드를 확인할 수 있습니다. Thanos SDK를 활용하여 L1과 L2 간에 `NativeToken`을 입출금하는 예제 코드는 [여기](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/sdk/tasks/portals.ts)에서 확인할 수 있습니다.

## 생성자 (Constructor)

생성자는 `CrossChainProvider` 인스턴스를 초기화하는 데 필요한 정보를 설정합니다. 각 생성자는 L1과 L2 간의 상호작용을 원활하게 처리하기 위해 필수적인 요소들로 구성됩니다.

* `l1SignerOrProvider`: L1과 상호작용하기 위한 서명자 또는 제공자
* `l2SignerOrProvider`: L2와 상호작용하기 위한 서명자 또는 제공자
* `l1ChainId`: L1 체인의 고유 식별자
* `l2ChainId`: L2 체인의 고유 식별자
* `depositConfirmationBlocks`: 입금이 확인되기까지 필요한 블록 수
* `l1BlockTimeSeconds`: L1 체인의 블록 생성 시간 (초)
* `contracts`: 오버라이드할 수 있는 특정 계약 주소

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

Ethereum 블록체인과 상호작용하는 `ethers` 라이브러리와 `@tokamak-network/thanos-sdk` 패키지를 import 합니다.

또한, Portal의 생성자를 통해 L1과 L2간의 상호작용을 위한 인스턴스를 생성하고 Portal에서 제공하는 다양한 함수를 사용할 준비를 합니다.

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

## 입금 (Deposit)

1. **NativeToken 생성**: L1에서 `NativeToken` 컨트랙트와 상호작용하기 위한 인스턴스를 생성합니다. 해당 인스턴스를 사용하여 이후 토큰의 승인 및 전송 작업을 수행할 수 있습니다. 컨트랙트의 주소와 ERC20 표준 ABI를 사용하여 `NativeToken` 컨트랙트 객체를 생성하며 이를 기반으로 `OptimismPortal` 컨트랙트와의 상호작용이 가능해집니다.

```typescript
// NativeTokenContract in L1 (like TON)
const l2NativeTokenContract = new ethers.Contract(
  l2NativeToken,
  erc20ABI,
  l1Wallet
)
```

2. **사전 승인**: L1에서 `NativeToken`을 입금하기 전, `OptimismPortal` 컨트랙트가 해당 토큰을 사용할 수 있도록 사전에 승인을 받아야 합니다. 이를 위해 `approve` 함수를 호출하여 지정된 수량만큼의 토큰을 `OptimismPortal`에서 처리할 수 있도록 허용합니다. 이 절차는 입금 과정에서 필수이며 승인 이후에 입금 트랜잭션이 성공적으로 진행됩니다.

```typescript
// approve
const approveTx = await l2NativeTokenContract.approve(optimismPortal, amount)
await approveTx.wait()
console.log('approveTx:', approveTx.hash)
```

3. **입금 실행**: `portals.depositTransaction` 함수를 사용하여 L2로 `NativeToken`을 입금합니다. 이 과정에서 `to`, `value`, `gasLimit`, `data`와 같은 매개변수를 설정하여 트랜잭션을 실행합니다. 트랜잭션이 블록에 포함되면 `depositReceipt을` 통해 트랜잭션 해시를 확인할 수 있습니다.

* **`to`**: 입금을 진행할 L2 주소
* **`value`**: 입금 금액
* **`gasLimit`**: 트랜잭션 실행 시 필요한 가스 한도 설정, `data.length * 16 + 21000` 보다 커야합니다.
* **`data`**: 추가로 전달한 데이터 (기본 값은 `0x`)

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

4. **입금 확인**: 입금이 성공적으로 처리되었는지 확인하기 위해  `portals.waitingDepositTransactionRelayed` 함수를 사용합니다. 이 함수는 트랜잭션이 L2에 도달하고 최종적으로 처리되었는지 확인 후 관련된 트랜잭션 해시를 반환합니다. `getTransactionReceipt`를 통해 입금이 올바르게 처리되었는지 확인합니다.

```typescript
// get relayedTxHash in L2
const relayedTxHash = await portals.waitingDepositTransactionRelayed(
  depositReceipt,
  {}
)
console.log('relayedTxHash:', relayedTxHash)
const depositedTxReceipt = await l2Provider.getTransactionReceipt(
  relayedTxHash
)
```

## 출금 (Withdrawal)

1. **출금 시작**: `portals.initiateWithdrawal`함수를 사용하여 L2에서 L1으로 출금을 시작합니다. 이 함수는 출금 트랜잭션을 생성하고 `target`, `value`, `gasLimit`, `data`와 같은 매개변수를 사용하여 출금을 요청합니다. `withdrawalReceipt`을 통해 트랜잭션이 완료될 때까지 대기합니다.

* **`target`**: 출금을 진행할 L1 주소
* **`value`**: 출금 금액
* **`gasLimit`**: 가스 한도 설정
* **`data`**: 추가로 전달한 데이터 (기본 값은 `0x`)

```typescript
const withdrawalTx = await portals.initiateWithdrawal({
  target: l1Wallet.address,
  value: BigNumber.from(amount),
  gasLimit: BigNumber.from('200000'),
  data: '0x12345678',
})
const withdrawalReceipt = await withdrawalTx.wait()
```

2. **릴레이 대기**: `portals.waitForWithdrawalTxReadyForRelayUsingL2Tx` 함수를 통해 L2 트랜잭션 해시를 기반으로 출금 트랜잭션이 L1으로 릴레이가 될 때까지 대기합니다.

```typescript
await portals.waitForWithdrawalTxReadyForRelayUsingL2Tx(
  withdrawalReceipt.transactionHash
)
```

3. **출금 증명**: 출금 트랜잭션이 L2에서 L1으로 릴레이될 준비가 되었음을 확인한 후 `portals.proveWithdrawalTransactionUsingL2Tx` 함수를 통해 L2 트랜잭션 해시를 기반으로 출금 트랜잭션을 증명합니다. 이 단계에서는 출금 트랜잭션의 유효성을 확인하기 위해 증명 트랜잭션을 제출합니다.&#x20;

```typescript
const proveTransaction = await portals.proveWithdrawalTransactionUsingL2Tx(
  withdrawalReceipt.transactionHash
)
await proveTransaction.wait()
```

4. **확정 대기**:  `portals.waitForFinalizationUsingL2Tx` 함수를 사용하여 출금 트랜잭션이 확정될 때까지 대기합니다. 출금 트랜잭션의 확정이 완료되면 출금이 성공적으로 처리됩니다.

```typescript
await portals.waitForFinalizationUsingL2Tx(withdrawalReceipt.transactionHash)
```

5. **출금 확정**: `portals.finalizeWithdrawalTransactionUsingL2Tx`함수를 사용하여 L2 트랜잭션 해시를 기반으로 출금 트랜잭션을 확정합니다. 이 단계에서 출금 트랜잭션의 최종 처리가 완료되며 `finalizedTransactionReceipt`을 확인합니다.

```typescript
const finalizedTransaction =
  await portals.finalizeWithdrawalTransactionUsingL2Tx(
    withdrawalReceipt.transactionHash
  )
const finalizedTransactionReceipt = await finalizedTransaction.wait()
console.log('finalized transaction receipt:', finalizedTransactionReceipt)
```

6. **토큰 전송**: 출금이 확정된 후 `OptimismPortal` 컨트랙트를 이용하여 사용자의 지갑 주소로 토큰을 전송합니다. 이 과정은 확정된 출금 트랜잭션에 따라 L1으로 토큰을 전송하는 작업을 수행합니다.

```typescript
// Transferfrom (after finalization, user have to transferFrom to get his token)
const transferTx = await l2NativeTokenContract.transferFrom(
  l1Contracts.OptimismPortal,
  l1Wallet.address,
  amount
)
await transferTx.wait()
```

## 트랜잭션 상태 확인 (getMessageStatus)

`getMessageStatus`는 `OptimismPortal` 및 `L2ToL1MessagePasser`를 사용할 때  L1과 L2 간의 입출금 트랜잭션의 상태를 확인하는 데 사용됩니다.&#x20;

### OptimismPortal

1. **트랜잭션 생성**: `portals.depositTransaction` 함수를 사용하여 L1에서 L2로 `NativeToken` 입금 트랜잭션을 생성합니다.

```typescript
const depositTx = await portals.depositTransaction({
    to: l2Wallet.address,
    value: BigNumber.from(amount),
    gasLimit: BigNumber.from('200000'),
    data: '0x',
  })
```

2. **트랜잭션 대기**: `depositTx.wait()`을 호출하여 트랜잭션이 완료되면 트랜잭션 영수증인 `depositReceipt` 를 받습니다.  이 영수증에는  `transactionHash` 등 중요한 정보가 포함됩니다.

<pre class="language-typescript"><code class="lang-typescript"><strong>const depositReceipt = await depositTx.wait()
</strong>console.log('depositTx:', depositReceipt.transactionHash)
</code></pre>

3. **릴레이 해시**: 트랜잭션이 L1에서 성공적으로 처리한 후 해당 트랜잭션이 L2로 릴레이될 때까지 기다려야 합니다. `portals.waitingDepositTransactionRelayed` 함수를 사용하여 L1에서 제출된 트랜잭션이 L2에 성공적으로 릴레이되었는지 확인하고 `relayedTxHash`를 가져옵니다. 이 해시는 L2에서 발생한 트랜잭션을 추적하는 데 사용됩니다.

```typescript
const relayedTxHash = await portals.waitingDepositTransactionRelayed(
  depositReceipt,
  {}
)
```

4. **상태 확인**: `portals.getMessageStatus`를 사용하여 입금 트랜잭션의 상태를 확인하고 L2에서 성공적으로 릴레이되었는지 확인합니다.&#x20;

```typescript
const status = await portals.getMessageStatus(depositReceipt)
console.log('deposit status relayed:', status === MessageStatus.RELAYED)
```

이 흐름을 통해 L1에서 L2로 `NativeToken`을 입금한 후 `getMessageStatus`를 사용하여 트랜잭션의 최종 상태를 모니터링하여 전체 입금 과정이 정상적으로 처리되었는지 확인할 수 있습니다.

### L2ToL1MessagePasser

1. **출금 트랜잭션 생성 및 제출**: `portals.initiateWithdrawal`을 통해 L2에서 L1으로 출금 트랜잭션을 생성합니다. 생성된 트랜잭션의 결과는 `withdrawalReceipt` 으로 반환되며 이후 출금 트랜잭션의 상태를 추적하는 데 사용됩니다.

```typescript
const withdrawalTx = await portals.initiateWithdrawal({
    target: l1Wallet.address,
    value: BigNumber.from(amount),
    gasLimit: BigNumber.from('200000'),
    data: '0x12345678',
  })
const withdrawalReceipt = await withdrawalTx.wait()
```

2. **출금 메세지 정보 계산**: `portals.calculateWithdrawalMessage`는 출금 메시지에 대한 정보를 계산합니다. 이 정보는 출금 트랜잭션의 다음 단계인 증명 및 확정 프로세스에서 활용됩니다.

```typescript
const withdrawalMessageInfo = await portals.calculateWithdrawalMessage(
  withdrawalReceipt
)
console.log('withdrawalMessageInfo:', withdrawalMessageInfo)
```

3. **출금 트랜잭션 상태 확인**: `portals.getMessageStatus`를 사용하여 출금 트랜잭션의 상태를 확인합니다. 이 단계에서는 L2에서 root가 성공적으로 게시되었는지 확인합니다.

```typescript
let status = await portals.getMessageStatus(withdrawalReceipt)
console.log('[Withdrawal Status] check publish L2 root:', status)
```

4. **릴레이 준비 상태 대기**: 출금 트랜잭션이 L1으로 릴레이될 준비가 될 때까지 `portals.waitForWithdrawalTxReadyForRelay` 함수를 사용하여 대기합니다. 이 과정은 L2에서 생성된 출금 메시지가 L1으로 전송될 준비가 되었음을 나타냅니다.

```typescript
await portals.waitForWithdrawalTxReadyForRelay(withdrawalReceipt)
```

5. **출금 트랜잭션 상태 확인 1**: `getMessageStatus`를 다시 호출하여 출금 트랜잭션이 L1에서 증명될 준비가 되었는지 확인합니다.

```typescript
status = await portals.getMessageStatus(withdrawalReceipt)
console.log('[Withdrawal Status] check ready for proving:', status)
```

6. **출금 트랜잭션 증명**: `portals.proveWithdrawalTransaction`을 사용하여 L1에서 출금 트랜잭션을 증명합니다. 이 단계는 출금 트랜잭션을 L1에서 확인하는 과정입니다.

```typescript
const proveTransaction = await portals.proveWithdrawalTransaction(
  withdrawalMessageInfo
)
await proveTransaction.wait()
```

7. **출금 트랜잭션 상태 확인 2**: 트랜잭션이 L1에서 챌린징 상태에 있는지 `getMessageStatus`로 확인합니다. 이 상태는 트랜잭션이 아직 처리 중임을 나타냅니다.

```typescript
status = await portals.getMessageStatus(withdrawalReceipt)
console.log('[Withdrawal Status] check in challenging:', status)
```

8. **확정 대기**: 트랜잭션이 최종적으로 완료될 때까지 `portals.waitForFinalization`을 호출해 대기합니다. 이후, 트랜잭션의 확정 프로세스를 수행합니다.

```typescript
await portals.waitForFinalization(withdrawalMessageInfo)
const finalizedTransaction = await portals.finalizeWithdrawalTransaction(
  withdrawalMessageInfo
```

9. **출금 트랜잭션 확정 및 상태 확인**: `portals.finalizeWithdrawalTransaction`을 사용하여 출금 트랜잭션을 확정합니다. 이 과정은 트랜잭션이 성공적으로 완료되었음을 나타냅니다. 마지막으로 `getMessageStatus`를 호출하여 출금 트랜잭션이 L1에서 성공적으로 릴레이되었는지 확인합니다.

```typescript
const finalizedTransactionReceipt = await finalizedTransaction.wait()
console.log('finalized transaction receipt:', finalizedTransactionReceipt)

status = await portals.getMessageStatus(withdrawalReceipt)
console.log('[Withdrawal Status] check relayed:', status)
```



