# Deployment Guide

### Introduction

In previous steps, we deployed the Thanos chain with the Thanos stack. Now, we are ready to deploy the op bridge, through which the users can seamlessly move assets between the L1 chain and the L2 chain, which was deployed in previous steps.

This bridge supports users to deposit and withdraw ETH, Native token, USDT, and USDC through minimized and lightweight User interface.

In this guide, you will learn how to deploy op bridge by following the structured approach.

The deployment process involves a few phases and is quite simple:

1. **Prerequisite**: Make sure that you get the necessary network configuration info and contract addresses from the previous deployment process and that the latest public bridge Docker image is in the Docker Hub.
2. **Deployment:** This step guides you to deploy the op bridge using the helm chart, simplifying the deployment process.

### Prerequisite

1.  Make sure that you’ve got the necessary network info and contract addresses from the previous network deployment.

    ```jsx
    l1_chain_name: // The name of L1 chain
    l1_chain_id: // The id of L1 chain
    l1_rpc: // The rpc of L1 chain
    l1_native_currency_name: // The name of L1 native currency
    l1_native_currency_symbol: // The symbol of L1 native currency
    l1_native_currency_decimals: // The decimals of L1 native currency
    l1_block_explorer: // The block explorer of L1 chain

    l2_chain_name: // The name of L2 chain
    l2_chain_id: // The id of L2 chain
    l2_rpc: // The rpc of L2 chain
    l2_native_currency_name: // The name of L2 native currency
    l2_native_currency_symbol: // The symbol of L2 native currency
    l2_native_currency_decimals: // The decimals of L2 native currency
    l2_block_explorer: // The block explorer of L2 chain

    native_token_l1_address: // The L1 address of the L2 native currency
    l1_usdc_address: // The address of the L1 USDC
    l1_usdt_address: // The address of the L1 USDT
    l2_usdt_address: // The address of the L2 USDT

    standard_bridge_address: // The address of the l1 standard bridge
    address_manager_address: // The address of the address manager
    l1_cross_domain_messenger_address: // The address of the L1 cross domain messenger
    optimism_portal_address: // The address of the Optimism portal
    l2_output_oracle_address: // The address of the L2 output oracle
    l1_usdc_bridge_address: // The address of the L1 USDC bridge
    dispute_game_factory_address: // The address of the dispute game factory

    ```
