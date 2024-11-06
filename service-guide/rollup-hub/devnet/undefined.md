---
description: Tokamak Rollup Hub를 통해 자체 롤업을 배포하는데 필요한 정보들과 단계별 가이드가 제공됩니다.
---

# 단계별 배포 가이드

### **하드웨어 요구 사항**

* 최소
  * 멀티코어 이상의 CPU
  * 4GB RAM
  * 메인넷 동기화를 위한 1TB의 여유 저장 공간
  * 초당 8MBit 다운로드 인터넷 서비스
* 권장
  * 4코어 이상의 빠른 CPU
  * 16GB 이상의 RAM
  * 1TB 이상의 여유 공간이 있는 고성능 SSD
  * 초당 25MBit 이상의 다운로드 인터넷 서비스

최소 요구 사항으로도 배포할 수 있지만, 최소한 권장 사항 이상의 하드웨어로 배포하는 것을 추천합니다.

### **프로세스 가이드**

Devnet 배포를 시작하기 전에 종속성(Dependencies)이 설치되어 있는지 확인하세요. 최소 버전 이상이 설치되지 않은 경우 아래에서 단계별 가이드를 제대로 따르셨다고 하더라도 원활한 설치 및 동작이 진행되지 않을 수 있습니다.

*   Software Dependency 버전 정보

    * 가능한 최신 버전을 사용하고 최소 버전 이상을 사용하는 것이 좋습니다.

    | **Dependency**                                                | **최소 버전**         | **권장 버전**             | **버전 확인 커맨드**    |
    | ------------------------------------------------------------- | ----------------- | --------------------- | ---------------- |
    | [git](https://git-scm.com/)                                   | `^2`              | `최소 버전 이상`            | git --version    |
    | [go](https://go.dev/)                                         | `^1.21`           | `^1.22.7 and earlier` | go version       |
    | [node](https://nodejs.org/en/)                                | `^20`             | `최소 버전 이상`            | node --version   |
    | [pnpm](https://pnpm.io/installation)                          | `^8`              | `최소 버전 이상`            | pnpm --version   |
    | [foundry](https://github.com/foundry-rs/foundry#installation) | `0.2.0 (a5efe4f)` | `0.2.0 (63fff35)`     | forge --version  |
    | [make](https://linux.die.net/man/1/make)                      | `^3`              | `최소 버전 이상`            | make --version   |
    | [jq](https://github.com/jqlang/jq)                            | `^1.6`            | `최소 버전 이상`            | jq --version     |
    | [direnv](https://direnv.net/)                                 | `^2`              | `최소 버전 이상`            | direnv --version |

    1.  Devnet을 배포하기 전에 먼저 시스템 실행에 필요한 소프트웨어를 설치하고 버전을 확인해야 합니다. 배포를 진행하기 전에 적절한 버전이 설치되어 있는지 확인하시기 바랍니다.\
        \
        ※ Mac OS / Linux 사용자의 경우 원스텝 설치가 가능하도록 스크립트를 미리 구성해두었습니다. 아래 2단계에 따라 리포지토리를 Git clone한 후 다음 스크립트를 입력하여 설치를 완료하세요.

        <pre class="language-bash"><code class="lang-bash"><strong>cd tokamak-thanos
        </strong>./install-devnet-packages.sh
        </code></pre>
    2.  아래 리포지토리를 로컬로 배포하려는 PC에 clone합니다.

        ```bash
        git clone <https://github.com/tokamak-network/tokamak-thanos.git>
        ```
    3.  Clone한 리포지토리로 이동하여 make build를 입력합니다. 즉시, 롤업 배포를 위한 여러 파일들이 설치됩니다.

        <pre class="language-bash"><code class="lang-bash"><strong>make build
        </strong></code></pre>
    4.  모든 작업이 완료되면,  devnet-up을 사용해 롤업을 배포합니다.

        <pre class="language-bash"><code class="lang-bash"><strong>make devnet-up
        </strong></code></pre>

### **성공적인 배포 후 점검**

배포가 정상적으로 되었는지 확인하는 명령어를 통해 점검해볼 수 있습니다. 다음의 스크립트 입력 이후 응답이 있다면, 롤업이 정상적으로 배포되었음을 의미합니다.

*   L1 체인 점검

    <pre class="language-bash"><code class="lang-bash"><strong>cast chain-id --rpc-url http://localhost:8545
    </strong></code></pre>
*   L2 체인 점검

    ```bash
    cast chain-id --rpc-url http://localhost:9545
    ```

### **공개된 개발용 계정**

Devnet 배포를 간소화하기 위해 사전에 충분한 잔액이 있는 공개용 테스트 계정을 지원합니다.

&#x20;⚠️ 아래의 비공개 키는 외부에 널리 공개되어 있으므로, Devnet 이외의 다른 네트워크에서 사용해서는 **절대 안됩니다**. 메인넷이나 테스트넷에서 이러한 비공개 키를 사용하면 **자금 손실**이 발생할 가능성이 높습니다.

| **주소(Address)**                                            | **비공개 키(Private Key)**                                             | **잔액**     |
| ---------------------------------------------------------- | ------------------------------------------------------------------ | ---------- |
| 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (**Admin**)     | 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 | 10,000 ETH |
| 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (**Proposer**)  | 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d | 10,000 ETH |
| 0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC (**Batcher**)   | 0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a | 10,000 ETH |
| 0x90F79bf6EB2c4f870365E785982E1f101E93b906                 | 0x7c852118294e51e653712a81e05800f419141751be58f605c371e15141b007a6 | 10,000 ETH |
| 0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65                 | 0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a | 10,000 ETH |
| 0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc (**Sequencer**) | 0x8b3a350cf5c34c9194ca85829a2df0ec3153be0318b5e2d3348e872092edffba | 10,000 ETH |
| 0x976EA74026E726554dB657fA54763abd0C3a0aa9                 | 0x92db14e403b83dfe3df233f83dfa3a0d7096f21ca9b0d6d6b8d88b2b4ec1564e | 10,000 ETH |
| 0x14dC79964da2C08b23698B3D3cc7Ca32193d9955                 | 0x4bbbf85ce3377467afe5d46f804f221813b2bb87f24d81f60f1fcdbf7cbf4356 | 10,000 ETH |
| 0x23618e81E3f5cdF7f54C3d65f7FBc0aBf5B21E8f                 | 0xdbda1821b80551c9d65939329250298aa3472ba22feea921c0cf5d620ea67b97 | 10,000 ETH |
| 0xa0Ee7A142d267C1f36714E4a8F75612F20a79720                 | 0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6 | 10,000 ETH |

• Mnemonic: `test test test test test test test test test test test junk`&#x20;

• 파생 경로(Derivation path): `m/44'/60'/0'/0/`
