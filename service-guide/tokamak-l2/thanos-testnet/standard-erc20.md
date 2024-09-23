# Standard ERC20

## Bridging Your Standard ERC-20 Token Using the Standard Bridge

Learn how to bridge ERC-20 tokens from Ethereum to Thanos using the Standard Bridge system. This tutorial is intended for developers who have an existing ERC-20 token on Ethereum and want to create a bridged representation of that token on Thanos.

This tutorial explains how to use the [`OptimismMintableERC20Factory`](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/universal/OptimismMintableERC20Factory.sol) to deploy standardized ERC-20 tokens on Thanos. Tokens created with this factory contract are compatible with the Standard Bridge system and include basic logic for deposits, transfers, and withdrawals. If you need to include custom logic in your L2 token, refer to the tutorial on [Bridging Your Custom ERC-20 Token Using the Standard Bridge](standard-erc20.md#bridging-your-custom-erc-20-token-using-the-standard-bridge) instead.

#### Prerequisite

* [foundry](https://book.getfoundry.sh/getting-started/installation)

#### L1 ERC-20 Token Address

For this tutorial, you will need an L1 ERC-20 token. If you already have an L1 ERC-20 token deployed on Sepolia, you can skip this step. Otherwise, you can use a test token available at [`0x5589BB8228C07c4e15558875fAf2B859f678d129`](https://sepolia.etherscan.io/address/0x5589BB8228C07c4e15558875fAf2B859f678d129), which includes a function that allows you to mint tokens.

#### Creating an L2 ERC-20 Token

Once you have the L1 ERC-20 token, you can use the [OptimismMintableERC20Factory](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/universal/OptimismMintableERC20Factory.sol) to create the corresponding L2 ERC-20 token on Thanos. All tokens created from the factory implement the `IOptimismMintableERC20` interface and are compatible with the Standard Bridge system.

1. Add Private Key to Environment Variables

```bash
export PRIVATE_KEY=0x...
```

2. Add RPC to Environment Variables

```bash
export TUTORIAL_RPC_URL=https://rpc.thanos-sepolia.tokamak.network
```

3. Add L1 ERC-20 Address to Environment Variables

```bash
# Replace this with your L1 ERC-20 token if not using the testing token!
export TUTORIAL_L1_ERC20_ADDRESS=0x5589BB8228C07c4e15558875fAf2B859f678d129
```

4. Deploy L2 ERC-20

```bash
cast send 0x4200000000000000000000000000000000000012 "createOptimismMintableERC20(address,string,string)" $TUTORIAL_L1_ERC20_ADDRESS "My Standard Demo Token" "L2TKN" --private-key $PRIVATE_KEY --rpc-url $TUTORIAL_RPC_URL --json | jq -r '.logs[0].topics[2]' | cast parse-bytes32-address
```

## Bridging Your Custom ERC-20 Token Using the Standard Bridge

Learn how to bridge ERC-20 tokens from Ethereum to Thanos using the Standard Bridge system. This tutorial is for developers who have an existing ERC-20 token on Ethereum and want to create a bridged representation of that token on Thanos.

In this tutorial, we will explain how to create a custom token and provide an interface that allows it to be used with the [`IOptimismMintableERC20`](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/universal/IOptimismMintableERC20.sol) Standard Bridge system. Custom tokens can trigger additional logic, such as executing specific actions whenever the token is deposited. If you don’t need such additional functionality, consider following the tutorial on [Bridging Your Standard ERC-20 Token Using the Standard Bridge](standard-erc20.md#bridging-your-standard-erc-20-token-using-the-standard-bridge) instead.

#### Prerequisite

* [node](https://nodejs.org/en/download/package-manager) (v20.16.0)
* [pnpm](https://pnpm.io/installation)

#### L1 ERC-20 Token Address

For this tutorial, you will need an L1 ERC-20 token. If you already have an L1 ERC-20 token deployed on Sepolia, you can skip this step. Otherwise, you can use a test token available at [`0x5589BB8228C07c4e15558875fAf2B859f678d129`](https://sepolia.etherscan.io/address/0x5589BB8228C07c4e15558875fAf2B859f678d129), which includes a function that allows you to mint tokens.

#### Creating an L2 ERC-20 Token

If you have an L1 ERC-20 token, you can create the corresponding L2 ERC-20 token on Thanos. In this tutorial, we will use Remix, allowing you to easily deploy the token without frameworks like Hardhat. Alternatively, if you prefer, you can follow the same general process within your preferred framework.

In this section, we will create an ERC-20 token that allows deposits but does not allow withdrawals. This is just one of the many ways you can customize an L2 token.

1. [Remix](https://remix.ethereum.org/#lang=en\&optimize=false\&runs=200\&evmVersion=null\&version=soljson-v0.8.26+commit.8a97fa7a.js)

Click the 📄 ("Create New File") button to create a new blank Solidity file. You can name this file whatever you like. Copy the following sample contract into the new file.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
 
import { ERC20 } from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import { IERC165 } from "@openzeppelin/contracts/utils/introspection/IERC165.sol";
import { IOptimismMintableERC20 } from "https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/universal/IOptimismMintableERC20.sol";
 
contract MyCustomL2Token is IOptimismMintableERC20, ERC20 {
    /// @notice Address of the corresponding version of this token on the remote chain.
    address public immutable REMOTE_TOKEN;
 
    /// @notice Address of the StandardBridge on this network.
    address public immutable BRIDGE;
 
    /// @notice Emitted whenever tokens are minted for an account.
    /// @param account Address of the account tokens are being minted for.
    /// @param amount  Amount of tokens minted.
    event Mint(address indexed account, uint256 amount);
 
    /// @notice Emitted whenever tokens are burned from an account.
    /// @param account Address of the account tokens are being burned from.
    /// @param amount  Amount of tokens burned.
    event Burn(address indexed account, uint256 amount);
 
    /// @notice A modifier that only allows the bridge to call.
    modifier onlyBridge() {
        require(msg.sender == BRIDGE, "MyCustomL2Token: only bridge can mint and burn");
        _;
    }
 
    /// @param _bridge      Address of the L2 standard bridge.
    /// @param _remoteToken Address of the corresponding L1 token.
    /// @param _name        ERC20 name.
    /// @param _symbol      ERC20 symbol.
    constructor(
        address _bridge,
        address _remoteToken,
        string memory _name,
        string memory _symbol
    )
        ERC20(_name, _symbol)
    {
        REMOTE_TOKEN = _remoteToken;
        BRIDGE = _bridge;
    }
 
    /// @custom:legacy
    /// @notice Legacy getter for REMOTE_TOKEN.
    function remoteToken() public view returns (address) {
        return REMOTE_TOKEN;
    }
 
    /// @custom:legacy
    /// @notice Legacy getter for BRIDGE.
    function bridge() public view returns (address) {
        return BRIDGE;
    }
 
    /// @notice ERC165 interface check function.
    /// @param _interfaceId Interface ID to check.
    /// @return Whether or not the interface is supported by this contract.
    function supportsInterface(bytes4 _interfaceId) external pure virtual returns (bool) {
        bytes4 iface1 = type(IERC165).interfaceId;
        // Interface corresponding to the updated OptimismMintableERC20 (this contract).
        bytes4 iface2 = type(IOptimismMintableERC20).interfaceId;
        return _interfaceId == iface1 || _interfaceId == iface2;
    }
 
    /// @notice Allows the StandardBridge on this network to mint tokens.
    /// @param _to     Address to mint tokens to.
    /// @param _amount Amount of tokens to mint.
    function mint(
        address _to,
        uint256 _amount
    )
        external
        virtual
        override(IOptimismMintableERC20)
        onlyBridge
    {
        _mint(_to, _amount);
        emit Mint(_to, _amount);
    }
 
    /// @notice Prevents tokens from being withdrawn to L1.
    function burn(
        address,
        uint256
    )
        external
        virtual
        override(IOptimismMintableERC20)
        onlyBridge
    {
        revert("MyCustomL2Token cannot be withdrawn");
    }
}
```

2. Save and Deploy the Contract

Save the file to automatically compile the contract. If you have disabled automatic compilation, click on the "Solidity Compiler" tab and manually compile the contract by pressing the blue "Compile" button.

Open the Deploy tab. Make sure the environment is set to "Injected Provider" your wallet is connected to Thanos, and Remix has access to your wallet. Then, select the contract from the `MyCustomL2Token` deployment dropdown and deploy it with the following parameters.

```bash
_BRIDGE: "0x4200000000000000000000000000000000000010"
_REMOTETOKEN: "<L1 ERC-20 address>"
_NAME: "My Custom L2 Token"
_SYMBOL: "MCL2T"
```
