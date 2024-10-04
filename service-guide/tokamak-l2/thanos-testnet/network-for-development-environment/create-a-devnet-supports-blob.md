# Create a Devnet supports blob

Layer 2 Solutions like Thanos periodically roll-up L2 transactions and state data to Layer 1 (L1) to ensure data availability. [EIP-4844 ](https://eips.ethereum.org/EIPS/eip-4844)introduced a new data type called blob for use in blob-carrying transactions, replacing calldata when rolling up transaction data. This feature was implemented on Ethereum mainnet with the Dencun upgrade on March 13, 2024. Blob offers more efficient data processing and storage compared to calldata, significantly reducing gas costs.&#x20;

This section explains how developers can set up a devnet supporting blobs.

## P**rerequisite**

{% hint style="info" %}
The basic hardware and software requirements are the same as those for the [Network for Development Environment.](./) This page provides an overview of the additional prerequisites.
{% endhint %}

**Client Version**

* Consensus Layer
  * Upgrade your Lighthouse BN(s) to a v5.x.x release
  * Upgrade your Lighthouse VC(s) to a v5.x.x release.
* Execution Layer
  * Upgrade your Execution Engine to a Cancun-ready release (op-geth [v1.101315.0](https://github.com/ethereum-optimism/op-geth/releases/tag/v1.101315.0) or later)

**Deploy Configuration (**[**path**](https://github.com/tokamak-network/tokamak-thanos/blob/fjord\_devnet\_blob/packages/tokamak/contracts-bedrock/deploy-config/devnetL1-template.json)**)**

* `l1UseClique`is set to **false**
* `l1BlockTime`is set to **6**

**Node Configuration (**[**path**](https://github.com/tokamak-network/tokamak-thanos/blob/fjord\_devnet\_blob/ops-bedrock/docker-compose.yml)**)**

* op-node
  * `--l1.beacon`: the HTTP server endpoint of beacon client (**http://localhost:5052**)
  * `--l1.epoch-poll-interval`: Poll interval for retrieving new L1 epoch updates such as safe and finalized block changes. Disabled if 0 or negative (**12s**)
  * `--l1.http-poll-interval`: Polling interval for latest-block subscription when using an HTTP RPC provider. Ignored for other types of RPC endpoints (**6s**)
* op-batcher
  * `--data-availability-type`is set to **blobs**

## Add a Beacon Chain

Beacon Chain is the Consenus Layer of Ethereum 2.0, which is responsible for implementing the Proof of Stake (PoS) consensus mechanism.

* Beacon Node (= Consensus Node): Manages the state of the blockchain, validates new blocks, and maintains consensus on the network.
* Validator Client: Acts as a validator in Ethereum 2.0, increasing the security of the network by proposing and validating blocks.

You can see services and components in this network to [`ops-bedrock/docker-compose.yml`](https://github.com/tokamak-network/tokamak-thanos/blob/feat/devnet-blob/ops-bedrock/docker-compose.yml). (additional services: `l1-bn`, `l1-vc)`

We use [https://github.com/protolambda/eth2-testnet-genesis](https://github.com/protolambda/eth2-testnet-genesis) for setup beacon genesis state in the devnet.

1. Deploy a deposit contract.
2. Select an Eth1 block hash and corresponding timestamp as the starting point. The deposit contract contents must be empty at this block.
3. Set the minimum genesis time. This usually determines the first eligible Eth1 block starting point.
4. Ensure the selected Eth1 block is compatible (i.e., has an equal or later timestamp) with the minimum genesis time. If a future compared to the min. genesis time is chosen, the generated state will be invalid.
5. Adjust the actual genesis time by configuring the genesis delay (`actual genesis time = chosen Eth1 time + delay`).

## Run

```
git clone https://github.com/tokamak-network/tokamak-thanos.git
cd tokamak-thanos

git checkout fjord_devnet_blob

pnpm install:foundry

make devnet-up
```

The beacon API allows you to retrieve beacon chain information, blob data, blocks, and more.

* beacon API: [https://ethereum.github.io/beacon-APIs/](https://ethereum.github.io/beacon-APIs/)
* beacon endpoint: http://localhost:5052

```bash
# get latest blob data
curl -X 'GET' \
  'http://localhost:5052/eth/v1/beacon/blob_sidecars/head' \
  -H 'accept: application/json'

# get information of beacon genesis
curl -X GET \
    "http://localhost:5052/eth/v1/beacon/genesis" \
    -H "accept: application/json" 

# get head block
curl -X GET \
    "http://localhost:5052/eth/v1/beacon/blocks/head/root" \
    -H "accept: application/json"
```

## Stop

```
# down the network
make devnet-down

# down the network and remove generated files
make devnet-clean
```
