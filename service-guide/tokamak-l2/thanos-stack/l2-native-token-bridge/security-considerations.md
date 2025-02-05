# Security Considerations

## General

Please review this guide and code comments thoroughly before using the L2 Native Token Bridge. Using the L2 Native Token Bridge without understanding its [requirements](https://www.notion.so/1-Overview-137d96a400a380ce9d38e73035fbeaca?pvs=21) could lead to unexpected behavior or loss of funds.

## L1 Withdrawal Failure Risk with EOA Target

{% hint style="danger" %}
**Important: EOA Addresses Do Not Support Contract Calls**

Ensure to verify the target address during withdrawals to avoid potential issues or asset loss. If `_tx.target` is an EOA, it can only process simple transfers. Attempting a contract call may result in asset loss.
{% endhint %}

Users must be careful when finalizing withdrawals from L1. When initializing a withdrawal transaction in L2, a user may lose funds if `_tx.target` is EOA and `_tx.data.length` is not 0. Since `_data` is non-empty, this transaction is interpreted as a contract call and not a simple transfer. However, if `_target` is the EOA, it will be a wallet address and that address cannot process any callback.

This can be found in `finalizeWithdrawalTransaction` of `OptimismPortal`([source](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/L1/OptimismPortal2.sol#L389-L414)):

* The token approval is completed by `approve` and `SafeCall.callWithMinGas` is executed for calling `transferFrom`. The withdrawal should be confirmed by the callback function but if the `_tx.target` is EOA, nothing happens. Therefore, `SafeCall.callWithMinGas` succeeds and the allowance is set back to zero. As a result, the user will never get the asset to withdraw.

```solidity
if (_tx.value != 0) {
    if (_tx.data.length != 0) {
        IERC20(_nativeTokenAddress).approve(_tx.target, _tx.value);
    } else {
        IERC20(_nativeTokenAddress).safeTransfer(_tx.target, _tx.value);
    }
}

bool success;
if (_tx.data.length != 0) {
    success = SafeCall.callWithMinGas(_tx.target, _tx.gasLimit, 0, _tx.data);
} else {
    success = true;
}

// Reset approval after a call
if (_tx.data.length != 0 && _tx.value != 0) {
    IERC20(_nativeTokenAddress).approve(_tx.target, 0);
}
```

## Dependency on ERC20 Approval

The L2 native token utilizes an ERC20 approval flow consisting of three steps:

1. Approving the ERC20 token
2. Executing an external call to transfer the token from the CrossDomainMessenger
3. Revoking the approval to prevent further access

```solidity
if (_value != 0 && _target != address(0)) {
    IERC20(_nativeTokenAddress).approve(_target, _value);
}
// _target is expected to perform a transferFrom to collect token
bool success = SafeCall.call(_target, gasleft() - RELAY_RESERVED_GAS, 0, _message);
if (_value != 0 && _target != address(0)) {
    IERC20(_nativeTokenAddress).approve(_target, 0);
}
```

If this process fails—for instance, when the external call encounters an error—tokens may become locked in the `CrossDomainMessenger`. Such a scenario could result in token loss for the user.
