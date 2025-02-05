# Building AWS infra with terraform

To set up the infrastructure, you need to clone the `tokamak-thanos-stack` repository to your local machine.

Visit the link below to clone the repository:

The `tokamak-thanos-stack` repository is structured as follows:

* `charts`: Helm charts for deploying the Thanos chain.
* `terraform`: Terraform code is used to configure the AWS infrastructure to run the Thanos chain.

In this guide, we will use the Terraform code to set up the infrastructure that forms the foundation of the Thanos chain.

## Configure env

Before deploying the infrastructure with Terraform, you need to set up environment variables. we manage the environment variables by creating a `.envrc` file.

1.  Go to terraform directory

    ```bash
    cd terraform
    ```
2.  Copy `.envrc.example` file to `.envrc`.

    ```bash
    cp .envrc.example .envrc
    ```
3. Please fill in the values for the items listed below in the copied file.
   * `TF_VAR_thanos_stack_name`: Name
   * `TF_VAR_aws_region`: The AWS region to use.
   * `TF_VAR_sequencer_key`: The Sequencer Private key.
   * `TF_VAR_batcher_key`: The Batcher Private key.
   * `TF_VAR_proposer_key`: The Proposer Private key.
   * `TF_VAR_challenger_key`: The Challenger Private key.
   * `TF_VAR_eks_cluster_admins`: Your IAM user ARN.
   * `TF_VAR_stack_deployments_path`: The path to the file containing the L1 contract address. You can find it [here](https://www.notion.so/Deploy-contracts-c1da69618ff84f0982922b2d99960044?pvs=21).
   * `TF_VAR_stack_l1_rpc_url`: The L1 RPC endpoint.
   * `TF_VAR_stack_l1_rpc_provider`: The L1 RPC Provider.
   * `TF_VAR_stack_l1_beacon_url`: The beacon RPC endpoint

{% hint style="info" %}
Note to env configure

1. Variables with default values do not need to be modified.
2. You can ignore `TF_VAR_backend_bucket_name` for now, as it will be filled in after configuring the Terraform backend.
3. The ARN of the IAM user can be found in the EC2 console under **Users > User name > Summary** tab.&#x20;
{% endhint %}

**When you've filled it all in, it should look like this:**

```bash
# .envrc file

# Init variables
export TF_VAR_thanos_stack_name=thanos-stack-example
export TF_VAR_aws_region=ap-northeast-2

# Backend variables
export TF_VAR_backend_bucket_name=

export TF_CLI_ARGS_init="-backend-config='bucket=$TF_VAR_backend_bucket_name'"
export TF_CLI_ARGS_init="$TF_CLI_ARGS_init -backend-config='region=${TF_VAR_aws_region}'"

# Sequencer private keys
export TF_VAR_sequencer_key=0xYourSequencerSecretKey
export TF_VAR_batcher_key=0xYourBatcherSecretKey
export TF_VAR_proposer_key=0xYourProposerSecretKey
export TF_VAR_challenger_key=0xYourChallengerSecretKey

# VPC variables
export TF_VAR_azs='["ap-northeast-2a", "ap-northeast-2c"]'
export TF_VAR_vpc_cidr="192.168.0.0/16"
export TF_VAR_vpc_name="${TF_VAR_thanos_stack_name}/VPC"

# EKS
export TF_VAR_eks_cluster_admins='["arn:aws:iam::111111111111:user/thanos-stack"]'

# Uploading chain config files
export TF_VAR_genesis_file_path="config-files/genesis.json"
export TF_VAR_rollup_file_path="config-files/rollup.json"
export TF_VAR_prestate_file_path="config-files/prestate.json"
export TF_VAR_prestate_hash="0x03ab262ce124af0d5d328e09bf886a2b272fe960138115ad8b94fdc3034e3155"

# Thanos config
export TF_VAR_stack_deployments_path="~/tokamak-thanos/packages/tokamak/contracts-bedrock/deployments/1-deploy.json"
export TF_VAR_stack_l1_rpc_url="YourRPCEndpointURL"
export TF_VAR_stack_l1_rpc_provider="alchemy"
export TF_VAR_stack_l1_beacon_url="YourBeaconRPCEndpointURL"
```

4. Save and apply the environment variables using the following command.

```bash
$ direnv allow

direnv: loading ~/Documents/tokamak-network/tokamak-thanos-stack/terraform/.envrc
direnv: export +TF_CLI_ARGS_init +TF_VAR_aws_region +TF_VAR_azs +TF_VAR_backend_bucket_name +TF_VAR_batcher_key +TF_VAR_challenger_key +TF_VAR_eks_cluster_admins +TF_VAR_genesis_file_path +TF_VAR_prestate_file_path +TF_VAR_prestate_hash +TF_VAR_proposer_key +TF_VAR_rollup_file_path +TF_VAR_sequencer_key +TF_VAR_stack_deployments_path +TF_VAR_stack_l1_beacon_url +TF_VAR_stack_l1_rpc_provider +TF_VAR_stack_l1_rpc_url +TF_VAR_thanos_stack_name +TF_VAR_vpc_cidr +TF_VAR_vpc_name
```

## Make backend

Before provisioning AWS resources with Terraform, you need to set up a backend storage to store the state information. The backend storage will use AWS S3.

1.  Go to `backend` directory

    ```bash
    $ cd terraform/backend
    ```
2.  Command `terraform init`

    ```bash
    $ terraform init

    ......

    Terraform has been successfully initialized!

    You may now begin working with Terraform. Try running "terraform plan" to see
    any changes that are required for your infrastructure. All Terraform commands
    should now work.

    If you ever set or change modules or backend configuration for Terraform,
    rerun this command to reinitialize your working directory. If you forget, other
    commands will detect it and remind you to do so if necessary.
    ```
3.  Command `terraform plan` to verify if the backend creation is set up correctly.

    ```bash
    $ terraform plan

    ......

    Plan: 4 to add, 0 to change, 0 to destroy.

    Changes to Outputs:
      + backend_bucket_name = (known after apply)
    ```
4.  Command `terraform apply` to create the backend.

    ```bash
    $ terraform apply

    ......

    Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

    Outputs:

    backend_bucket_name = "thanos-stack-example-thanos-stack-tfstate-avfalb9c"
    direnv: error /Users/austin/Documents/tokamak-network/tokamak-thanos-stack/terraform/.envrc is blocked. Run `direnv allow` to approve its content
    ```

    \<aside> 💡

    The error can be ignored because it occurred after the backend was created and the `TF_VAR_backend_bucket_name` value in the environment file was automatically filled in.

    \</aside>
5.  Go back `terraform` directory and apply the environment variables

    ```bash
    $ cd ..

    $ direnv allow

    direnv: loading ~/Documents/tokamak-network/tokamak-thanos-stack/terraform/.envrc
    direnv: export +TF_CLI_ARGS_init +TF_VAR_aws_region +TF_VAR_azs +TF_VAR_backend_bucket_name +TF_VAR_batcher_key +TF_VAR_challenger_key +TF_VAR_eks_cluster_admins +TF_VAR_genesis_file_path +TF_VAR_prestate_file_path +TF_VAR_prestate_hash +TF_VAR_proposer_key +TF_VAR_rollup_file_path +TF_VAR_sequencer_key +TF_VAR_stack_deployments_path +TF_VAR_stack_l1_beacon_url +TF_VAR_stack_l1_rpc_provider +TF_VAR_stack_l1_rpc_url +TF_VAR_thanos_stack_name +TF_VAR_vpc_cidr +TF_VAR_vpc_name
    ```

## Set up Thanos stack infrastructure

In this phase, we will set up the infrastructure needed to run the apps required for operating the Thanos chain, and to do so, we will create the following resources:

* `vpc`: VPC is a networking service provided by AWS.
* `secretsmanager`: The Secrets Manager stores the private keys used by operator nodes.
* `chain-config`: Storage for genesis file and rollup file. used AWS S3
* `EFS`: EFS is a cloud-based file system service. It is used for the file system storage of Thanos stack services.
* `EKS`: EKS is a managed Kubernetes service. It is used to deploy and manage applications in the Thanos stack.
* `kubernetes`: This module is for deploying controllers to use AWS services in Kubernetes. It deploys controllers for load balancers, coredns, and secrets.

### Bring genesis and rollup files

Before building the infrastructure, make sure to have the genesis and rollup files created during [step 2](https://www.notion.so/Deploy-contracts-c1da69618ff84f0982922b2d99960044?pvs=21).

1.  Go to `thanos-stack` directory

    ```bash
    $ cd terraform/thanos-stack
    ```
2.  Copy generate and rollup files to `config-files` directory.

    ```bash
    $ cp tokamak-thanos/build/genesis.json .

    $ cp tokamak-thanos/build/rollup.json .
    ```

### Building infrastructure

1.  Go to `thanos-stack` directory

    ```bash
    $ cd terraform/thanos-stack
    ```
2.  Command `terraform init`

    ```bash
    $ terraform init

    ......

    Terraform has been successfully initialized!

    You may now begin working with Terraform. Try running "terraform plan" to see
    any changes that are required for your infrastructure. All Terraform commands
    should now work.

    If you ever set or change modules or backend configuration for Terraform,
    rerun this command to reinitialize your working directory. If you forget, other
    commands will detect it and remind you to do so if necessary.
    ```
3.  Command `terraform plan` to verify if the thanos-stack infra creation is set up correctly.

    ```bash
    $ terraform plan

    ......

    Plan: 79 to add, 0 to change, 0 to destroy.

    Changes to Outputs:
      + aws_secretsmanager_id = (known after apply)
      + efs_id                = (known after apply)
      + genesis_file_url      = (known after apply)
      + prestate_file_url     = (known after apply)
      + private_subnet_ids    = [
          + (known after apply),
          + (known after apply),
        ]
      + public_subnet_ids     = [
          + (known after apply),
          + (known after apply),
        ]
      + rollup_file_url       = (known after apply)
      + vpc_id                = (known after apply)
    ```
4.  Command `terraform apply` to create the thanos-stack infra. The process will take around 15 to 20 minutes to complete.

    ```bash
    $ terraform apply

    ......

    Apply complete! Resources: 79 added, 0 changed, 0 destroyed.

    Outputs:

    aws_secretsmanager_id = "arn:aws:secretsmanager:ap-northeast-2:992382494724:secret:guide/secrets-wyxACb"
    efs_id = "fs-016a9cecb8d4fd7fb"
    genesis_file_url = "<https://guide-eb5tpdll.s3.ap-northeast-2.amazonaws.com/thanos-stack/genesis.json>"
    prestate_file_url = "<https://guide-eb5tpdll.s3.ap-northeast-2.amazonaws.com/thanos-stack/0x03ab262ce124af0d5d328e09bf886a2b272fe960138115ad8b94fdc3034e3155.json>"
    private_subnet_ids = [
      "subnet-0392e0a4580bc69ce",
      "subnet-09aaf04937a5d493c",
    ]
    public_subnet_ids = [
      "subnet-0b196fec4038dbd17",
      "subnet-09b558afb248f9644",
    ]
    rollup_file_url = "<https://guide-eb5tpdll.s3.ap-northeast-2.amazonaws.com/thanos-stack/rollup.json>"
    vpc_id = "vpc-0977f0febdd925ef5"
    ```
5.  Once the infrastructure setup is complete, you’ll see that the `thanos-stack-values.yaml` file has been created. This file will be used in the next step to deploy the Thanos chain.

    ```bash
    $ ls

    ... thanos-stack-values.yaml ...
    ```
6.  To interact with EKS, register the EKS kubeconfig with kubectl.

    ```bash
    $ aws eks update-kubeconfig --region $TF_VAR_aws_region --name $TF_VAR_thanos_stack_name

    Added new context arn:aws:eks:ap-northeast-2:111111111111:cluster/thanos-stack-example to /Users/austin/.kube/config
    ```

With the infrastructure setup complete, proceed to the next phase
