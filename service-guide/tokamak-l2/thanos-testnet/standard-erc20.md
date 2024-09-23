# Standard ERC20

## Bridging Your Standard ERC-20 Token Using the Standard Bridge

Standard Bridge 시스템을 사용하여 Ethereum 에서 Thanos 으로 ERC-20 토큰을 브리지하는 방법을 알아봅니다. 이 튜토리얼은 Ethereum 에 기존 ERC-20 토큰이 있고 Thanos 에서 브릿지를 지원하는 토큰을 만들고자 하는 개발자를 대상으로 합니다.

이 튜토리얼에서는 [`OptimismMintableERC20Factory`](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/universal/OptimismMintableERC20Factory.sol) 을 사용하는 방법을 설명합니다. Thanos 에 표준화된 ERC-20 토큰을 배포합니다. 이 팩토리 계약으로 생성된 토큰은 Standard Bridge 시스템과 호환되며 입금, 이체 및 인출에 대한 기본 로직을 포함합니다. L2 토큰에 특수 로직을 포함하려면 대신 [Bridging Your Custom ERC-20 Token Using the Standard Bridge](standard-erc20.md#bridging-your-custom-erc-20-token-using-the-standard-bridge) 에 대한 튜토리얼을 참조하세요 .

#### Prerequisite

* [foundry](https://book.getfoundry.sh/getting-started/installation)

#### L1 ERC-20 토큰 주소 <a href="#get-an-l1-erc-20-token-address" id="get-an-l1-erc-20-token-address"></a>

이 튜토리얼을 위해서는 L1 ERC-20 토큰이 필요합니다. 이미 Sepolia에 배포된 L1 ERC-20 토큰이 있는 경우 이 단계를 건너뛸 수 있습니다. 그렇지 않은 경우 [`0x5589BB8228C07c4e15558875fAf2B859f678d129`](https://sepolia.etherscan.io/address/0x5589BB8228C07c4e15558875fAf2B859f678d129)에 정의된 테스트 토큰을 사용할 수 있습니다.  토큰을 주조하는 데 사용할 수 있는 기능이 포함되어 있습니다.

#### L2 ERC-20 토큰 생성 <a href="#create-an-l2-erc-20-token" id="create-an-l2-erc-20-token"></a>

L1 ERC-20 토큰이 있으면 [`OptimismMintableERC20Factory`](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/universal/OptimismMintableERC20Factory.sol)을 사용하여 Thanos에서 ERC-20 토큰을 생성할 수 있습니다. 팩토리 계약에서 생성된 모든 토큰은 `IOptimismMintableERC20`인터페이스를 구현하며  Standard Bridge 시스템과 호환됩니다.

1. 환경 변수에 Private key 추가

```sh
export PRIVATE_KEY=0x...
```

2. 환경 변수에 RPC 추가

```sh
export TUTORIAL_RPC_URL=https://rpc.thanos-sepolia.tokamak.network
```

3. 환경 변수에 L1 ERC20 주소 추가

```sh
# Replace this with your L1 ERC-20 token if not using the testing token!
export TUTORIAL_L1_ERC20_ADDRESS=0x5589BB8228C07c4e15558875fAf2B859f678d129
```

4. L2 ERC-20 토큰 배포

```sh
cast send 0x4200000000000000000000000000000000000012 "createOptimismMintableERC20(address,string,string)" $TUTORIAL_L1_ERC20_ADDRESS "My Standard Demo Token" "L2TKN" --private-key $PRIVATE_KEY --rpc-url $TUTORIAL_RPC_URL --json | jq -r '.logs[0].topics[2]' | cast parse-bytes32-address
```

## Bridging Your Custom ERC-20 Token Using the Standard Bridge

Standard Bridge 시스템을 사용하여 Ethereum 에서 Thanos 으로 ERC-20 토큰을 브리지하는 방법을 알아봅니다. 이 튜토리얼은 Ethereum 에 기존 ERC-20 토큰이 있고 Thanos 에서 브릿지를 지원하는 토큰을 만들고자 하는 개발자를 대상으로 합니다.

이 튜토리얼에서는 사용자 정의 토큰을 생성하는 방법을 설명합니다. 표준 브리지 시스템과 함께 사용할 수 있도록 [`IOptimismMintableERC20`](https://github.com/tokamak-network/tokamak-thanos/blob/main/packages/tokamak/contracts-bedrock/src/universal/IOptimismMintableERC20.sol) 인터페이스를 제공합니다. 사용자 지정 토큰을 사용하면 토큰이 입금될 때마다 추가 로직을 트리거하는 것과 같은 작업을 수행할 수 있습니다. 이와 같은 추가 기능이 필요하지 않으면 대신 [Bridging Your Standard ERC-20 Token Using the Standard Bridge](standard-erc20.md#bridging-your-standard-erc-20-token-using-the-standard-bridge) 에 대한 튜토리얼을 따르는 것을 고려하세요.

#### Prerequisite

* [node](https://nodejs.org/en/download/package-manager) (v20.16.0)
* [pnpm](https://pnpm.io/installation)

#### L1 ERC-20 토큰 주소 <a href="#get-an-l1-erc-20-token-address" id="get-an-l1-erc-20-token-address"></a>

이 튜토리얼을 위해서는 L1 ERC-20 토큰이 필요합니다. 이미 Sepolia에 배포된 L1 ERC-20 토큰이 있는 경우 이 단계를 건너뛸 수 있습니다. 그렇지 않은 경우 [`0x5589BB8228C07c4e15558875fAf2B859f678d129`](https://sepolia.etherscan.io/address/0x5589BB8228C07c4e15558875fAf2B859f678d129)에 정의된 테스트 토큰을 사용할 수 있습니다.  토큰을 주조하는 데 사용할 수 있는 기능이 포함되어 있습니다.

#### L2 ERC-20 토큰 생성

L1 ERC-20 토큰이 있으면 Thanos 에서 해당 L2 ERC-20 토큰을 만들 수 있습니다. 이 튜토리얼에서는 Remix를 사용하여 Hardhat과 같은 프레임워크 없이도 쉽게 토큰을 배포하는 방법을 설명합니다. 또는, 선호하는 프레임워크를 사용할 경우 동일한 일반적인 과정을 따를 수 있습니다.

이 섹션에서는 입금은 가능하지만 인출은 불가능한 ERC-20 토큰을 만들 것입니다. 이것은 L2 토큰을 사용자 정의할 수 있는 무한한 방법 중 하나에 불과합니다.

1. [Remix](https://remix.ethereum.org/)

📄("새 파일 만들기") 버튼을 클릭하여 새 빈 Solidity 파일을 만듭니다. 이 파일의 이름은 원하는 대로 지정할 수 있습니다. 다음 예시 계약서를 새 파일에 복사하세요.

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

2. 계약 저장 및 배포

파일을 저장하여 자동으로 계약을 컴파일합니다. 자동 컴파일을 비활성화한 경우 "Solidity Compiler" 탭을 클릭하고 파란색 "Compile" 버튼을 눌러 수동으로 계약을 컴파일해야 합니다.

배포 탭을 엽니다. 환경이 "Injected Provider"로 설정되어 있고, 지갑이 Thanos 에 연결되어 있으며, Remix가 지갑에 액세스할 수 있는지 확인합니다. 그런 다음 `MyCustomL2Token`배포 드롭다운에서 계약을 선택하고 다음 매개변수로 배포합니다.

```sh
_BRIDGE: "0x4200000000000000000000000000000000000010"
_REMOTETOKEN: "<L1 ERC-20 address>"
_NAME: "My Custom L2 Token"
_SYMBOL: "MCL2T"
```

