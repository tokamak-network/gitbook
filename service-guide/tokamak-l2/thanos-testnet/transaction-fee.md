# Transaction Fee

It is important to correctly calculate transaction costs before submitting them to the Thanos Network. This section explains how to calculate the **Execution Gas Fee** and **L1 Data Fee**, two components that make up the total cost of a Thanos Network transaction.

Transaction fees in Thanos are determined by adding the following two components:

* Execution Gas Fee
* L1 Data Fee

## Execution Gas Fee

The Execution Gas Fee is exactly the same as the fee paid for the same transaction on Ethereum. This fee is calculated as the amount of gas used by the transaction multiplied by the gas price. Like Ethereum, the Thanos Network uses EIP-1559. In EIP-1559, the price paid per unit of gas in a transaction is `base_fee`. If a `priority_fee` is added, the price per unit of gas becomes `base_fee + priority_fee`.

The amount of gas used for a transaction in Thanos is exactly the same as the amount of gas used for the same transaction in Ethereum. If a transaction costs 100,000 gas on Ethereum, it also costs 100,000 gas on Thanos Network.

So you can use the same tools that estimate transaction costs on Ethereum to estimate the total cost of the transaction. You can read more about how gas fees work on Ethereum at [Ethereum.org](https://ethereum.org/en/developers/docs/gas/).

## L1 Data Fee

The L1 Data Fee is a key difference of the Thanos transaction fee compared to the Ethereum transaction fee. This fee is derived from the fact that data for all Thanos transactions is submitted on Ethereum. This allows individual nodes to download and execute the transaction data. The L1 Data Fee is determined by the current gas fees on Ethereum, and if Blob support is enabled, it will be based on the current Ethereum Blob data gas price.

#### Mechanism <a href="#mechanism" id="mechanism"></a>

The L1 Data Fee is automatically charged for all transactions included in a Thanos block. This fee is directly deducted from the address sending the transaction. The exact amount paid depends on the estimated size of the transaction in bytes after compression, the current Ethereum gas price and/or blob gas price, and several other parameters.

The L1 Data Fee is most affected by the Ethereum `base fee`, which is continuously and trustlessly relayed to Thanos from Ethereum. The Ethereum `blob base fee` also influences the fee when the chain is configured to use blobs instead of tx.data. The `base fee` and `blob base fee` are updated in Thanos for each Ethereum block and can vary by up to 12.5% between updates. As a result, short-term variations in the L1 Data Fee are typically very small and do not significantly affect average transactions.

#### Formula

It is the FastLZ compressed size of the signed transaction. This calculation considers the current Ethereum base fee and/or blob base fee (relayed trustlessly from Ethereum). The calculation of the L1 Data Fee begins by estimating the transaction size using a linear model based on the size of the transaction compressed with FastLZ.

`estimatedSizeScaled = max(minTransactionSize * 1e6, intercept + fastlzCoef * fastlzSize)`

The model parameters, intercept and fastlzCoef, were determined through a linear regression analysis on previous L2 transaction datasets to minimize the root mean square error of batch size changes when compressed with Brotli. These parameters are fixed. ([link](https://github.com/tokamak-network/tokamak-thanos-geth/blob/main/core/types/rollup\_cost.go#L51))

Next, two chain parameters, baseFeeScalar and blobFeeScalar, are used to calculate `l1FeeScaled`:

`l1FeeScaled = baseFeeScalar * l1BaseFee * 16 + blobFeeScalar * l1BlobBaseFee`

Both scalars are scaled by 1e6. The final L1 Data Fee is then:

`l1Cost = estimatedSizeScaled * l1FeeScaled / 1e12`
