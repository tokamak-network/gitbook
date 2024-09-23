# Network for development environment

To develop smart contracts on Titan L2, it is necessary to create an L2 development environment. While setting the development environment for L1 (Ethereum) is relatively simple, using tools such as Ganache or hardhat-node, L2 development environment requires additional resources. This section explains how to create a Thanos L2 development environment in a Linux-based environment.

Thanos supports L2 network deployment through simple command execution. This allows users to easily manage the network without complicated setup tasks. In addition, Thanos is highly scalable and flexible, allowing developers to adjust the network according to business requirements. These benefits will go a long way in helping developers build the IT infrastructure they need.

## P**rerequisite**

### **Recommended Spec**

* CPU - 16 Core
* Memory - 32GB

#### Git

* [https://git-scm.com/download/linux](https://git-scm.com/download/linux)

#### Make

* Debian/Ubuntu – `apt install make`
* Fedora/RHEL – `yum install make`
* Arch/Manjaro – `pacman -S make`

#### Build-essential

* Debian/Ubuntu – `apt install build-essential`
* Fedora/RHEL – `yum groupinstall ‘Development Tools’`
* Arch/Manjaro – `pacman -S base-devel`

#### Go (v1.22.6)

* [https://go.dev/doc/install](https://go.dev/doc/install)
* [https://go.dev/dl/go1.22.6.linux-amd64.tar.gz](https://go.dev/dl/go1.22.6.linux-amd64.tar.gz)

#### Node.js (v20.16.0)

* [https://nodejs.org/en/download/package-manager](https://nodejs.org/en/download/package-manager)

#### pnpm

* [https://pnpm.io/installation](https://pnpm.io/installation)

#### Cargo (v1.78.0)

* [https://doc.rust-lang.org/cargo/getting-started/installation.html](https://doc.rust-lang.org/cargo/getting-started/installation.html)

```
rustup install 1.78.0
rustup default 1.78.0
```

#### Docker engine

* [https://docs.docker.com/engine/install](https://docs.docker.com/engine/install/)
* [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)

## Run

```
git clone https://github.com/tokamak-network/tokamak-thanos.git
cd tokamak-thanos

pnpm install:foundry

make devnet-up
```

## Stop

```
# down the network
make devnet-down

# down the network and remove generated files
make devnet-clean
```
