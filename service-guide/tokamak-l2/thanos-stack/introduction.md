# Introduction

This guide provides a comprehensive overview of the Thanos Stack, which supports a layer 2 chain development environment based on optimal rollups, with information for users and developers. The Tokamak Network provides easy deployment of Layer 2 on top of the Thanos Stack through the [Tokamak Rollup Hub](https://rolluphub.tokamak.network/). For those looking to deploy Layer 2 chains with the Thanos Stack, this guide is an essential starting point for understanding the protocol.

The Thanos Stack is built on top of [OP Stack v1.7.7.](https://github.com/ethereum-optimism/optimism/tree/v1.7.7) To fit the needs of the ecosystem and the direction of the market, we can utilize ERC20 tokens as native tokens at layer 2. We also support permission less fault proof. By allowing anyone to validate and challenge incorrect state transitions in the network, it provides a key mechanism to enhance decentralization and security. The Tokamak Network aims to create a flexible ecosystem that supports on-demand Layer 2 solutions that fit the needs of users. This flexibility allows dApp developers and operators to choose and manage the Layer 2 chain that best suits the specific needs of their application. The Thanos Stack is the first Layer 2 stack that will serve as the cornerstone for building on-demand Layer 2 chains.

The Thanos Stack integrates USDC Bridge and Uniswap V3 as a predeploys. This simplifies the onboarding process by enabling USDC out of the box without the need to deploy a separate bridge contract or additional setup. It also provides developers with a solid foundation on which to build DeFi applications, driving adoption and growth within the ecosystem. In the Thanos Stack, we used contracts under the Uniswap v3 predeploys.

* v3-core: [https://github.com/Uniswap/v3-core](https://github.com/Uniswap/v3-core)
* v3-periphery: [https://github.com/Uniswap/v3-periphery](https://github.com/Uniswap/v3-periphery)
* universal-router: [https://github.com/Uniswap/universal-router](https://github.com/Uniswap/universal-router)
* permit2: [https://github.com/Uniswap/permit2](https://github.com/Uniswap/permit2)

#### Architecture Overview

Thanos Stack consists of several components that work together to deliver L2 scalability, security, and reliability while interacting seamlessly with L1 blockchain systems. Below is overview of the workflow:

1. Transactions are submitted to the L2 mempool, where they are processed by the execution layer (**op-geth**).
2. L2 blocks are proposed by the consensus layer (**op-node**) and validated against L1 data.
3. The **op-batcher** compresses and submits L2 block data to L1 for data availability.
4. Disputes over state transitions are resolved by the **op-challenger**, ensuring the validity of the chain.
5. The **op-proposer** submits periodic state updates (output roots) to L1, enabling communication.

<figure><img src="../../../.gitbook/assets/system_architecture2 (1).png" alt=""><figcaption></figcaption></figure>

The Thanos Stack offers significant benefits, including faster transaction processing and lower transaction fees, and for dApp developers, it provides a seamless development environment that mirrors the familiar Ethereum environment. (To simplify terminology, we’ll refer to the Layer 2 chain built on the Thanos Stack as “Thanos.”)

#### **This guide covers the following sections:**

* System Architecture: An overview of the core components of the Thanos stack and their roles.
* Differences from Ethereum: Key differences from Ethereum, such as gas fee structure, address aliasing, EIP-1559 parameters, and more.
* Transaction fee calculation: A detailed methodology for calculating transaction fees within the Thanos stack.
* L2 Native Token Bridge: Technical specifications for a bridge designed to allow ERC-20 tokens to operate as native tokens at Layer 2.
