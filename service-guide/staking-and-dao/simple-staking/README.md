---
description: 토카막 네트워크가 제공하는 스테이킹 시스템을 소개합니다.
---

# Simple staking

{% hint style="info" %}
서비스 링크

* Website: [https://simple.staking.tokamak.network](https://simple.staking.tokamak.network/)
* Github (contract): [https://github.com/tokamak-network/ton-staking-v2](https://github.com/tokamak-network/ton-staking-v2)
* Contract addresses: [link ](broken-reference)
{% endhint %}

### 1. 특징

토카막 네트워크의 스테이킹서비스는 스테이킹 과정을 통해 DAO 위원회(committee) 맴버 3인이 선정되는 것이 특징입니다. 구체적인 특징은 다음과 같습니다.&#x20;

* 첫째, DAO 후보자에게 TON 또는 WTON을 스테이킹을 해서 일정한 스테이킹 보상을 거둘 수 있습니다.&#x20;
* 둘째, 스테이킹 물량의 순서대로 상위 DAO 후보자 3명은 DAO committee에서 각종 의사결정을 담당하는 맴버로 활동할 수 있습니다.

### 2. 페이지 구성

<figure><img src="../../../.gitbook/assets/image (322).png" alt="" width="375"><figcaption><p>스테이킹 페이지의 초도화면</p></figcaption></figure>

1. **Home**

* Connect Wallet 버튼을 클릭하여 메타마스크(Metamask) 지갑 또는 트레저(Trezor) 지갑을 연결합니다.
* 파란색 그래프는 스테이커가 토카막 네트워크에 매일 스테이킹한 금액을, 회색 그래프는 매일 실제 APY를 보여줍니다. 그래프 위에 마우스를 가져가면 일일 총 스테이킹 금액과 실제 APY를 확인할 수 있습니다.

2. **Staking**

* 스테이킹 페이지에서 DAO 후보에 대한 정보를 확인할 수 있습니다. 지갑을 연결한 경우 각 운영자의 오른쪽에 있는 파란색 화살표를 클릭하면 DAO 후보에 대한 자세한 정보와 스테이킹 버튼을 볼 수 있습니다. 지갑을 연결하지 않은 상태에서는 운영자의 세부 정보만 볼 수 있습니다.
* 지갑이 연결되면 Staking 버튼을 클릭하여 귀하가 보유한 TON(혹은 WTON)을 특정 DAO 후보자에게 스테이킹 할 수 있습니다.&#x20;

3. **Account**

* Account 페이지는 귀하가 보유한 자산에 대한 정보를 제공합니다.
* 본 서비스를 이용하려면 먼저 우상단의 지갑연결을 통해 로그인을 해야 합니다.&#x20;
* 제공되는 정보는 아래와 같습니다
  * Total Staked: 스테이킹된 TON의 합계입니다.&#x20;
  * Pending Withdrawal: Unstake 된 TON의 합계액입니다. 이 금액은 DAO 후보가 설정한 출금 지연 기간(기본값은 93,046블록 경과후, 약 14일)이 지난 후에만 출금할 수 있습니다.&#x20;
  * History: 스테이킹 관련 거래 내역을 보여줍니다. 이 정보에는 트랜잭션 해시, 관련 DAO후보, 유형, 금액, 실행시각이 포함됩니다.&#x20;

### **3. 로그인**

스테이킹 및 계정 정보와 같은 일부 기능에 액세스하려면 지갑연결을 통해 로그인을 먼저 해야 합니다. 화면 오른쪽 상단의 지갑 연결 버튼을 클릭해 지갑 연결 방법을 선택할 수 있습니다.

1. Metamask 연결로 로그인하기

* 이더리움 메인넷 네트워크에 연결되어 있는지 확인합니다. 그런 다음 Connect Wallet 버튼을 클릭합니다.&#x20;
* 팝업창에서 Metamask 아이콘을 클릭합니다.&#x20;
* 브라우저 확장 프로그램 목록에서 Metamask 아이콘을 클릭하고 서비스에 연결할 계정을 선택합니다.
* 메타마스크 설치방법은 아래를 참고하여 주십시오

<details>

<summary>메타마스크 설치 (Install Metamask) 방법</summary>

1. 메타마스크로 당사 서비스에 연결하려면 [Google Chrome](https://chrome.google.com/webstore/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn) 또는 [Firefox](https://addons.mozilla.org/en-US/firefox/addon/ether-metamask/)브라우저에 메타마스크 확장 프로그램을 설치하세요. 이미 메타마스크 지갑에 TON이 있는 경우 해당 지갑에 로그인하거나 이미 TON이 있는 계정(account)을 가져오세요.

<img src="../../../.gitbook/assets/image (1).png" alt="Metamask in Google Chrome Extenstion" data-size="original">

2. 지갑에 TON을 추가하려면 이더리움 메인넷 네트워크에 연결되어 있는지 확인하세요. 그런 다음 토큰 추가 버튼을 클릭합니다.
3. Custom Token 탭을 클릭합니다.&#x20;
4. 아래의 컨트랙트 주소를 입력합니다. 토큰 심볼과 소수 자릿수는 자동으로 채워집니다.&#x20;
   * Token Contract Address: 0x2be5e8c109e2197D077D13A82dAead6a9b3433C5
5. Next 버튼을 클릭합니다.&#x20;
6. 토큰 추가 버튼을 눌러 계정에 TON을 추가합니다.

<img src="../../../.gitbook/assets/image (4).png" alt="How to add TON in your wallet" data-size="original">

</details>

2. Trezor 연결로 로그인하기

* Trezor 지갑을 연결할 수 있습니다.&#x20;

<figure><img src="../../../.gitbook/assets/image (324).png" alt="" width="188"><figcaption><p>암호화폐지갑 연결을 통한 로그인</p></figcaption></figure>
