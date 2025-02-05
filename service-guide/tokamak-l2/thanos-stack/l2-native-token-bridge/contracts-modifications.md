# Contracts Modifications

This section describes the contract modifications based on Optimism's Custom Gas Token to enable ERC-20 assets to function as native tokens on Layer 2. Thanos’ L2 Native Token Bridge allows seamless gas fee payments and transaction handling with tokens other than ETH to expand users' flexibility and utility. Through this enhancement, users can interact with L2 without ETH to expand token utility within the ecosystem.

Below are some points to keep in mind about the Optimism smart contracts’s design.

* `OptimismPortal` contract provides low level APIs for communication between L1 and L2. It is implemented as proxy and connected to `OptimismPortal2` that supported fault proof.
* `L1CrossDomainMessenger` contract handles cross-chain messages.
* `L1StandardBridge` and `L2StandardBridge` contracts provide high-level APIs for bridging tokens cross-chain.
* The L1 token when depositing, every kind of supported tokens are locked in `L1StandardBridge` except L2 native token is locked in `OptimismPortal`

The L2 Native Token Bridge incorporates additional asset transfer functionality, including `transferFrom` and `approve` logic, to enable withdrawals of ERC20-based native tokens. It also enhances message relay logic for better ERC20 compatibility and addresses potential security vulnerabilities in relaying cross-chain message.

In this section, we'll compare the differences between Optimism's Custom Gas Token and Thanos’ L2 Native Token Bridge. For ease of description, we'll use the following tags.

* **\[New]**: New features in L2 Native Token Bridge
* **\[Modified]**: Modifications in L2 Native Token Bridge compared to Optimism's Custom Gas Token
* **\[Deleted]**: Features removed from the L2 Native Token Bridge that were present in Optimism's Custom Gas Token

### SystemConfig

In Thanos, the L1 token address of the corresponding L2 native token, called `nativeTokenAddress`, is added to the storage. This signals to other contracts that this L1 token will serve as the native currency on L2.

Following contracts are updated to be aware of L2 native token using the `nativeTokenAddress`:

* `OptimismPortal`
* `L1CrossDomainMessenger`
* `L1StandardBridge`

#### **\[New]** `nativeTokenAddress()`

`nativeTokenAddress` is a new view function that returns the L1 token address corresponding to the L2 native token. `OptimismPortal`, `L1CrossDomainMessenger` and `L1StandardBridge` uses this to be aware of L2 native token.

```solidity
function nativeTokenAddress() public view returns (address) {
    return systemConfig.nativeTokenAddress();
}
```

### OptimismPortal

#### **\[Modified]** `receive`

This function MUST revert because depositing ETH is not supported; only deposits of the L2 native token are allowed.

#### **\[Modified]** `depositTransaction`

Optimism’s Custom Gas Token uses `depositTransaction` to send a message to L2 with ETH, and `msg.value` determines how much ETH to mint on L2. On the other hand, L2 Native Token Bridge only supports L2 native tokens, so a separate calldata `_mint` is used to specify how much L2 native token to mint on L2.

The differences in the interface are shown below.

*   Optimism

    ```solidity
    function depositTransaction(
        address _to,
        uint256 _value,
        uint64 _gasLimit,
        bool _isCreation,
        bytes memory _data
    )
        public
        payable
        metered(_gasLimit)
    ```
*   Thanos

    ```solidity
    function depositTransaction(
    	  address _to,
    	  uint256 _mint, // the mint amount to deposit
    	  uint256 _value,
    	  uint64 _gasLimit,
    	  bool _isCreation,
    	  bytes calldata _data
    )
    	  external
    ```

    *   Parameters

        | Name          | Type      | Description                                                                                       |
        | ------------- | --------- | ------------------------------------------------------------------------------------------------- |
        | `_to`         | `address` | The target of the deposit transaction                                                             |
        | `_mint`       | `uint256` | The amount of token to deposit                                                                    |
        | `_value`      | `uint256` | The value of the deposit transaction, used to transfer native asset that is already on L2 from L1 |
        | `_gasLimit`   | `uint64`  | The gas limit of the deposit transaction                                                          |
        | `_isCreation` | `bool`    | Signifies the `_data` should be used with `CREATE`                                                |
        | `_data`       | `bytes`   | The calldata of the deposit transaction                                                           |

This function is is not payable and It cannot be called inside the `OptimismPortal`, so this function should be declared as an external function in Thanos. It does not include the modifier `metered(gasLimit)`_,_ however the logic of `metered*(*gasLimit)` MUST be kept in the internal `_depositTransaction`() function.

