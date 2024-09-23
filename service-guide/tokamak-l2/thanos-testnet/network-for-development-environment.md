# Network For Development Environment

L2에서 스마트 컨트랙트를 개발하기 위해서는 L2 개발 환경을 만들어야 합니다. L1은 Ganache, hardhat 등을 활용하여 간단히 개발 환경을 만들 수 있는 반면에, L2를 위한 개발 환경은 프로젝트에 따라 더 많은 서비스를 실행해야 합니다. 본 섹션에서는 리눅스 기반 환경에서 Thanos L2 개발 환경을 만드는 방법을 설명합니다.

Thanos 는 간단한 명령어 실행을 통한 L2 네트워크 구축을 지원합니다. 이를 통해 사용자들은 복잡한 설정 작업 없이 쉽게 네트워크를 관리할 수 있습니다. 더불어, Thanos 는 높은 확장성과 유연성을 가지고 있어 개발자들이 비즈니스 요구사항에 따라 네트워크를 조정할 수 있습니다. 이러한 이점들은 개발자들이 필요한 IT 인프라를 구축하는 데 큰 도움을 줄 것입니다.

## P**rerequisite**

### **권장 사양**

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
