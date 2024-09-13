---
description: 스테이킹된 TON을 인출(withdraw) 하는 과정을 소개합니다.
---

# Withdraw

중요한 점은 첫째, 스테이킹 물량을 인출하기 위해서는 스테이킹 해제(unstake)와 인출(withdraw), 두 단계를 거쳐야 한다는 것이며, 둘째, 인출(withdraw) 은 스테이크 해제(unstake) 이후 93,046 블록이(\~14일) 지나야 가능하다는 점입니다.&#x20;

{% hint style="info" %}
출금지연기간

* 기간 : Unstake 완료 후 93,046 블록이 경과되는 시간(약 14일에 해당)



스테이킹 해제(unstake) 이후 약 2주간의 출금지연기간이 경과했음에도 본인의 지갑에 토큰이 입금되지 않았다는 이슈를 제기하는 경우가 있습니다. 이는 출금지연기간 경과 후 인출(withdraw) 버튼을 실행하지 않았기 때문에 발생하는 현상입니다.&#x20;
{% endhint %}

### &#x20;1. 스테이크 해제(Unstake)

* Unstake 버튼을 누르면 팝업이 나타납니다.&#x20;

<figure><img src="../../../.gitbook/assets/image (352).png" alt="" width="375"><figcaption><p>스테이킹 해제(Unstake)</p></figcaption></figure>

* 팝업창이 나타나면 스테이킹을 해제할 TON 수량을 입력합니다.
* Unstake 버튼을 클릭합니다.&#x20;

<figure><img src="../../../.gitbook/assets/image (354).png" alt="" width="148"><figcaption><p>스테이크 해제(unstake) 팝업창</p></figcaption></figure>

* 브라우저의 Metamask 확장 프로그램에서 열리는 Metamask 팝업의 확인 버튼을 클릭합니다.&#x20;

{% hint style="warning" %}
&#x20;스테이크 해제에 앞서 **받지 못한 스테이킹 보상**(Unclaimed Staking Reward)이 있는지 확인하세요&#x20;

* 스테이크 해제를 완료한 이후에는 **받지 못한 스테이킹 보상**이 버닝 됩니다.&#x20;
* **받지 못한 보상이** 있다면, Your Staked로 먼저 옮겨놓으세요 [link](staking-reward.md)
{% endhint %}

### &#x20;2. 인출(Withdraw)

* 스테이크 해제후 93,046 블록(\~14일) 이후에 인출이 가능합니다. Withdraw 버튼을 누르면 팝업창이 나타납니다.&#x20;

<figure><img src="../../../.gitbook/assets/image (355).png" alt="" width="375"><figcaption><p>인출(Withdraw) 버튼</p></figcaption></figure>

* Withdrawable Balance에 인출가능한 물량이 표시됩니다. 스테이크 해제(unstake) 후  93,046 블록간의 출금지연기간이 경과한 물량이 해당됩니다.
* Withdraw 버튼을 클릭하면 인출가능한 물량 전액이 출금됩니다 (일부만 출금할 수 없습니다)

<figure><img src="../../../.gitbook/assets/image (357).png" alt="" width="167"><figcaption></figcaption></figure>

* Metamask 팝업의 컨펌 버튼을 클릭하여 거래를 완료합니다.&#x20;

### **3. 스테이킹 재설정 (**Restake)

스테이킹 해제(unstake)를 실시한 이후, 인출(withdraw)을 하지 않았다면 언제든지 스테이킹을 다시 할 수 있습니다. 이러한 인출대기 물량은 Restake 버튼을 누르면 나타나는 팝업창에서 Restakable Amount라는 이름으로 나타나게 됩니다.&#x20;

<figure><img src="../../../.gitbook/assets/image (347).png" alt="" width="375"><figcaption><p>Restake 버튼(좌) 실행시 나타나는 팝업창(우)</p></figcaption></figure>

* Restake 버튼을 누르면 팝업이 나타납니다.&#x20;
* Restakable Amount 표시 아래에 다시 스테이킹 할 수 있는 물량이 나타납니다. 이 물량 전체에 대해서 일괄적으로 Restake를 적용할 수 있습니다(일부 물량에 대해서만 적용은 안됨)
* Restake 버튼을 누르면, 브라우저의 Metamask 확장 프로그램에서 열리는 Metamask 팝업창이 열립니다.  컨펌 버튼을 눌러서 완료합니다.&#x20;

