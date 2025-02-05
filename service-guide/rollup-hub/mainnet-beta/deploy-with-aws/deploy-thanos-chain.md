# Deploy Thanos chain

In this step, we will deploy the Thanos chain. The Thanos stack is configured using a Helm chart for deployment convenience.

## Add Helm Repository

1.  Add helm repo from tokamak-thanos-stack

    ```bash
    $ helm repo add thanos-stack <https://tokamak-network.github.io/tokamak-thanos-stack>
    ```
2.  You can check helm repo

    ```bash
    $ helm search repo thanos-stack

    NAME                            CHART VERSION   APP VERSION     DESCRIPTION
    thanos-stack/thanos-stack       {VERSION}                       A Helm chart to deploy Thanos stack
    thanos-stack/op-bridge          {VERSION}                       A Helm chart to deploy Optimistic bridge
    ```

## Deploy Thanos chain

To deploy the Thanos chain using the Helm chart, you’ll need a file with the chart parameters. We’ll use the `thanos-stack-values.yaml` file created in [step 3](https://www.notion.so/Building-AWS-infra-with-terraform-99b7d31fae274c06b1f44b14a14c42e3?pvs=21) to proceed with the deployment.

1.  Helm install

    ```bash
    $ helm install {YOUR_HELM_RELEASE_NAME} thanos-stack/thanos-stack \
    		--values thanos-stack-values.yaml \
    		--namespace {YOUR_NAMESPACE}
    ```

{% hint style="info" %}
The namespace must be set to the Thanos stack name configured in [step 3](https://www.notion.so/Building-AWS-infra-with-terraform-99b7d31fae274c06b1f44b14a14c42e3?pvs=21).
{% endhint %}

2. Check pods.

```bash
$ kubectl -n {YOUR_NAMESPACE} get pods

NAME                                                                  READY   STATUS    RESTARTS   AGE
{YOUR_HELM_REALEASE_NAME}-thanos-stack-op-proposer-343224a2dc8-kw9sm  1/1     Running   0          15m
{YOUR_HELM_REALEASE_NAME}-thanos-stack-op-batcher-57784b95c8-n6zmt    1/1     Running   0          15m
{YOUR_HELM_REALEASE_NAME}-thanos-stack-op-geth-0                      1/1     Running   0          15m
{YOUR_HELM_REALEASE_NAME}-thanos-stack-op-node-0                      1/1     Running   0          15m
```

3. Check `op-node` logs. During the initial deployment, it synchronizes with the L1 chain, by the following logs.

```bash
$ kubectl -n {YOUR_NAMESPACE} logs -f {YOUR_HELM_REALEASE_NAME}-thanos-stack-op-node-0

t=2024-12-16T02:28:43+0000 lvl=info msg="Advancing bq origin" origin=0xfe71755a024cbd36b7b5bc051b7529f5b1c6fdf0765e2084d72870f410b3ce98:7287497 originBehind=false
t=2024-12-16T02:28:44+0000 lvl=info msg="Advancing bq origin" origin=0x82a7a5dd8b09226fd7b4a5c77b1aca55cc0778e695f1de1f9ada16b3ae96e3b6:7287498 originBehind=false
t=2024-12-16T02:28:46+0000 lvl=info msg="Advancing bq origin" origin=0xd3381dbfb2c8bb744d8e7c3a5da3f6fdf8127c4bbd2f983e6fe08040f5b90f9b:7287499 originBehind=false
t=2024-12-16T02:28:47+0000 lvl=info msg="Advancing bq origin" origin=0xcf5e9c66d7bf78dea7dc32ad73c80a8e4c6b35110f170c38545260364d4143f3:7287500 originBehind=false
t=2024-12-16T02:28:48+0000 lvl=info msg="Advancing bq origin" origin=0x94712bc35fa4a38f7028de2f65ae438d7fa681391434bf94f5bf060f42d51736:7287501 originBehind=false
t=2024-12-16T02:28:50+0000 lvl=info msg="Advancing bq origin" origin=0x5756eca87dd60f4cc1519a15a0ea2e4746faa5ca4a56457c7b9fd8cb72080ff4:7287502 originBehind=false
t=2024-12-16T02:28:51+0000 lvl=info msg="Advancing bq origin" origin=0x63d92d6b4115bba5f04223f8e6e5552105228adefe23abf6011dcde2daeab62c:7287503 originBehind=false
t=2024-12-16T02:28:52+0000 lvl=info msg="Advancing bq origin" origin=0xf9d619fb930fec707e93017a6c55cefb70aaf8e92c52766790299dafd402f55f:7287504 originBehind=false
t=2024-12-16T02:28:54+0000 lvl=info msg="Advancing bq origin" origin=0x89d282d1b84328ee5cc09339e4cb9f0074a74915a2509694a2aed1e5a624e0c8:7287505 originBehind=false
```

## Chain RPC

You can interact with the thanos chain through the address provided by Ingress.

1.  Get ingress

    ```bash
    $ kubectl -n {YOUR_NAMESPACE} get ingress

    NAME                                                     CLASS   HOSTS   ADDRESS                                                             PORTS   AGE
    {YOUR_HELM_REALEASE_NAME}-thanos-stack-op-geth-ingress   alb     *       k8s-opgeth-dc725455d2-1354467471.ap-northeast-2.elb.amazonaws.com   80      3m49s
    ```
