# Transaction Fee

Thanos Network 에서 네트워크에 제출하기 전에 거래 비용을 적절히 계산하는 것이 중요합니다. 여기에서는 Thanos Network 거래의 총 비용을 구성하는 두 가지 구성 요소인 **Execution Gas Fee** 와  **L1 Data Fee** 를 계산하는 방법을 알아봅니다.&#x20;

Thanos 의 거래 수수료는 다음의 두 구성 요소를 더해서 결정됩니다.

* Execution Gas Fee
* L1 Data Fee

## Execution Gas Fee

Execution Gas Fee는 이더리움에서 동일한 거래에 대해 지불하는 수수료와 정확히 같습니다. 이 수수료는 트랜잭션에서 사용된 gas 에 가스 가격을 곱한 값과 같습니다. 이더리움과 마찬가지로 Thanos Network 는 EIP-1559를 사용합니다. EIP-1559 에서 트랜잭션에서 지불하는 단위 가스당 가격은 `base_fee` 입니다. 그리고 추가적으로 `priority_fee` 를 설정하면 단위 가스당 가격은 `base_fee + priority_fee` 가 됩니다.

Thanos Network 에서 트랜잭션에 사용되는 가스의 양은 Ethereum 에서 동일한 트랜잭션에 사용되는 가스의 양과 정확히 같습니다. Ethereum 에서 트랜잭션 비용이 100,000 gas 이면 Thanos Network 에서도 100,000 gas 가 듭니다.

이더리움에서 거래 비용을 추정하는 데 사용하는 것과 동일한 도구를 사용하여 거래의 총 비용을 추정할 수 있습니다. 이더리움의 가스 수수료가 어떻게 작동하는지에 대한 자세한 내용은 [Ethereum.org](https://ethereum.org/en/developers/docs/gas/) 에서 읽을 수 있습니다.

## L1 Data Fee

L1 Data Fee 는 Thanos Transaction 수수료 중 Ethereum Transaction 수수료와 다른 부분입니다. 이 수수료는 모든 Thanos Transaction에 대한 data가 Ethereum 에 게시된다는 사실에서 발생합니다. 이를 통해 개별 Node 에서 Transaction data 를 다운로드하여 실행할 수 있습니다. L1 Data Fee 는 Ethereum 의 현재 가스비에 따라 결정되며 Blob을 지원할 경우 현재 Ethereum Blob 데이터 가스 가격에 의해 결정됩니다.

#### Mechanism <a href="#mechanism" id="mechanism"></a>

L1 Data Fee는 Thanos 블록에 포함된 모든 거래에 대해 자동으로 청구됩니다. 이 수수료는 거래를 보낸 주소에서 직접 공제됩니다. 지불되는 정확한 금액은 압축 후 바이트 단위의 거래 추정 크기, 현재 Ethereum 가스 가격 및/또는 Blob 가스 가격, 그리고 여러 가지 매개변수에 따라 달라집니다.

L1 Data Fee는 Ethereum 에서 Thanos 로 지속적이고 신뢰할 수 없이 전달되는 Ethereum `base fee` 에 가장 큰 영향을 받습니다. Ethereum `blob base fee` 도 Thanos 로 전달되고 tx.data 대신 blob 을 사용하도록 구성된 체인의 가장 중요한 요소가 됩니다. `base fee` 와 `blob base fee` 는 모든 Ethereum 블록에 대해 Thanos 에서 업데이트 되며 각각 업데이트 사이에 최대 12.5% ​​변동합니다. 결과적으로 L1 Data Fee의 단기 변동은 일반적으로 매우 작으며 평균 거래에 영향을 미치지 않습니다.

#### Formula

서명된 거래의 FastLZ 압축 크기입니다. 현재 이더리움 기본 수수료 및/또는 블롭 기본 수수료(이더리움에서 신뢰할 수 없이 전달됨). L1 Data Fee의 계산은 FastLZ 로 압축된 거래 크기에 대한 선형 모델을 사용하여 거래 크기를 추정하는 것으로 시작됩니다.

`estimatedSizeScaled = max(minTransactionSize * 1e6, intercept + fastlzCoef * fastlzSize)`

모델 매개변수 intercept 및 fastlzCoef 는 Brotli 로 압축했을 때 배치 크기 변경에 대한 평균 제곱근 오차를 최소화하는 이전 L2 거래 데이터 세트에 대한 선형 회귀 분석을 수행하여 결정되었습니다. 이러한 매개변수는 고정됩니다. ([link](https://github.com/tokamak-network/tokamak-thanos-geth/blob/main/core/types/rollup\_cost.go#L51))

다음으로, 두 개의 체인 매개변수 baseFeeScalar 와 blobFeeScalar 를 사용하여 `l1FeeScaled` 를 계산합니다.

`l1FeeScaled = baseFeeScalar * l1BaseFee * 16 + blobFeeScalar * l1BlobBaseFee`

두 스칼라는 모두 1e6으로 스케일됩니다. 최종 L1 Data Fee는 다음과 같습니다.

`l1Cost = estimatedSizeScaled * l1FeeScaled / 1e12`

