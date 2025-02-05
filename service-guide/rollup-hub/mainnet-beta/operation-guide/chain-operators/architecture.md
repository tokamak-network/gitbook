# Architecture

### Intro

This page provides information on how the various components of Thanos Stack interact to build an on-demand L2 blockchain system. It conceptually explains each component and introduces the important roles they play within the system, aiming to provide a deeper understanding of how the entire system operates efficiently.

### Overview

The Thanos Stack comprises the following fundamental modules: op-node, op-proposer, op-geth, op-batcher, op-challenger, and the optional proxyd.

In addition, optional components like graph-node and IPFS are primarily used for supporting L2 DApps. These services, however, are not directly related to the core architecture of the Thanos Stack.

The diagram below illustrates how the core components: op-node, op-proposer, op-geth, op-batcher, and op-challenger interact:

<figure><img src="../../../../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

### Components

#### op-node

The [`op-node`](https://github.com/tokamak-network/tokamak-thanos/tree/main/op-node) functions as the consensus layer for L2, handling the building, relaying, and verification of blocks using data from L1. It determines how transactions are added to blocks, which blocks are included in the L2 chain, and which blocks are finalized through the Engine API, similar to L1.

The key responsibilities of op-node include

* Listening for deposit events on the L2.
* Building unsafe blocks that include deposit events and transactions from the L2 mempool.
* Fetching batch data posted by op-batcher to identify which blocks are safe.
* Finalizing L2 blocks based on finalized blocks from L1.

#### op-proposer

The [`op-proposer`](https://github.com/tokamak-network/tokamak-thanos/tree/main/op-proposer) is a lightweight service responsible for automating the submission of output-root proposals to L1 at regular intervals, forming a key part of the Fault Dispute Games. It handles chains with pre-Fault-Proof deployments by submitting proposals to the **OptimismPortal**, and for Fault-Proof-enabled chains, it interacts with the **DisputeGameFactory** to instantiate and resolve fault-proof games. Once an output root is validated, it enables users to finalize withdrawal messages associated with that root, authenticating withdrawals by registering proof of inclusion in the L2 withdrawal contract storage. This process facilitates L2-to-L1 messaging and provides L1 with an accurate view of the L2 state.

#### op-geth

The [`op-geth`](https://github.com/tokamak-network/tokamak-thanos/blob/main/op-e2e/op_geth.go) acts as the execution layer for the L2, where transactions are executed, and blocks are constructed. It receives block decisions from the op-node to determine which blocks should be added or finalized. Since op-geth is a stateful service, operators must exercise caution when handling unexpected shutdowns to avoid issues with state integrity.

#### op-batcher

The [`op-batcher`](https://github.com/tokamak-network/tokamak-thanos/tree/main/op-batcher) is a service responsible for submitting L2 block data to L1 to ensure data availability for verifiers while minimizing costs. It bundles and compresses L2 transactions before posting them to L1, using blob transactions in the Thanos stack for gas efficiency. Working in conjunction with the sequencer, the data availability layer, and the pipeline, the op-batcher supports the progression of the safe chain by optimizing data submission and maintaining availability.

#### op-challenger

The [`op-challenger`](https://github.com/tokamak-network/tokamak-thanos/tree/main/op-challenger) is an agent in the fault dispute system that defends the chain by verifying output root proposals, challenging invalid ones, and resolving claims in dispute games. It relies on a synced rollup node and trace provider for claim verification and works with the op-proposer to resolve claims after the time limit expires. The op-challenger ensures the correct state of the chain by defending valid proposals and assisting in the resolution of fault disputes.

Its key responsibilities include

* Monitoring and interacting with dispute games.
* Defending valid output root proposals.
* Challenging invalid output root proposals.
* Resolving Fault Dispute Games.
* Claiming payout bonds for both challengers and proposers.

#### L1 Proxyd

[`proxyd`](https://github.com/tokamak-network/tokamak-thanos/tree/main/proxyd) functions as a proxy server to route requests to **op-geth**. It facilitates efficient management of RPC services between L2 components, ensuring performance, fault tolerance, and security.

The diagram below illustrates how **proxyd** interacts with **op-geth** and \*\*\*\*end users.

<figure><img src="../../../../../.gitbook/assets/Screenshot from 2024-12-19 22-04-37.png" alt=""><figcaption></figcaption></figure>

#### Graph Node

`Graph node` enables efficient indexing and querying of blockchain data within the Thanos Stack, allowing fast access to event and transaction data. Through the integration of Graphnode, Thanos Stack can interact with blockchain information and query it efficiently, making the system more flexible and scalable.

#### IPFS

**`IPFS`**, when combined with the Thanos Stack, provides decentralized and efficient file storage. This integration ensures that large datasets, such as off-chain data, are securely stored and accessed in a distributed manner, complementing the blockchain's capabilities within the Thanos Stack.
