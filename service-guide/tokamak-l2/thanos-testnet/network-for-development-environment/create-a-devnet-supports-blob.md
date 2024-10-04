# Create a Devnet supports blob

Thanos와 같은 L2들은 L1에 L2의 트랜잭션과 상태 데이터를 주기적으로 롤업하여 데이터 가용성을 확보합니다. [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) 에서는 트랜잭션 데이터를 롤업할 때 calldata 대신 새로운 데이터 타입인 blob을 사용하는 blob-carrying transaction을 제안했습니다. 이 기능은 2024년 3월 13일, Dencun 업그레이드로 이더리움 메인넷에 적용되었습니다. Blob은 calldata 대비 더 효율적인 데이터 처리 및 저장 방식을 지원하며 가스 비용을 크게 줄일 수 있는 장점이 있습니다.

본 섹션에서는 개발자가 Blob을 지원하는 데브넷을 구축하는 방법을 설명합니다.

## P**rerequisite**

{% hint style="info" %}
기본적인 하드웨어 및 소프트웨어 요구사항은 [Network For Development Environment](./) 와 동일합니다. 이 페이지에서는 추가적인 Prerequisite를 설명합니다.
{% endhint %}

**Client Version**

* Consensus Layer
  * Lighthouse BN(s)을 v5.x.x 버전으로 업그레이드.
  * Lighthouse VC(s)을 v5.x.x 버전으로 업그레이드.
* Execution Layer
  * Execution Engine을 Cancun-ready 버전으로 업그레이드 (op-geth [v1.101315.0](https://github.com/ethereum-optimism/op-geth/releases/tag/v1.101315.0) 이상 사용)

**Deploy Configuration (**[**path**](https://github.com/tokamak-network/tokamak-thanos/blob/fjord\_devnet\_blob/packages/tokamak/contracts-bedrock/deploy-config/devnetL1-template.json)**)**

* `l1UseClique`를 **false**로 설정
* `l1BlockTime`을 **6**으로 설정

**Node Configuration (**[**path**](https://github.com/tokamak-network/tokamak-thanos/blob/fjord\_devnet\_blob/ops-bedrock/docker-compose.yml)**)**

* op-node
  * `--l1.beacon`: 비콘 클라이언트의 HTTP 서버 엔드포인트 (**http://localhost:5052**)
  * `--l1.epoch-poll-interval`: 새로운 L1 에포크 업데이트를 가져오는 폴링 간격 (**12s**)
  * `--l1.http-poll-interval`: HTTP RPC provider 사용 시 최신 블록의 폴링 간격 (**6s**)
* op-batcher
  * `--data-availability-type`을 **blobs**로 설정

## Add a Beacon Chain

비콘 체인(Beacon Chain)은 이더리움 2.0의 Consenus Layer로 Proof of Stake(PoS) 합의 메커니즘을 구현하는 역할을 합니다.

* **비콘 노드(= 합의 노드)**: 블록체인의 상태를 관리하고, 새로운 블록을 검증하며 네트워크에서 합의를 유지합니다.
* **검증자 클라이언트**: 이더리움 2.0에서 검증자로서 활동하며, 블록을 제안하고 검증하여 네트워크의 보안을 강화합니다.

[`ops-bedrock/docker-compose.yml`](https://github.com/tokamak-network/tokamak-thanos/blob/feat/devnet-blob/ops-bedrock/docker-compose.yml)에서 추가된 비콘 체인 서비스와 구성 요소를 확인할 수 있습니다. (추가된 서비스: `l1-bn`, `l1-vc)`

데브넷 구동 시 비콘 제네시스 상태를 설정하기 위해 [https://github.com/protolambda/eth2-testnet-genesis](https://github.com/protolambda/eth2-testnet-genesis) 를 사용합니다.

1. Deposit 계약을 배포합니다.
2. Eth1 블록 해시와 해당 타임스탬프를 시작점으로 선택합니다. 이 시점에 Deposit 계약 내용은 비어 있어야 합니다.
3. 최소 제네시스 시간을 설정합니다.
4. 선택한 Eth1 블록이 최소 제네시스 시간과 호환되는지 확인합니다.
5. 실제 제네시스 시간을 제네시스 지연 시간을 통해 조정합니다 (`실제 제네시스 시간 = 선택한 Eth1 시간 + 지연`).

## Run

```
git clone https://github.com/tokamak-network/tokamak-thanos.git
cd tokamak-thanos

git checkout fjord_devnet_blob

pnpm install:foundry

make devnet-up
```

beacon API 을 사용하면 beacon chain 정보, blob data, block 등을 조회할 수 있습니다.

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