To deposit L2 native tokens, users MUST first `approve` the address of `OptimismPortal` so that `OptimismPortal` can collect users‘ L2 native token. And it sends L2 native token from the `_sender` address to the `OptimismPortal`**.**

```solidity
if (_mint > 0) {
    IERC20(nativeTokenAddress).safeTransferFrom(_sender, address(this), _mint);
    depositedAmount += _mint;
}
```

#### **\[Modified]** `finalizeWithdrawalTransaction`

It MUST call `approve` to allow the target to transfer the L2 native token. The target MUST then use `transferFrom` to receive the withdrawn token from `OptimismPortal`.

* If `_tx.data.length != 0`, the `approve` function is called to grant `_tx.target` permission to access the amount specified by `_tx.value`. Otherwise, if `_tx.data.length == 0`, the token is directly transferred to `_tx.target` using `safeTransfer`.

```solidity
// Token Transfer
if (_tx.value != 0) {
    if (_tx.data.length != 0) {
        IERC20(_nativeTokenAddress).approve(_tx.target, _tx.value);
    } else {
        IERC20(_nativeTokenAddress).safeTransfer(_tx.target, _tx.value);
    }
}
// Call external contract
bool success;
if (_tx.data.length != 0) {
    success = SafeCall.callWithMinGas(_tx.target, _tx.gasLimit, 0, _tx.data);
```

The target address must not be the same as the L2 native token's address.

* If `_tx.data.length != 0` and `_tx.value` is non-zero, the token allowance is reset to 0, removing any remaining allowance. This security measure prevents potential vulnerabilities. It prevents spenders from using the same allowance multiple times or exploiting reentrancy attacks to gain additional benefits.

```solidity
// Reset approval after a call
if (_tx.data.length != 0 && _tx.value != 0) {
    IERC20(_nativeTokenAddress).approve(_tx.target, 0);
}
```

#### \[**New]** `onApprove`

