---
description: 타이탄에서 이더리움으로 자산을 출금(withdraw)하는 방법
---

# Withdraw

1. Withdraw을 수행하려면 origin 네트워크(왼쪽)는 타이탄, destination 네트워크(오른쪽)는 이더리움이어야 합니다.

<figure><img src="../../../.gitbook/assets/image (180).png" alt=""><figcaption><p>Withdraw하려면 origin 네트워크(왼쪽)가 타이탄으로 설정되어 있고 destination 네트워크(오른쪽)가 이더리움으로 설정되어 있는지 확인합니다.</p></figcaption></figure>



2. 카드를 클릭하면 카드 리스트가 나타나고 withdraw 할 토큰을 클릭하면 됩니다.

<figure><img src="../../../.gitbook/assets/image (167).png" alt=""><figcaption><p>토큰 리스트에서 withdraw할 토큰을 선택합니다.</p></figcaption></figure>



3. 토큰을 선택했으면 withdraw하고자 하는 금액을 입력합니다.
4. 이 거래와 관련된 자세한 내용을 보려면 거래 세부 정보 드롭다운을 클릭하세요.
5. 거래 내역이 확인되면 Withdraw 버튼을 클릭하면 최종 확인 화면이 표시됩니다.

<figure><img src="../../../.gitbook/assets/image (120).png" alt=""><figcaption><p> Withdraw 금액을 입력하고 가스 요금을 확인합니다.</p></figcaption></figure>



6. 세부 정보를 검토한 다음 initiate withdraw를 클릭하시면 메타마스크 확인 창을 불러옵니다. Initiate withdraw 과정은 타이탄에서 이더리움으로 자산을 출금하는 데 필요한 4단계 중 첫 번째 단계입니다.

<figure><img src="../../../.gitbook/assets/image (16).png" alt="" width="375"><figcaption></figcaption></figure>

{% hint style="warning" %}
Withdraw는 7일 정도 걸립니다. Initiate withdraw를 실행하고 7일 후, Ethereum으로 자산을 withdraw하기 위해서는 Claim Withdraw를 요청해야 합니다.
{% endhint %}

{% hint style="info" %}
Titan -> Ethereum 자산 Withdraw 절차:

1. Initiate withdraw: Titan에서 Ethereum 브릿지로 출금 요청을 보내기 위해 거래가 전송됩니다.
2. Wait for \~11 minutes for rollup: Titan에서 전송한 거래는 Ethereum 시퀀서에 의해 롤업되어야 합니다. 롤업은 11분 이상 걸릴 수 있습니다.
3. Wait for 7 days: 시퀀서가 악의적이지 않은지 확인하기 위해 롤업된 메시지를 최종화하기 위해 7일을 기다려야 합니다. 이 기간은 챌린지 기간으로도 알려져 있습니다.
4. Claim withdrawal: 이 단계에서는 withdraw가 Ethereum에서 완료되며, 지정된 Ethereum 계정으로 자산이 브릿지에서 이동됩니다.
{% endhint %}



7. initiate withdraw txn이 컨펌된 이후, 이더리움 네트워크에서 transaction과 state root가 롤업될 때까지 약 11분간 기다리세요.

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

8. 이더리움 네트워크에서 시퀀서가 txn과 state root를 롤업한 후, 7일 동안 기다려야 합니다. 대기 기간이 끝나면 알림을 받으려면 Google 캘린더 로고를 클릭하거나 "Google 캘린더에 추가"를 선택하여 알람을 설정할 수 있습니다.

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

9. 7일이 지난 후에, Claim Withdraw를 실행할 수 있습니다. 이 경우 자산이 브릿지에서 지정된 계정으로 이동됩니다.

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>
