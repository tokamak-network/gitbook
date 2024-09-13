---
description: 유동성 공급자는 풀에서 유동성을 제공함으로써 스왑 수수료를 받을 수 있습니다.
---

# Pools

{% hint style="info" %}
토카막 브리지의 풀은 유니스왑 v3 컨트랙트를 사용하여 서비스됩니다.&#x20;

* 네트워크가 이더리움 네트워크로 설정된 경우, 공식 유니스왑 v3 컨트랙트를 사용하게 됩니다.&#x20;
* 네트워크가 타이탄 네트워크로 설정되어 있는 경우, 토카막 네트워크에서 배포한 유니스왑 v3 컨트랙트를 사용하게 됩니다.

개발자들은 Uniswap v3의 [개발 가이드](https://docs.uniswap.org/contracts/v3/overview)를 참고하는 것을 권장합니다.
{% endhint %}

{% hint style="danger" %}
Titan 네트워크 정기 유지보수 일정

* Titan: 매주 금요일 9:00 \~ 10:00 KST

네트워크가 다시 가동되면, 타이탄 네트워크가 다운된 동안 L1 시장 가격이 변동한 경우 유동성 포지션의 가격이 급변할 수 있습니다. 타이탄 네트워크에서 유동성 제공을 결정할 때 네트워크 다운타임의 위험을 고려하시기 바랍니다.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (312).png" alt=""><figcaption><p>토카막 브리지의 'Pools'를 위한 예시 서비스 흐름</p></figcaption></figure>

유저가이드:

* [유동성 제공 ](provide-liquidity.md)
* [유동성 증가](increase-liquidity.md)
* [유동성 제거](remove-liquidity.md)
* [수수료 클레임](claim-swap-fee.md)
* 더 자세한 정보를 알고 싶다면 [Uniswap 기사](https://docs.uniswap.org/contracts/v2/concepts/core-concepts/pools)를 참고하세요.&#x20;
