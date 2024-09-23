# Differences Between Ethereum and Thanos

Thanos 는 EVM 과 동등하도록 설계되었습니다. 그리고 Ethereum 프로토콜에 가능한 한 적은 변경 사항을 도입합니다. 하지만, Ethereum 과 Thanos 의 동작에는 개발자가 알아야 할 몇 가지 차이점이 있습니다.

## Opcodes <a href="#opcodes" id="opcodes"></a>

| Opcode       | Solidity Equivalent | Behavior                                                                                                                                  |
| ------------ | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `COINBASE`   | `block.coinbase`    | 현재 Sequencer 의 수수료 지갑 주소를 반환합니다. Ethereum 과 실질적으로 동일하지만 값은 일반적으로 블록마다 변경되지 않습니다.                                                          |
| `PREVRANDAO` | `block.prevrandao`  | Ethereum 에서 [RANDAO](https://eips.ethereum.org/EIPS/eip-4399) 가 제공하는 강력한 보장과 달리 Sequencer 는 각 블록에 대해 **의사 난수를** 설정합니다.                    |
| `ORIGIN`     | `tx.origin`         | 거래가 L1 의 Smart Contract 에  에 의해 시작된 L1 ⇒ L2 거래인 경우, L1 ⇒ L2 거래를 시작한 주소(`tx.origin`) 의 Address Alias 로 설정됩니다 . 그렇지 않으면 이 명령어는 정상적으로 작동합니다. |
| `CALLER`     | `msg.sender`        | 거래가 L1 의 Smart Contract 에 의해 시작된 L1 ⇒ L2 거래이고, 이것이 L2 에서 첫 번째로 호출된 Contract 인 경우, Address Alias 가 적용됩니다.                                  |

## Address Aliasing

{% hint style="info" %}
주소 별칭은 스마트 계약에서 L1에서 L2로 전송된 거래의 동작에 영향을 미치는 중요한 보안 기능입니다. 크로스 체인 거래를 사용하는 경우 이 섹션을 주의 깊게 읽어보세요. 계약이 `CrossChainMessenger` 내부적으로 주소 별칭을 처리한다는 점에 유의하세요.
{% endhint %}

EOA 를 사용하여 L1에서 L2로 트랜잭션을 보내는 경우, L2 에서 트랜잭션을 보낸 사람의 주소는 L1 에서 트랜잭션을 보낸 사람의 주소로 설정됩니다. **그러나 트랜잭션이 L1에서 스마트 계약에 의해 트리거된 경우 L2에서 트랜잭션을 보낸 사람의 주소는 다릅니다**.

`CREATE` opcode 의 동작으로 인해 L1 과 L2 모두에서 동일한 주소를 공유하지만 Smart Contract 의 `bytecode` 가 달라질 수 있습니다. 이러한 계약은 동일한 주소를 공유하지만 근본적으로 두 개의 다른 스마트 계약이며 동일한 계약으로 취급할 수 없습니다. 결과적으로 스마트 계약에 의해 L1에서 L2로 전송된 트랜잭션의 발신자는 L1 의 스마트 계약 주소가 될 수 없으며 L1 의 스마트 계약은 L2의 스마트 계약인 것처럼 작동할 수 있습니다. (두 계약이 동일한 주소를 공유하기 때문에)

이런 종류의 사칭을 방지하기 위해 스마트 계약에 의해 L1에서 L2로 거래가 전송될 때 거래 발신자는 약간 수정됩니다. 실제 L1 계약 주소에서 전송된 것처럼 보이는 대신, L2 거래는 L1 계약 주소의 "별칭" 버전에서 전송된 것처럼 보입니다. 이 별칭 주소는 실제 L1 계약 주소에서 일정한 오프셋이므로 별칭 주소는 L2의 다른 주소와 충돌하지 않으며 원래 L1 주소는 별칭 주소에서 쉽게 복구할 수 있습니다.

발신자 주소의 이 변경은 L1 스마트 계약에서 보낸 L2 거래에만 적용됩니다. 다른 모든 경우에는 거래 발신자 주소가 Ethereum에서 사용하는 것과 동일한 규칙에 따라 설정됩니다.

| Transaction Source                                      | Sender Address                                                     |
| ------------------------------------------------------- | ------------------------------------------------------------------ |
| L2 user (Externally Owned Account)                      | The user's address (same as in Ethereum)                           |
| L1 user (Externally Owned Account)                      | The user's address (same as in Ethereum)                           |
| L1 contract (using `OptimismPortal.depositTransaction`) | `L1_contract_address + 0x1111000000000000000000000000000000001111` |

## Transactions

#### Transaction Fees

Thanos 에서의 Transaction 은 Ethereum 에서 예상하는 표준 `Execution Gas Fee` 외에 `L1 Data Fee` 를 지불해야 합니다. 자세한 내용은 [Transaction Fee](transaction-fee.md) 가이드를 참조하세요.

#### EIP-1559 Parameters

Thanos 의 기본 수수료는 Ethereum 과 마찬가지로 [EIP-1559를 통해 계산됩니다.](https://notes.ethereum.org/@vbuterin/eip-1559-faq) Thanos 에서 사용하는 EIP-1559 매개변수는 다음과 같이 Ethereum에서 사용하는 매개변수와 다릅니다.

<table><thead><tr><th width="277">Parameter</th><th width="196">Thanos value</th><th>Ethereum value (for reference)</th></tr></thead><tbody><tr><td>Block gas limit</td><td>30,000,000 gas</td><td>30,000,000 gas</td></tr><tr><td>Block gas target</td><td>5,000,000 gas</td><td>15,000,000 gas</td></tr><tr><td>EIP-1559 elasticity multiplier</td><td>6</td><td>2</td></tr><tr><td>EIP-1559 denominator</td><td>250</td><td>8</td></tr><tr><td>Maximum base fee increase (per block)</td><td>2%</td><td>12.5%</td></tr><tr><td>Maximum base fee decrease (per block)</td><td>0.4%</td><td>12.5%</td></tr><tr><td>Block time in seconds</td><td>12</td><td>12</td></tr></tbody></table>

## Mempool Rules

Ethereum 과 달리 Thanos 에는 public mempool 이 없습니다. Thanos mempool 은 Sequencer 만 볼 수 있습니다. Sequencer 는 멤풀에서 priority fee(가장 높은 수수료가 먼저) 순서대로 트랜잭션을 실행합니다.