2. Make sure that there is a docker image for bridge application [here](https://hub.docker.com/r/tokamaknetwork/trh-op-bridge-app/tags).

### Deployment

1.  Add Helm chart

    ```bash
    $ helm repo add thanos-stack <https://tokamak-network.github.io/tokamak-thanos-stack>

    "thanos-stack" has been added to your repositories
    ```
2.  Search thanos-stack repo

    You can check that the thanos-stack chart includes op-bridge.

    ```bash
    $ helm search repo thanos-stack

    NAME                            CHART VERSION   APP VERSION     DESCRIPTION
    thanos-stack/thanos-stack       1.0.0                           A Helm chart to deploy Thanos stack
    thanos-stack/op-bridge          1.0.0                           A Helm chart to deploy Optimistic bridge
    ```
3.  Make values.yaml file

    You can check the value file needed to deploy op-bridge at the link below.

    * Link: [https://github.com/tokamak-network/tokamak-thanos-stack/blob/main/charts/op-bridge/values.yaml](https://github.com/tokamak-network/tokamak-thanos-stack/blob/main/charts/op-bridge/values.yaml)

    The values.yaml file currently used for [bridge.thanos-sepolia.tokamak.network](https://bridge.thanos-sepolia.tokamak.network/) is as follows.

    ```bash
    # values.yaml

    op_bridge:
      env:
        l1_chain_name: Sepolia
        l1_chain_id: "11155111"
        l1_rpc: <https://sepolia.rpc.tokamak.network>
        l1_native_currency_name: "Sepolia Ether"
        l1_native_currency_symbol: ETH
        l1_native_currency_decimals: 18
        l1_block_explorer: <https://sepolia.etherscan.io>

        l2_chain_name: "Thanos Sepolia"
        l2_chain_id: "111551119090"
        l2_rpc: <https://rpc.thanos-sepolia.tokamak.network>
        l2_native_currency_name: TON
        l2_native_currency_symbol: TON
        l2_native_currency_decimals: 18
        l2_block_explorer: <https://explorer.thanos-sepolia.tokamak.network>

        native_token_l1_address: 0xa30fe40285b8f5c0457dbc3b7c8a280373c40044
        l1_usdc_address: 0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238
        l1_usdt_address: 0x42d3b260c761cD5da022dB56Fe2F89c4A909b04A
        l2_usdt_address: 0xF5559bf8241AC3f8CC067C7BaD7Ec3eec176DB1b

        standard_bridge_address: 0x757EC5b8F81eDdfC31F305F3325Ac6Abf4A63a5D
        address_manager_address: 0xca074de2a95EE6ea37003585E8679f8f215Ea04c
        l1_cross_domain_messenger_address: 0xd054Bc768aAC07Dd0BaA2856a2fFb68F495E4CC2
        optimism_portal_address: 0x2fbD30Fcd1c4573b0288E706Be56B5c0d2DfcAF6
        l2_output_oracle_address: 0xC0885eEc313e31a917DFd5d6Bf33565826B93A3F
        l1_usdc_bridge_address: 0x7dD2196722FBe83197820BF30e1c152e4FBa0a6A
        dispute_game_factory_address: 0x524c885A976c13497900A04257605cd231Ab0026
    ```
4. Deploy op-bridge
   1.  Create namespace

       ```bash
       $ kubectl create ns {NAMESPACE_NAME}
       ```

       e.g.

       ```bash
       $ kubectl create ns bridge
       ```
   2.  Deploy charts with values.yaml

       ```bash
       $ helm install {RELEASE_NAME} thanos-stack/op-bridge \\
       		--values values.yaml \\
       		--namespace {NAMESPACE_NAME}
       ```

       e.g.

       ```bash
       $ helm install mini-bridge thanos-stack/op-bridge \\
       		--values values.yaml \\
       		--namespace bridge
       ```
   3.  Check you Release

       ```bash
       helm -n {NAMESPACE_NAME} list
       ```

       e.g.

       ```bash
       $ helm -n bridge list
       NAME            NAMESPACE       REVISION        UPDATED                              STATUS   CHART           APP VERSION
       mini-bridge     bridge          1               2024-12-20 10:35:57.474309 +0900 KST deployed op-bridge-1.0.0
       ```
   4.  Check your pods

       ```bash
       $ kubectl -n {NAMESPACE_NAME} get pods
       ```
   5.  Check your service

       The bridge service is provided on port 3000.

       ```bash
       $ kubectl -n {NAMESPAC_NAME} get svc

       NAME                         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
       your-bridge-name-op-bridge   ClusterIP   10.100.96.63   <none>        3000/TCP   3d
       ```
   6.  You can enable ingress using the following options in values.yaml file.

       ```bash
        # values.yaml

        op-bridge:
       	 env:
       	 
       			......
       			
       	 ingress:
       	    enabled: true
       	    className: alb
       	    annotations:
       	      alb.ingress.kubernetes.io/target-type: ip
       	      alb.ingress.kubernetes.io/scheme: internet-facing
       	      alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
       	      alb.ingress.kubernetes.io/ssl-redirect: "443"
       	      alb.ingress.kubernetes.io/group.name: thanos-stack
       	    hostname: bridge.thanos-sepolia.tokamak.network
       	    tls:
       	      enabled: true
       ```
5.  Delete op-bridge

    ```bash
    $ helm -n {NAMESPACE_NAME} uninstall {RELEASE_NAME}
    ```

e.g.

```bash
$ helm -n bridge uninstall mini-bridge
```
