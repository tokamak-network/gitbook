# Deposit & Withdraw Flow

This section describes the differences between depositing and withdrawing assets on Thanos' L2 Native Token Bridge compared to Optimism's bridge design. L2 Native Token Bridge supports deposits and withdrawals of ETH, L2 native tokens, and ERC20 assets that are not L2 native tokens. Understanding these processes is essential for smooth operations within the L2 environment, as they are the foundation for cross-layer interoperability.

`L1StandardBridge` and `L2StandardBridge` are smart contracts used for depositing and withdrawing tokens. They're responsible for initiating and finalizing the bridge process and can be modified based on specific functionality requirements. The following section details the flow of function calls and token movements when users deposit or withdraw tokens using the `L1StandardBridge` and `L2StandardBridge` contracts.

## Deposit flows

The process begins with a deposit request on L1, which the bridge processes to ensure a secure and synchronized asset transfer to L2. Understanding each step of this deposit process is crucial for enhancing user experience and maintaining network security.

{% hint style="warning" %}
In the finalization process with `L1StandardBridge`, the `op-node` takes on the role of delivering messages from L1 to L2, acting on behalf of `L1CrossDomainMessenger`. Actions by services or users that are not calls between contracts are shown with a dotted line.
{% endhint %}

#### **Deposit ETH**

The deposit flow of ETH differs from Optimism's `depositETH` where:

* ETH is locked in the `L1StandardBridge` instead of `OptimismPortal`**.**
* On the Thanos, users receive ERC20 token that represents ETH instead of native ETH as in Optimism.

<figure><img src="../../../../.gitbook/assets/Screenshot from 2024-06-28 10-44-34.png" alt=""><figcaption></figcaption></figure>

#### **Deposit ERC20 (not L2 native token)**

The flow is the same as Optimism’s `depositERC20`, except it does not allow using L2 Native Token.

<figure><img src="../../../../.gitbook/assets/Screenshot from 2024-06-28 10-44-47.png" alt=""><figcaption></figcaption></figure>

#### **Deposit L2 native token**

The L2 native token must be locked in `OptimismPortal`. And the flow of deposit L2 native token is almost the same as Optimism’s `depositETH` except using some other function interfaces (but provides the same logic) and an ERC20 is used instead of ETH. Transferring token by using the function `transferFrom` in the destination contracts/EOAs.

<figure><img src="../../../../.gitbook/assets/Screenshot from 2024-06-28 10-59-19.png" alt=""><figcaption></figcaption></figure>

## Withdraw flows

This flow outlines the process of transferring assets from Layer 2 (L2) back to Layer 1 (L1). It provides a step-by-step breakdown, from initiating the withdrawal on L2 to finalizing it on L1, offering users a comprehensive understanding of the cross-layer asset return process.

#### **Withdraw ETH**

Compare to Optimism’s `withdraw`:

* L2’s ETH is an ERC20 instead of native ETH as in Optimism.
* Users will receive ETH from `L1StandardBridge` instead of `OptimismPortal` as in Optimism. On Thanos, ETH is also locked at `L1StandardBridge` when depositing ETH.

<figure><img src="../../../../.gitbook/assets/Screenshot from 2024-06-28 10-45-16.png" alt=""><figcaption></figcaption></figure>

#### **Withdraw ERC20**

The process mirrors Optimism's, with one crucial distinction: the L2 native token cannot be withdrawn through this mechanism.

<figure><img src="../../../../.gitbook/assets/Screenshot from 2024-06-28 10-45-26.png" alt=""><figcaption></figcaption></figure>

#### **Withdraw L2 native token**

The process is nearly identical to withdrawing ETH on Optimism. The key difference is that on Thanos, users will receive the L2 Native Token instead of ETH.



<figure><img src="../../../../.gitbook/assets/Screenshot from 2024-06-28 10-45-04.png" alt=""><figcaption></figcaption></figure>
