# Step-by-Step Guide

### Hardware requirement

* Minimum
  * CPU with 2+ cores
  * 4GB RAM
  * 1TB free storage space to sync the Mainnet
  * 8 MBit/sec download Internet service
* Recommended
  * Fast CPU with 4+ cores
  * 16GB+ RAM
  * High-performance SSD with at least 1TB of free space
  * 25+ MBit/sec download Internet service

While it is possible to deploy with minimum requirements, we encourage you to deploy with hardware that is at least as good as we recommend.



### **Process Guide (for Devnet Deployment)**

Before you start deploying Devnet, make sure you have the dependencies installed. If you don't have at least the minimum version installed, it may not install and work properly even if you follow the guide below and enter the script.

*   Software dependency minimum version information

    We recommend that you use the latest version possible, and at least the minimum version possible.

<table data-header-hidden><thead><tr><th width="189" align="center"></th><th width="340" align="center"></th><th></th></tr></thead><tbody><tr><td align="center"><strong>Dependency</strong></td><td align="center"><strong>Minimum Version</strong></td><td><strong>Version Check Command</strong></td></tr><tr><td align="center"><a href="https://git-scm.com/">git</a></td><td align="center"><code>^2</code></td><td>git --version</td></tr><tr><td align="center"><a href="https://go.dev/">go</a></td><td align="center"><code>^1.21</code> <br><code>(^1.22.7 and earlier is recommended)</code></td><td>go version</td></tr><tr><td align="center"><a href="https://nodejs.org/en/">node</a></td><td align="center"><code>^20</code></td><td>node --version</td></tr><tr><td align="center"><a href="https://pnpm.io/installation">pnpm</a></td><td align="center"><code>^8</code></td><td>pnpm --version</td></tr><tr><td align="center"><a href="https://github.com/foundry-rs/foundry#installation">foundry</a></td><td align="center"><code>0.2.0 (63fff35)</code></td><td>forge --version</td></tr><tr><td align="center"><a href="https://linux.die.net/man/1/make">make</a></td><td align="center"><code>^3</code></td><td>make --version</td></tr><tr><td align="center"><a href="https://github.com/jqlang/jq">jq</a></td><td align="center"><code>^1.6</code></td><td>jq --version</td></tr><tr><td align="center"><a href="https://docs.docker.com/compose/install/">docker compose</a></td><td align="center"><code>^2.26.1</code></td><td>docker compose version</td></tr></tbody></table>

1. Before deploying Devnet, you must first install and version check the required software to run the system. Please make sure you have the appropriate versions installed before proceeding with the deployment.

※ For Mac OS users, we've configured a script to allow for a one-step installation. After you Git clone the repo per Step 2 below, enter the following script to complete the Dependencies installation.

```bash
cd tokamak-thanos
./install-for-mac.sh
```

2. copy the repository below to the PCs you want to deploy locally.

```bash
git clone https://github.com/tokamak-network/tokamak-thanos.git
```

3. Download the JSON file generated based on your input. (_with Download files button_)
4. After downloading, overwrite ‘**./packages/tokamak/contracts-bedrock/deploy-config/devnetL1-template.json**’
5. Go to the repository you cloned and type make build. This will install the various files for the rollup deployment.

```bash
make build
```

6. When all is done, type make devnet-up to deploy the rollup with the information from the build.

```bash
make devnet-up
```



### **Pre-Funded Dev Accounts**

We support known accounts with a sufficient balance in advance to simplify Devnet deployment.

⚠️ These private keys are common knowledge, you should **not** use them on any network other than this dev network. Using these private keys on mainnet, or even a testnet, will most likely result **in a loss of funds**.

<table data-header-hidden><thead><tr><th></th><th width="254"></th><th></th></tr></thead><tbody><tr><td><strong>Address</strong></td><td><strong>Private Key</strong></td><td><strong>Balance</strong></td></tr><tr><td>0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 </td><td>0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80</td><td>10,000 ETH</td></tr><tr><td>0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (<strong>Proposer</strong>)</td><td>0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d</td><td>10,000 ETH</td></tr><tr><td>0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC (<strong>Batcher</strong>)</td><td>0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a</td><td>10,000 ETH</td></tr><tr><td>0x90F79bf6EB2c4f870365E785982E1f101E93b906</td><td>0x7c852118294e51e653712a81e05800f419141751be58f605c371e15141b007a6</td><td>10,000 ETH</td></tr><tr><td>0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65</td><td>0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a</td><td>10,000 ETH</td></tr><tr><td>0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc (<strong>Sequencer</strong>)</td><td>0x8b3a350cf5c34c9194ca85829a2df0ec3153be0318b5e2d3348e872092edffba</td><td>10,000 ETH</td></tr><tr><td>0x976EA74026E726554dB657fA54763abd0C3a0aa9</td><td>0x92db14e403b83dfe3df233f83dfa3a0d7096f21ca9b0d6d6b8d88b2b4ec1564e</td><td>10,000 ETH</td></tr><tr><td>0x14dC79964da2C08b23698B3D3cc7Ca32193d9955</td><td>0x4bbbf85ce3377467afe5d46f804f221813b2bb87f24d81f60f1fcdbf7cbf4356</td><td>10,000 ETH</td></tr><tr><td>0x23618e81E3f5cdF7f54C3d65f7FBc0aBf5B21E8f</td><td>0xdbda1821b80551c9d65939329250298aa3472ba22feea921c0cf5d620ea67b97</td><td>10,000 ETH</td></tr><tr><td>0xa0Ee7A142d267C1f36714E4a8F75612F20a79720 (<strong>Admin</strong>)</td><td>0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6</td><td>10,000 ETH</td></tr></tbody></table>

• Mnemonic: `test test test test test test test test test test test junk`&#x20;

• Derivation path: `m/44'/60'/0'/0/`

