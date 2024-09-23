# L2 fee

Titan에서 트랜잭션 수수료는 이더리움보다 저렴합니다. 이 문서에서는 Titan의 트랜잭션 수수료가 어떠한 방식으로 계산되는지 설명합니다.

Titan의 트랜잭션 수수료는 아래 두 가지의 합입니다.

* L2 execution fee
* L1 security fee

### L2 execution fee

```python
tx.gasPrice * l2GasUsed
```

* L2 execution fee는 사용자의 트랜잭션이 L2에서 실행될 때 부과되는 수수료로써, 위와 같이 계산되어 소모됩니다. 이 부분은 일반적인 이더리움 트랜잭션 수수료 계산 방식과 동일한데, Titan의 gas price가 이더리움에 비해 현저히 낮습니다.

### L1 security fee

```python
l1_base_fee * (tx_data_gas + overhaed) * scalar
```

* L1 security fee는 Optimistic Rollup의 특징으로 인해 필요한 수수료입니다. Optimistic Rollup에서는 L2의 Data availability를 위해 L2에서 발생하는 모든 트랜잭션에 대한 정보를 L1으로 올립니다. 이러한 Rollup 트랜잭션을 L1으로 보내기 위해 트랜잭션 수수료로 사용할 ETH를 L2의 트랜잭션 수수료에서 가져옵니다. Rollup 트랜잭션에는 모든 L2 트랜잭션들의 `calldata`가 포함되지만, L2의 트랜잭션들을 L1에서 직접 실행하지는 않기 때문에, 실행에 필요한 만큼의 `calldata` 만 수수료로 계산됩니다. 위에서 `tx_data_gas`로 표현된 것이 이 부분에 해당하며, 구체적으로는 아래와 같이 계산됩니다.

```python
tx_data_gas = count_zero_bytes(tx_data) * 4 + count_non_zero_bytes(tx_data) * 16
```

* `l1_base_fee`는 Rollup 트랜잭션을 위해 필요한 ETH를 계산하기 위해 사용되는 L1의 base fee 입니다. Titan은 주기적으로 이더리움의 base fee 변화량을 지켜보면서, 변화량이 클 때마다 새로운 `l1_base_fee`를 L2의 `OVM_GasPriceOracle` 컨트랙트에 저장합니다. `overhead`는 gas 단위로 계산되며, 작게 설정됩니다. `scalar`는 L1 security fee를 위해 곱해지는 가중치입니다.