This function is a callback that is triggered when an ERC20 token's `approveAndCall` that extends the functionality of ERC20 tokens, allowing certain logic to be executed simultaneously with the token's allowance. This feature is primarily used to simplify interactions with smart contracts and improve the user experience. `onApprove` is used to perform token transfers or handle specific tasks. It is only used with a chosen L2 native token that supports `approveAndCall` (e.g., [TON](https://etherscan.io/token/0x2be5e8c109e2197d077d13a82daead6a9b3433c5))

```solidity
function onApprove(
    address _owner,
    address,
    uint256 _amount,
    bytes calldata _data
)
    external
    override
    returns (bool)
```

*   Parameters:

    | Name      | Type      | Description                                         |
    | --------- | --------- | --------------------------------------------------- |
    | `_owner`  | `address` | address of the account that called `approveAndCall` |
    | `_amount` | `uint256` | The amount of token to deposit                      |
    | `_data`   | `bytes`   | The calldata of the deposit transaction             |

#### \[New] `unpackOnApproveData`

This function decodes a bytes calldata input into its predefined structure using low-level assembly. It validates the input length and efficiently extracts the required components: target address, value, gas limit, and message.

```solidity
function unpackOnApproveData(bytes calldata _data)
    internal
    pure
    returns (address _to, uint256 _value, uint32 _gasLimit, bytes calldata _message)
```

* Parameter: The function takes a single parameter `_data`, which is a bytes calldata.
* Return: It returns four values
  * `_to`: The target address.
  * `_value`: The value to be used in the transaction.
  * `_gasLimit`: The gas limit for the transaction.
  * `_message`: The remaining data to be used as a message.
* Data Layout:
  * The first 20 bytes represent the target address (`_to`).
  * The next 32 bytes represent the value (`_value`).
  * The following 4 bytes represent the gas limit (`_gasLimit`).
  * The rest of the data is the message (`_message`)

### CrossDomainMessenger

#### \[New] `sendNativeTokenMessage`

This function is nearly identical to `sendMessage`, except it works with the L2 native token instead of ETH. The interface differences are outlined below.

*   Optimism

    ```jsx
        function sendMessage(
            address _target,
            bytes calldata _message,
            uint32 _minGasLimit
         ) external payable
    ```
*   Thanos

    ```
        function sendNativeTokenMessage(
            address _target,
            uint256 _amount,
            bytes calldata _message,
            uint32 _minGasLimit
        ) external
    ```

    *   Parameters

        | Name           | Type      | Description                                              |
        | -------------- | --------- | -------------------------------------------------------- |
        | `_target`      | `address` | address of the account that called `approveAndCall`      |
        | `_amount`      | `uint256` | The amount of token to deposit                           |
        | `_message`     | `bytes`   | The calldata of the deposit transaction                  |
        | `_minGasLimit` | `uint32`  | Minimum gas limit that the message can be executed with. |
* This function MUST not be payable. Senders must first `approve` the `L1CrossDomainMessenger` address, allowing it to use `transferFrom` to collect the L2 native token.
*   `L1CrossDomainMessenger` MUST approve address of the OptimismPortal, so that the `OptimismPortal` can use `transferFrom` to collect `L1CrossDomainMessenger`’s L2 native token when `depositTransaction` is called.

    ```solidity
    if (_amount > 0) {
        address _nativeTokenAddress = nativeTokenAddress();
        IERC20(_nativeTokenAddress).safeTransferFrom(_sender, address(this), _amount);
        IERC20(_nativeTokenAddress).approve(address(portal), _amount);
    }
    ```

#### \[Modified] `relayMessage`

This function securely relays messages, ensuring they are not replayed, have sufficient gas, and are sent to safe target addresses. The target MUST NOT be the address of the L2 native token and MUST revert if `CALLVALUE` is not zero.

It verifies whether the message originates from the other messenger. If so, it confirms the message hasn't previously failed and transfers the value if necessary. If not, it ensures the message can be replayed.

```solidity
if (_isOtherMessenger()) {
    // These properties should always hold when the message is first submitted (as
    // opposed to being replayed).
    assert(!failedMessages[versionedHash]);
    if (_value > 0) {
        IERC20(_nativeTokenAddress).safeTransferFrom(address(portal), address(this), _value);
    }
} else {
    require(failedMessages[versionedHash], "CrossDomainMessenger: message cannot be replayed");
}
```

It MUST include logic to `approve` the target, allowing the target to use `transferFrom` to collect the L2 native token from `L1CrossDomainMessenger` when executing the message.

*   Thanos

    ```solidity
    // _target must not be address(0). otherwise, this transaction could be reverted
    if (_value != 0 && _target != address(0)) {
        IERC20(_nativeTokenAddress).approve(_target, _value);
    }
    // _target is expected to perform a transferFrom to collect token
    bool success = SafeCall.call(_target, gasleft() - RELAY_RESERVED_GAS, 0, _message);
    if (_value != 0 && _target != address(0)) {
        IERC20(_nativeTokenAddress).approve(_target, 0);
    }
    ```
*   Optimism

    ```solidity
    bool success = SafeCall.call(_target, gasleft() - RELAY_RESERVED_GAS, _value, _message);
    ```

#### \[New]`onApprove`

This function is only used with a chosen L2 native token that supports `approveAndCall`. Users can conveniently use `sendNativeTokenMessage` by leverage this function.

```solidity
    function onApprove(
        address _owner,
        address,
        uint256 _amount,
        bytes calldata _data
    )
        external
        override
        returns (bool)
```

*   Parameters:

    | Name      | Type      | Description                                         |
    | --------- | --------- | --------------------------------------------------- |
    | `_owner`  | `address` | address of the account that called `approveAndCall` |
    | `_amount` | `uint256` | The amount of token to deposit                      |
    | `_data`   | `bytes`   | The calldata of the deposit transaction             |

#### \[New] `unpackOnApproveData`

This function efficiently unpacks and retrieves structured data from a bytes calldata input. It's particularly useful for handling complex data structures passed as a single bytes parameter, such as in the `approveAndCall` pattern.

```solidity
function unpackOnApproveData(bytes calldata _data)
    internal
    pure
    returns (address _to, uint32 _minGasLimit, bytes calldata _message)
```

* Parameter: a single parameter `_data`, which is a bytes calldata.
* Return: It returns three values:
  * `_to`: The target address to which the message is intended.
  * `_minGasLimit`: The minimum gas limit required for executing the message.
  * `_message`: The remaining data to be used as a message.
* **Data Layout**:
  * The first 20 bytes represent the target address (`_to`).
  * The next 4 bytes represent the minimum gas limit (`_minGasLimit`).
  * The rest of the data is the message (`_message`).

### StandardBridge

#### \[New] `bridgeNativeToken`

This function is responsible to send L2 native token to the sender’s address on the other layer. The sender MUST be an EOA (Externally Owned Account). In `L1StandardBridge`, Users have to approve the address of `L1CrossDomainMessenger` first.

```solidity
function bridgeNativeToken(
    uint256 _amount,
    uint32 _minGasLimit,
    bytes calldata _extraData
)
    public
    payable
    onlyEOA
```

* Parameters:

| Name           | Type      | Description                                |
| -------------- | --------- | ------------------------------------------ |
| `_amount`      | `uint256` | amount of deposited L2 native address      |
| `_minGasLimit` | `uint32`  | minimum gas limit for bridging other layer |
| `_extraData`   | `bytes`   | optional data to forward to L2             |

#### \[New] `finalizeBridgeNativeToken`

This function is used for finalizing withdrawal native token messages from other layer. The message for calling this function MUST be triggered from other bridge.

```solidity
function finalizeBridgeNativeToken(
    address _from,
    address _to,
    uint256 _amount,
    bytes calldata _extraData
)
    public
    override
    onlyOtherBridge
```

* Parameters

| Name         | Type      | Description                                   |
| ------------ | --------- | --------------------------------------------- |
| `_from`      | `address` | address of sender                             |
| `_to`        | `address` | address of receiver                           |
| `_amount`    | `uint256` | amount of L2 native token being bridged       |
| `_extraData` | `bytes`   | optional data to be sent with the transaction |

* Difference between L1 and L2 bridge: Token Transfer
  * L1: Uses `safeTransferFrom` and `safeTransfer` for token movement, which is necessary for ERC20 tokens
  * L2: Uses `SafeCall.call` to handle the transfer of native tokens.

#### \[New] `onApprove` & `unpackOnApproveData`

* These functions are only used with L2 native token that supports `approveAndCall`. There are Implemented with the same logic as `CrossDomainMessenger`.

#### \[New] `withdrawNativeToken`

This function initiates a withdrawal L2 native token form L2 to L1 in `L2StandardBridge`. Senders of this function must be EOA.

```solidity
function withdrawNativeToken(
    uint32 _minGasLimit,
    bytes calldata _extraData
) external payable onlyEOA
```

* Parameters

| Name           | Type     | Description                                                    |
| -------------- | -------- | -------------------------------------------------------------- |
| `_minGasLimit` | `uint32` | minimum gas limit for relaying this message successfully on L1 |
| `_extraData`   | `bytes`  | extra data that attached to the withdrawal transaction         |

#### \[Modified]`receive`

Users can deposit ETH by sending it directly to the `L1StandardBridge`. To withdraw L2 native tokens, they should send them to the `L2StandardBridge`. In both cases, senders MUST be externally owned accounts (EOAs).

#### \[Modified] `bridgeETH`

This function is responsible for initiating the bridging of ETH between layers. Senders MUST be EOA.

* Difference between L1 and L2 bridge: Value Handling
  * L1: Checks that msg.value is non-zero and matches the `_amount`, updating the deposit balance.
  * L2: Burns the ETH from the sender's balance using the `OptimismMintableERC20` interface.

#### \[Modified] `bridgeERC20`

This function sends ERC20 tokens to a receiver's address on the other layer. It can execute with `msg.value` set to zero, as it is designed to handle ERC20 tokens and does not involve any ETH specific logic. It MUST revert if using L2 native token. Senders MUST be EOA.

#### \[Modified] `finalizeBridgeETH`

This function is responsible for finalizing the bridging of ETH from one chain to another.

* Difference between L1 and L2 bridge: Transfer vs Minting
  * L1: Transfers ETH using `SafeCall.call`, ensuring the recipient receives the ETH directly.
  * L2: Mints `OptimismMintableERC20` tokens representing ETH to the recipient, as L2 typically uses token representations for assets.

#### \[Modified] `finalizeBridgeERC20`

This function is responsible for completing the bridging process of ERC20 tokens from one chain to another. So `_localToken` MUST not be the address of ETH or the address of an ERC20 token that does not follow `OptimismMintableERC20`.

#### \[Modified] `withdraw`

This function initiates the withdrawal of assets from L2 to L1. L2 Native token Bridge distinguishes between L2 native token and ERC20 and includes additional validation on `msg.value` for more secure usage. The caller of this function must be an EOA.

#### \[Deleted] `depositETH`, `depositETHTo`, `depositERC20`, `depositERC20To`

These functions are maintained for backward compatibility with high-level APIs by the Optimism team, ensuring that existing dApps on Optimism don't require updates when the chain is upgraded. However, as Thanos is a new chain without any running dApps, there's no need to retain these functions.
