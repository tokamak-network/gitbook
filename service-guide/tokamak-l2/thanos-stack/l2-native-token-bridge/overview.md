# Overview

Thanos is an L2 optimistic rollup (fork of Optimism) where the L2 creator designates an ERC20 token as the native currency on the L2 instead of ETH. This ERC20 token is referred as L2 native token.

Because the L2 native token is the native currency:

* All transaction fees are paid with it.
* Any L2 contract using the native currency will reference the L2 native token. Existing contracts assuming the native currency is ETH will need to be updated to reflect this change.
* Messaging to L2 accepts L2 native token.
* L2 native token cannot be changed once L2 native token is initialized.

Although there are some differences in contracts, the backend remains largely unchanged to ensure compatibility with Optimism. The low-level APIs remain unchanged to allow for future implementation of new Optimism features.

The L2 Native Token Bridge has been changed to be based on Optimism's contract. The code base we used is shown below.

* [Optimism](https://github.com/ethereum-optimism/optimism/tree/v1.7.7) (OP Stack v1.7.7: Fjord Network Upgrade)
* [Thanos](https://github.com/tokamak-network/tokamak-thanos/tree/main)

## Difference between L2 Native Token Bridge & Optimism’s Custom Gas Token

Please note that Thanos’ L2 Native Token Bridge is different than the [Custom Gas Token](https://specs.optimism.io/experimental/custom-gas-token.html) developed by Optimism. Thanos focuses on supporting ERC20 token as a native currency without supporting ETH as native currency.

**Unique** features in L2 Native Token Bridge compared to Custom Gas Token:

* Deposit & withdraw L2 native token is supported using `L1StandardBridge`, `L2StandardBridge`
* Have the features the `StandardBridge` and `CrossDomainMessenger` like replayability.
* Send messages and relay messages with non-zero native token in `L1CrossDomainMessenger`
* Allow deposit & withdraw ETH to `StandardBridge` or `CrossDomainMessenger`. Custom Gas Token not support them. It must revert when ETH is sent from the StandardBridge or `CrossDomainMessenger`.

**Unsupported** features in L2 Native Token Bridge compared to Custom Gas Token:

* ETH cannot be set as L2 native token
* However, WETH (wrapped ETH) is an ERC20 token that can be used as an L2 native token.

## L2 native token requirements

We can use custom ERC20 token as a native token in Thanos. **It is called L2 native token.** Not all tokens can be chosen as the L2 native token. For a token to be used as an L2 native token, the corresponding L1 token MUST satisfy the ERC20 standard and following additional requirements:

* Has 18 decimals. The L2 native token has 18 decimals, so corresponding L1 token must have exactly 18 decimals to ensure no loss of precision when depositing or withdrawing.
* Must not have fees on transfer.
* Must not have call hooks on transfer.
* Must not have out of band methods for modifying balance or allowance. For example, no tokes that have rebasing logic or [double entrypoint](https://stermi.xyz/blog/ethernaut-challenge-24-solution-double-entry-point) can be a L2 native token.
* Must not have pausable function.
* Must not revert `transfer()` or `approve()` function unexpectedly.
  * Has `transfer()` function should consume appropriate gas to prevent potential out-of-gas reverts during `relayMessage()` execution.
  * Has `approve()` function must be gas-efficient to ensure that approval operations will not cause unexpected reverts due to gas limits.
* Any other requirements set by standard bridge that is not mentioned here.

## Constants

Following constants will be used in the rest of this document.

| Name                        | Value                                                 | Description                                 |
| --------------------------- | ----------------------------------------------------- | ------------------------------------------- |
| `LEGACY_ERC20_NATIVE_TOKEN` | `address(0xDeadDeAddeAddEAddeadDEaDDEAdDeaDDeAD0000)` | “Virtual address” of the native token on L2 |
| `ETH`                       | `address(0x4200000000000000000000000000000000000486)` | Predeploy contract of ETH on L2             |

* `LEGACY_ERC20_NATIVE_TOKEN` is renamed from Optimism’s `LEGACY_ERC20_ETH`
* “Virtual address” means the address is empty. However, for backward compatibility, `LEGACY_ERC20_NATIVE_TOKEN` is needed when withdrawing the native token.
* These constants need to be defined in the [Predeploys.sol](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/libraries/Predeploys.sol).

## Tokens

### ETH (Official ERC20 version of ETH on L2)

To improve compatibility with existing OP stack backend, L2 native token’s address uses the address originally allocated for ETH. To support competitive gas price that can be trusted, we need a new address to support ETH.

* This is a new ERC20 smart contract for representing ETH on L2. Users will receive this token when they deposit ETH on L1
* This contract is inherited from `OptimismMintableERC20`

```jsx
// SPDX-License-Identifier: MIT
pragma solidity 0.8.15;

import { Predeploys } from "src/libraries/Predeploys.sol";
import { OptimismMintableERC20 } from "src/universal/OptimismMintableERC20.sol";

/// @custom:proxied
/// @custom:predeploy 0x4200000000000000000000000000000000000486
/// @title ETH
/// @notice ETH is a contract that Wrap ETH
contract ETH is OptimismMintableERC20 {
    /// @notice Initializes the contract as an Optimism Mintable ERC20.
    constructor() OptimismMintableERC20(Predeploys.L2_STANDARD_BRIDGE, address(0), "Ether", "ETH", 18) { }
}
```

### WNativeToken (Wrapped Native Token)

This contract is renamed from Optimism’s `WETH`. Because on Thanos, ETH is not the native token so this contract is renamed to `WNativeToken`(ERC20) for wrapping native token on L2.

* The token decimals is 18.
* We need a `WNativeToken` to use L2 native token like an ERC20 token in DeFi protocol and utilized when interacting with smart contracts requiring the ERC20
* It supports several functions such as `approve`, `transfer`, `transferFrom` according to the ERC20.
* It is defined in the [WNativeToken.sol](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/vendor/WNativeToken.sol)
