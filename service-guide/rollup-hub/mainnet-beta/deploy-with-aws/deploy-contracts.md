# Deploy contracts

To deploy the Thanos chain, which interacts with the L1 chain, system contracts need to be deployed on the L1 chain.

To proceed with this process, you need to clone the following repository:

[GitHub - tokamak-network/tokamak-thanos](https://github.com/tokamak-network/tokamak-thanos)

{% hint style="info" %}
The deployment consumes over 80 million gas; monitoring gas prices is crucial to minimize ETH costs during mainnet deployment.
{% endhint %}

## Create deploy-config file

1. Enter some information on the Tokamak Rollup Hub website and download the `deploy-config` file.
   * Link: [https://rolluphub.tokamak.network/deploy](https://rolluphub.tokamak.network/deploy)

## Configure env

Before starting the deployment, you need to set up the environment variables required by the deployment script.

1.  Go to the scripts directory

    ```bash
    $ cd packages/tokamak/contracts-bedrock/scripts
    ```
2.  Copy the `env.example.deploy` to `env`

    ```bash
    $ cp env.example.deploy env
    ```
3. Please fill in the values for the items listed below in the copied file and save.
   * `GS_ADMIN_PRIVATE_KEY`: Admin private key for deploying contracts
   * `L1_RPC_URL`: L1 RPC URL address

## Build repository

1.  Go to the scripts directory

    ```bash
    $ cd packages/tokamak/contracts-bedrock/scripts
    ```
2.  Build repository

    ```bash
    $ ./start-deploy.sh build
    ```

## Deploy contracts

In this step, we will deploy the system contracts along with the `deploy-config` file.

1.  Run the script with deploy. You need to provide the paths to the `deploy-config` file and the `env` file as options.

    ```bash
    $ ./start-deloy.sh deploy \
    		-c {YOUR_DEPOY_CONFIG_FILE_PATH}/thanos-stack-example.json \
    		-e ./env
    ```
2.  After finish deploy, you can check the deployed contract addresses in the file below.

    ```bash
    $ cd tokamak-thanos/packages/tokamak/contracts-bedrock/deployments

    $ cat {l1-chian-id}-deploy.json
    {
      "AddressManager": "0x48d97a43b0dCb941bEaF7303D2C49d54AB8D25a0",
      "AnchorStateRegistry": "0xA5007d7fD72661Fd9687aBd703A610b53Bf9d98a",
      "AnchorStateRegistryProxy": "0x7D675d172ed343562c6b44db1e3e056EEd119963",
      "DelayedWETH": "0xBE4a9353245025b0Ce4C21a9dC415Bcf50ccB1D0",
      "DelayedWETHProxy": "0x7Ab6723b7A26B1D5D9e712BA15Dc4799Ea34323f",
      "DisputeGameFactory": "0xcc29D6d257f10055D7BF0a0c1CE788d02488D543",
      "DisputeGameFactoryProxy": "0x7Ae553e72EFC4Ba74BdCc20eb5DcEC084c3550eE",
      "L1CrossDomainMessenger": "0x4a281e2322d6B0047c3c62f4ab2F96fE870242A7",
      "L1CrossDomainMessengerProxy": "0x908b14ac8B638d83cF1E96ff93210637D135a9a6",
      "L1ERC721Bridge": "0x65cC27f4337C7c43fb63B70B2d248e1BE6cB2dfd",
      "L1ERC721BridgeProxy": "0x3fe36ecedA07E5DfAc647EA985260688Ed017C16",
      "L1StandardBridge": "0x9f01fde25587c91B1277E50503eDFa00749aC21b",
      "L1StandardBridgeProxy": "0xDE00115D2f21E895d780ED73eD119Abad5e0766b",
      "L1UsdcBridge": "0x2399ECC208D02FbeEC8e9C67145b163e67548398",
      "L1UsdcBridgeProxy": "0xB8baC5Bc7d01c3e50eaF4B9Bdd3687811974540c",
      "L2OutputOracle": "0x68dc3730dC31376284993d3eAB26Ae322391808c",
      "L2OutputOracleProxy": "0xBCc2fC7fEf30AdD2Ab1c0e28349840e75d21C342",
      "Mips": "0x376340A6e24ff4A3a414CD0F02001bEc8e51837b",
      "OptimismMintableERC20Factory": "0xfE8B60d07D3d6AD82489567FdA7F197C7922CcBC",
      "OptimismMintableERC20FactoryProxy": "0x3aCf83f57dC39aB90328A808D87f620c379c5D78",
      "OptimismPortal": "0x689Cfae5Ea657e41dA9E837aa8caA29f81478C92",
      "OptimismPortal2": "0x1722D0D01C8DCB786395E2974EeCD9A26b4fcAC6",
      "OptimismPortalProxy": "0xA54d54be7FfDd33D491A0eB11583A04d4b14CFC3",
      "PermissionedDelayedWETHProxy": "0xD5c535b1c50AE6929B8206B32191308DAa3dFB45",
      "PreimageOracle": "0x5c413d4Aff6881Cf37868AF3B4bB0530919baA78",
      "ProtocolVersions": "0x9abdf2478DD6927B0e24F7F2683807D5843d836e",
      "ProtocolVersionsProxy": "0xa752a5070FEc62e6387B4eEF2A40162b25480DeD",
      "ProxyAdmin": "0x6FF16D985D4BFB9b38DB9E0C3d65f423Da326587",
      "SafeProxyFactory": "0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2",
      "SafeSingleton": "0xd9Db270c1B5E3Bd161E8c8503c55cEABeE709552",
      "SuperchainConfig": "0x5ce577676288F3FbBb373Fc26d5A3f09deeB29E7",
      "SuperchainConfigProxy": "0xf9fE171d1c665544FD6A44F4E3Bb0852a1c07f7b",
      "SystemConfig": "0x2d200a81992A5f19DC2d027F4B430fBb342193d5",
      "SystemConfigProxy": "0x556390099844027766133433eB534cAF634d7934",
      "SystemOwnerSafe": "0xbdBD7c800065A319410fa3a63206eb751dBf696C"
    }
    ```

## Generate genesis and rollup files

To run the Thanos chain, a genesis file and a rollup file are required.

1.  Go to the scripts directory

    ```bash
    $ cd packages/tokamak/contracts-bedrock/scripts
    ```
2.  Run script with generate.

    ```bash
    $ ./start-deloy.sh generate \
    		-c {YOUR_DEPOY_CONFIG_FILE_PATH}/thanos-stack-example.json \
    		-e ./env
    		
    Genesis file: /Users/austin/Documents/tokamak-network/tokamak-thanos/build/genesis.json
    Rollup file: /Users/austin/Documents/tokamak-network/tokamak-thanos/build/rollup.json
    ```
3.  The generated files can be found at the path below.

    ```bash
    $ cd tokamak-thanos/build

    $ ls

    genesis.json  rollup.json
    ```

Once everything is complete, proceed to the next phase
