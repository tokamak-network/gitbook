---
description: TON ↔ WTON Etherscan guide
---

# TON ↔ WTON

You can wrap TON into WTON or unwrap WTON back into TON via Etherscan.&#x20;

1. TON → WTON
   1. Step 1 : Approve&#x20;
      1.  Visit [Approve](https://etherscan.io/address/0x2be5e8c109e2197d077d13a82daead6a9b3433c5#writeContract#F2)

          <figure><img src="../.gitbook/assets/image (369).png" alt=""><figcaption><p>Approve section of Etherscan</p></figcaption></figure>


      2. Spender : [0xc4A11aaf6ea915Ed7Ac194161d2fC9384F15bff2](https://etherscan.io/address/0xc4A11aaf6ea915Ed7Ac194161d2fC9384F15bff2) (fix value, WTON Contract)
      3. Amount : Amount to swap TON to WTON (TON decimal is 18)
         * Example : 1000000000000000000 (1 TON)
   2. Step 2 : swapFromTON&#x20;
      1.  Visit [swapFromTON ](https://etherscan.io/address/0xc4a11aaf6ea915ed7ac194161d2fc9384f15bff2#writeContract#F18)

          <figure><img src="../.gitbook/assets/image (370).png" alt=""><figcaption><p>swapFromTON section of Etherscan</p></figcaption></figure>


      2. tonAmount : Enter the amount you want to swap TON to WTON (it must not be greater than the value you approved in Step 1).\

2. WTON → TON
   1.  Visit [swapToTON](https://etherscan.io/address/0xc4a11aaf6ea915ed7ac194161d2fc9384f15bff2#writeContract#F20)&#x20;

       <figure><img src="../.gitbook/assets/image (371).png" alt=""><figcaption><p>swapToTON of Etherscan</p></figcaption></figure>


   2. wtonAmount : Amount to swap WTON to TON (WTON decimal is 27)
      * Example : 1000000000000000000000000000 (1 WTON)
