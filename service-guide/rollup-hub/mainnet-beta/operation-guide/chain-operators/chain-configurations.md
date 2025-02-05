# Chain Configurations

The Thanos Stack offers a variety of configurable parameters that can be customized to better suit specific operational goals or use cases. While the default settings provided by Optimism are the result of extensive testing and are considered the most stable, making adjustments may be necessary in certain scenarios. However, it's crucial to proceed with caution, as excessive or improper modifications could potentially impact the stability of the network.

{% hint style="info" %}
To estimate operational costs, we've created a worksheet that allows you to adjust parameters impacting fees. Use this tool to refine your network's needs before deployment. Access the worksheet here: [Thanos Stack Operation Fee Adjustment Spreadsheet](https://docs.google.com/spreadsheets/d/1RmyIg38Kkbf7ZFTG5LEqkBeaBRJ6Ohmsyg_mtZhsmDI/edit?gid=0#gid=0)
{% endhint %}



### **1. Configurable FPS Parameters from the Downloaded Config File**

<figure><img src="../../../../../.gitbook/assets/image (12).png" alt=""><figcaption><p>Fig. Generate and download config file</p></figcaption></figure>

**Parameters to Consider**

| `faultGameMaxDepth`         | `faultGameSplitDepth`           | `preimageOracleChallengePeriod`   |
| --------------------------- | ------------------------------- | --------------------------------- |
| `faultGameClockExtension`   | `faultGameWithdrawalDelay`      | `proofMaturityDelaySeconds`       |
| `faultGameMaxClockDuration` | `preimageOracleMinProposalSize` | `disputeGameFinalityDelaySeconds` |

> For detailed explanations of these parameters, refer to the [Link](https://www.notion.so/2-Key-Configuration-Parameters-160d96a400a3818a8de3d30649b23b7c?pvs=21)

#### Warning: Risks of Modifying Parameters

1. **System Stability**:
   * Reducing values like `faultGameMaxDepth` or `faultGameSplitDepth` could compromise the fault proof system’s ability to process complex disputes, leading to incomplete resolutions.
2. **Fairness of Disputes**:
   * Shortening time-related parameters such as `faultGameClockExtension`, `faultGameMaxClockDuration`, or `faultGameWithdrawalDelay` might not provide participants enough time to respond or complete disputes, leading to unfair outcomes.
   * Excessively long durations may cause unnecessary delays in resolution, thereby extending the withdrawal period.
   * Inadequate review periods, such as lowering `preimageOracleChallengePeriod`, could increase the risk of errors or malicious actions going undetected.
3. **Operational Efficiency**:
   * Adjustments to `preimageOracleMinProposalSize` could lead to inefficiencies by allowing trivial submissions or premature actions that are not thoroughly validated.
   * Longer delays in finality and maturity settings (`disputeGameFinalityDelaySeconds`, `proofMaturityDelaySeconds`) could reduce the system’s throughput and slow down operations.

***

## 2. Adjustment of Initial Bond(DisputeGameFactory\_setInitBond)

<figure><img src="../../../../../.gitbook/assets/6 (3).png" alt=""><figcaption></figcaption></figure>

Fig. Safe Wallet sending ‘new transaction’.

> After deploying the DisputeGameFactory contract, you need to configure the `initBonds` value. Follow the steps [here](https://www.notion.so/Deploy-Thanos-chain-ec16c65dc3634b59835660d7fe0a68ea?pvs=21) to set the initial bond for each DisputeGameFactory you create.

**Initial Bond Risks.**

* Setting an insufficiently high `InitialBond` may enable malicious actors to engage in disputes with low stakes, increasing spam or frivolous activities within the system.
* Conversely, an excessively high `InitialBond` could discourage legitimate participants from entering disputes, thereby reducing system fairness and efficiency.

***

### 3. Proposer and Batcher Poll Interval (Helm Deployment):

> To deploy a Helm chart, a values file containing the deployment configurations is required. You can create this file by referring to [this guide](https://www.notion.so/Deploy-Thanos-chain-ec16c65dc3634b59835660d7fe0a68ea?pvs=21). In the `thanos-stack-values.yaml` file, `max_channel_duration` is set to 1500, and `proposal_interval` is set to 21600s. You can adjust these variables to customize the polling intervals for both the batcher and the proposer.

```jsx
// thanos-stack-values.yaml

{
	......

	op_batcher:
	  env:
	    max_channel_duration: 1500
	
	op_proposer:
	  enabled: true
	  env:
	    # l2oo_address: {L2_OUTPUT_ORACLE_ADDRESS}
	    game_factory_address: {GAME_FACTORY_ADDRESS}
	    proposal_interval: 21600s
	    
	......
}
```

* **Proposer `PollInterval`**: Configures the delay between querying L2 for transactions and creating a new **output root proposal**.
  * `thanos-stack-values.yaml`
* **Batcher Poll Interval (`max-channel-duration`)**: Specifies the maximum duration (in L1 blocks) to keep a channel open for batching transactions. For example, if the `max-channel-duration` is set to `30` and the L1 block time is 12 seconds, the batch submission interval would be `30 * 12 = 360 seconds` (6 minutes).

#### Warning on Modifying `PollInterval`

Changes to the `PollInterval` parameter have a direct impact on system performance and operational costs:

* **Short intervals**:
  * May lead to excessive system load and higher costs due to frequent operations.
* **Long intervals**:
  * For batchers, delaying batch submissions to L1 postpones the inclusion of transaction data in finalized batches, which can slow down the finalization of state commitments. This, in turn, delays when users can initiate withdrawals, leading to a sub-optimal user experience and reduced system responsiveness.
  * For proposers, long intervals delay the submission of output roots, which serve as proof for withdrawals. This increases the time users have to wait to initiate their withdrawals, impacting the system's responsiveness and user experience.

***

#### Recommendation

{% hint style="info" %}
Before applying changes to any of the parameters above in a live system, always test modifications in a controlled environment. Adjustments to these parameters can have a significant impact on dispute resolution, withdrawal times, and overall system performance. It’s important to select values that balance performance, cost-efficiency, and user experience while maintaining network stability and reliability.
{% endhint %}
