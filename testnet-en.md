<div align="center">

# ⛓️ Worrell Testnet Full Node & Validator Setup Guide

**A complete guide to running a Worrell testnet full node and registering as a validator**
*Prebuilt binary or build from source, genesis, peers, systemd service, and validator creation — step by step.*

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Worrell](https://img.shields.io/badge/Worrell-Testnet-6C4DF6?style=flat-square)](https://worrellchain.com)
[![Version](https://img.shields.io/badge/Node%20Version-v0.1.2-brightgreen?style=flat-square)](https://github.com/worrellchain/worrell/releases/tag/v0.1.2)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-worrell--testnet--1-blue?style=flat-square)](https://github.com/worrellchain/networks)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions
> **Network:** Worrell Testnet (Chain ID: worrell-testnet-1)
> **Version:** v0.1.2
> **Last Updated:** September 2026

---

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Network Endpoints](#network-endpoints)
- [Step 1 — System Verification](#step-1--system-verification)
- [Step 2 — System Update and Dependencies](#step-2--system-update-and-dependencies)
- [Step 3 — Install worrelld (Prebuilt Binary)](#step-3--install-worrelld-prebuilt-binary)
- [Step 3 Alt — Build worrelld from Source](#step-3-alt--build-worrelld-from-source)
- [Step 4 — Initialize the Node](#step-4--initialize-the-node)
- [Step 5 — Download and Verify Genesis](#step-5--download-and-verify-genesis)
- [Step 6 — Configure Peers (config.toml)](#step-6--configure-peers-configtoml)
- [Step 7 — Set the Minimum Gas Price (app.toml)](#step-7--set-the-minimum-gas-price-apptoml)
- [Step 8 — Configure Custom Ports (Optional)](#step-8--configure-custom-ports-optional)
- [Step 9 — Enable Prometheus (Optional)](#step-9--enable-prometheus-optional)
- [Step 10 — Create Systemd Service](#step-10--create-systemd-service)
- [Step 11 — Start the Node](#step-11--start-the-node)
- [Step 12 — Verify Sync](#step-12--verify-sync)
- [Step 13 — Create a Wallet and Fund It (Faucet)](#step-13--create-a-wallet-and-fund-it-faucet)
- [Step 14 — Register as a Validator](#step-14--register-as-a-validator)
- [Monitoring & Avoiding Jailing](#monitoring--avoiding-jailing)
- [Useful Commands](#useful-commands)
- [Firewall](#firewall)

---

## Hardware Requirements

| Component        | Minimum (Testnet) |
| ---------------- | ------------------ |
| Operating System | Ubuntu 22.04 LTS   |
| CPU              | 2 vCPU             |
| RAM              | 4 GB               |
| Disk             | 100 GB SSD         |
| Network          | Stable connection, public IP for P2P |

> Mainnet, when it launches, will require more headroom (4+ vCPU, 16+ GB RAM, 500 GB+ NVMe SSD) since a validator must sign blocks continuously and keep history. This guide targets the **testnet**.

---

## Network Endpoints

| Type            | Endpoint                                                                 |
| ---------------- | ------------------------------------------------------------------------ |
| Chain ID          | `worrell-testnet-1`                                                     |
| REST API          | https://worrell.api.t.anode.team (community, by ANODE.TEAM)            |
| Explorer          | https://test.anode.team/worrell (community, by ANODE.TEAM)             |
| Genesis           | https://raw.githubusercontent.com/worrellchain/networks/main/worrell-testnet-1/genesis.json |
| Genesis sha256    | `a81c507b12ba0678c3172394ff4bb03e1c3db60050cc5568c127a24ec19378fd`      |
| Persistent peer   | `bb9164c1bd9ed9ff2c0fd9e09b23285698e231de@164.68.98.186:26656`          |
| Faucet            | `POST http://164.68.98.186:4500` (500 WORRELL, 1/hour per address)      |
| Node source       | https://github.com/worrellchain/worrell                                |
| Network configs   | https://github.com/worrellchain/networks                               |
| Website           | https://worrellchain.com                                               |
| Validator alerts  | https://t.me/worrellvalidators                                         |

---

## Step 1 — System Verification

After SSH-ing into your server, verify the system meets requirements:

```bash
lsb_release -a
uname -r
lscpu | grep -E "Model name|CPU\(s\)|Thread|Socket|Core"
free -h
df -h
```

---

## Step 2 — System Update and Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl build-essential jq wget tmux htop unzip
```

---

## Step 3 — Install worrelld (Prebuilt Binary)

The `worrelld` binary is statically linked and self-contained — no Go toolchain required. This is the fastest way to get running. Pick the tarball matching your platform from the [releases page](https://github.com/worrellchain/worrell/releases/tag/v0.1.2):

| Platform (`uname -s` / `uname -m`) | Tarball |
|---|---|
| `Linux` / `x86_64` | `v0.1.2_linux_amd64.tar.gz` |
| `Linux` / `aarch64` | `v0.1.2_linux_arm64.tar.gz` |
| `Darwin` / `arm64` (Apple Silicon) | `v0.1.2_darwin_arm64.tar.gz` |
| `Darwin` / `x86_64` (Intel Mac) | `v0.1.2_darwin_amd64.tar.gz` |

```bash
cd $HOME
curl -LO https://github.com/worrellchain/worrell/releases/download/v0.1.2/v0.1.2_linux_amd64.tar.gz

# Verify against the release_checksum file published on the releases page before extracting
sha256sum v0.1.2_linux_amd64.tar.gz

tar -xzf v0.1.2_linux_amd64.tar.gz
mkdir -p ~/bin && mv worrelld ~/bin/
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
worrelld version --long | head -5
# should report cosmos_sdk_version: v0.53.6
worrelld --help
```

> If you prefer to build (and audit) the binary yourself instead of using the prebuilt tarball, see the alternative step below and skip this one.

---

## Step 3 Alt — Build worrelld from Source

Dependencies: **Go 1.25+** (check `go.mod` in the repo for the exact minimum), `git`, `make`, `build-essential`.

```bash
sudo apt update && sudo apt install -y git curl build-essential
```

Install Go 1.25:

```bash
curl -LO https://go.dev/dl/go1.25.13.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.25.13.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.profile
source ~/.profile
go version
```

Clone and build the published release tag:

```bash
git clone https://github.com/worrellchain/worrell.git
cd worrell
git checkout v0.1.2
make install
```

`make install` compiles `worrelld` and places it in `$HOME/go/bin`.

```bash
echo 'export PATH=$PATH:$HOME/go/bin' >> ~/.profile
source ~/.profile
worrelld version --long | head -5
```

---

## Step 4 — Initialize the Node

Pick a moniker (your node's public display name) and initialize the data directory:

```bash
MONIKER="YOUR_MONIKER"

worrelld init "$MONIKER" --chain-id worrell-testnet-1

echo "export MONIKER=$MONIKER" >> $HOME/.bashrc
echo "export WORRELL_CHAIN_ID=\"worrell-testnet-1\"" >> $HOME/.bashrc
echo "export WORRELL_HOME=\"$HOME/.worrell\"" >> $HOME/.bashrc
source $HOME/.bashrc
```

> ℹ️ Replace `YOUR_MONIKER` with your own node display name.

This creates `~/.worrell` with:

- `config/genesis.json` — genesis (replaced in the next step)
- `config/config.toml` — CometBFT configuration (peers, P2P, RPC)
- `config/app.toml` — application configuration (gas, API, gRPC)
- `config/priv_validator_key.json` — validator signing key (**protect it!**)
- `config/node_key.json` — node network identity

---

## Step 5 — Download and Verify Genesis

Pull the canonical testnet genesis from the official [`worrellchain/networks`](https://github.com/worrellchain/networks) repo and verify the checksum before continuing:

```bash
curl -s https://raw.githubusercontent.com/worrellchain/networks/main/worrell-testnet-1/genesis.json \
  -o ~/.worrell/config/genesis.json

worrelld genesis validate-genesis
```

Verify the sha256 checksum:

```bash
sha256sum ~/.worrell/config/genesis.json
# expected: a81c507b12ba0678c3172394ff4bb03e1c3db60050cc5568c127a24ec19378fd
```

> ⚠️ If the checksum does not match, **do not continue**. Re-download the genesis file from the [official repo](https://github.com/worrellchain/networks/blob/main/worrell-testnet-1/genesis.json).

---

## Step 6 — Configure Peers (config.toml)

Set the persistent peer in the `[p2p]` section of `config.toml`. The up-to-date peer list is published in [`worrell-testnet-1/chain.json`](https://github.com/worrellchain/networks/blob/main/worrell-testnet-1/chain.json) (`peers` section) — check it if this guide is older than a few weeks.

```bash
sed -i.bak 's|^persistent_peers *=.*|persistent_peers = "bb9164c1bd9ed9ff2c0fd9e09b23285698e231de@164.68.98.186:26656"|' "$WORRELL_HOME/config/config.toml"
```

Optional — advertise your public IP for inbound P2P:

```bash
# Replace <PUBLIC_IP> with your actual server IP
sed -i.bak -e "s|^external_address *=.*|external_address = \"<PUBLIC_IP>:26656\"|" "$WORRELL_HOME/config/config.toml"
```

Verify:

```bash
grep -E '^(persistent_peers|external_address) ' "$WORRELL_HOME/config/config.toml"
```

---

## Step 7 — Set the Minimum Gas Price (app.toml)

This is **mandatory** for the node to accept and process transactions:

```bash
sed -i.bak 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025uworrell"|' "$WORRELL_HOME/config/app.toml"
```

Recommended additional settings:

```bash
# Pruning (use "nothing" for archive nodes)
sed -i -e 's/^pruning *=.*/pruning = "default"/' "$WORRELL_HOME/config/app.toml"
sed -i -e 's/^pruning-keep-recent *=.*/pruning-keep-recent = "100"/' "$WORRELL_HOME/config/app.toml"
sed -i -e 's/^pruning-interval *=.*/pruning-interval = "10"/' "$WORRELL_HOME/config/app.toml"
```

By default the REST API (`1317`) and gRPC (`9090`) listen only on `localhost` — leave them that way unless you intend to serve public endpoints behind a reverse proxy.

---

## Step 8 — Configure Custom Ports (Optional)

Useful if you plan to run more than one node on the same server. Enter a 2-digit prefix (e.g. `46`) and rewrite the relevant ports in both files:

```bash
read -p "Enter your PORT prefix (2-digit, e.g. 46): " WORRELL_PORT
echo "export WORRELL_PORT=$WORRELL_PORT" >> $HOME/.bashrc
source $HOME/.bashrc
```

Apply to `app.toml`:

```bash
sed -i.bak -e "s%:1317%:${WORRELL_PORT}317%g" \
           -e "s%:9090%:${WORRELL_PORT}090%g" \
           -e "s%:9091%:${WORRELL_PORT}091%g" \
           "$WORRELL_HOME/config/app.toml"
```

Apply to `config.toml`:

```bash
sed -i.bak -e "s%:26657%:${WORRELL_PORT}657%g" \
           -e "s%:26656%:${WORRELL_PORT}656%g" \
           -e "s%:26660%:${WORRELL_PORT}660%g" \
           "$WORRELL_HOME/config/config.toml"
```

Verify:

```bash
grep -E ":(${WORRELL_PORT})" "$WORRELL_HOME/config/config.toml" | head -5
grep -E ":(${WORRELL_PORT})" "$WORRELL_HOME/config/app.toml" | head -5
```

> If you skip this step, the defaults (`26656`/`26657`/`1317`/`9090`) are used throughout the rest of this guide.

---

## Step 9 — Enable Prometheus (Optional)

```bash
sed -i -e "s/prometheus = false/prometheus = true/" "$WORRELL_HOME/config/config.toml"
sed -i -e 's/^prometheus_listen_addr *=.*/prometheus_listen_addr = ":26660"/' "$WORRELL_HOME/config/config.toml"
```

Metrics become available at `http://127.0.0.1:26660/metrics`.

---

## Step 10 — Create Systemd Service

Running the node under `systemd` with automatic restart is strongly recommended — a node that restarts quickly after a crash is much less likely to be jailed for downtime.

```bash
sudo bash -c "cat > /etc/systemd/system/worrelld.service" << EOF
[Unit]
Description=Worrell node
After=network-online.target
Wants=network-online.target

[Service]
User=$USER
ExecStart=$HOME/bin/worrelld start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable worrelld
```

> Adjust `ExecStart` to `$HOME/go/bin/worrelld start` if you built from source instead of using the prebuilt binary.

---

## Step 11 — Start the Node

```bash
sudo systemctl start worrelld
sudo journalctl -u worrelld -f --no-pager -o cat
```

Verify the service is running:

```bash
sudo systemctl status worrelld --no-pager
```

Expected output:
```
● worrelld.service - Worrell node
     Active: active (running) since ...
```

---

## Step 12 — Verify Sync

```bash
worrelld status 2>&1 | jq '.sync_info | {latest_block_height, catching_up}'
```

Compare against the public REST API / explorer:

```bash
curl -s https://worrell.api.t.anode.team/cosmos/base/tendermint/v1beta1/blocks/latest \
  | jq '.block.header.height'
```

> ⚠️ Wait until `catching_up` is `false` before creating a validator. The network has State Sync enabled, allowing fast synchronization without downloading the full history — see the [official validator guide](https://github.com/worrellchain/worrell/blob/main/docs/RUNNING-A-NODE.md) if you want to configure it.

---

## Step 13 — Create a Wallet and Fund It (Faucet)

```bash
worrelld keys add wallet
```

> **CRITICAL:** Save your mnemonic phrase in a secure location. Without it, you cannot recover your wallet.

To recover an existing wallet:

```bash
worrelld keys add wallet --recover
```

Check your address:

```bash
worrelld keys show wallet -a
```

Request testnet funds from the faucet (500 WORRELL per request, rate-limited to once per hour per address):

```bash
curl -X POST http://164.68.98.186:4500 \
  -H "Content-Type: application/json" \
  -d "{\"address\":\"$(worrelld keys show wallet -a)\"}"
```

Check your balance:

```bash
worrelld query bank balances $(worrelld keys show wallet -a) \
  --node https://worrell.api.t.anode.team 2>/dev/null \
  || worrelld query bank balances $(worrelld keys show wallet -a)
```

---

## Step 14 — Register as a Validator

> The node must be **fully synchronized** (`catching_up: false`) before creating a validator.

### 14.1 Get the consensus public key

```bash
worrelld tendermint show-validator
```

Returns a value like `{"@type":"/cosmos.crypto.ed25519.PubKey","key":"..."}`.

### 14.2 Create the validator JSON

```bash
cat > $HOME/validator.json << EOF
{
  "pubkey": $(worrelld tendermint show-validator),
  "amount": "20000000000000uworrell",
  "moniker": "$MONIKER",
  "identity": "",
  "website": "",
  "security": "",
  "details": "Worrell testnet validator",
  "commission-rate": "0.05",
  "commission-max-rate": "0.25",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1000000"
}
EOF
```

### 14.3 Submit the transaction

```bash
worrelld tx staking create-validator $HOME/validator.json \
  --from wallet \
  --chain-id worrell-testnet-1 \
  --gas auto \
  --gas-adjustment 1.5 \
  --gas-prices 0.025uworrell \
  --yes
```

### 14.4 Field reference

| Flag / field | Example value | Meaning |
|--------------|-------------------|-------------|
| `amount` | `20000000000000uworrell` | Initial self-delegation (20,000,000 WORRELL in this example). Adjust to your case and available balance. |
| `commission-rate` | `0.05` | Current commission: 5%. **Cannot be lower than the global network minimum (5%).** |
| `commission-max-rate` | `0.25` | Maximum commission this validator allows itself. **Cannot be raised later** — choose carefully. |
| `commission-max-change-rate` | `0.01` | Maximum commission change per day: 1%. |
| `min-self-delegation` | `1000000` | Minimum self-delegation to maintain, in `uworrell`. `1000000 uworrell = 1 WORRELL`. |

> ⚠️ **Critical:** `min-self-delegation` is expressed in `uworrell`, **not** WORRELL. To guarantee 1 WORRELL minimum, use `"1000000"`. **Never** use `"1"` — that means 0.000001 WORRELL and leaves your validator far below the intended minimum.

> ℹ️ The **5% (`0.05`) minimum commission is global** — no validator can go below it. The maximum commission and max daily change rate are set per-validator at creation time and the max **cannot be increased afterward**.

### 14.5 Verify the validator

```bash
worrelld query staking validator $(worrelld keys show wallet --bech val -a)
```

Confirm `status` is `BOND_STATUS_BONDED` and `jailed` is `false`.

---

## Monitoring & Avoiding Jailing

### Relevant slashing parameters

| Parameter | Value | Meaning |
|-----------|-------|---------|
| Signed-blocks window | 10,000 blocks | Period over which availability is measured |
| Minimum signed per window | 5% | Must sign at least 5% of blocks in the window |
| Downtime slashing | 0.01% | Slash on jailing for inactivity |
| Double-sign slashing | 5% + permanent tombstone | Slash for signing two blocks at the same height |
| Jail duration (downtime) | 1 hour on testnet / 12h on mainnet | Minimum time jailed |

To avoid jailing due to downtime:

- Run the node under `systemd` with automatic restart (Step 10).
- Keep disk/CPU headroom — a node that can't keep up misses blocks.
- **Never** run two instances with the same `priv_validator_key.json` at once — this causes a **double sign** and a 5% + permanent tombstone slash, far worse than downtime jailing.
- Enable Prometheus (Step 9) and set up alerts on missed blocks.
- Subscribe to [t.me/worrellvalidators](https://t.me/worrellvalidators) for upgrade/governance announcements.

### Check signing info

```bash
worrelld query slashing signing-info $(worrelld tendermint show-address)
```

Watch `missed_blocks_counter` and `jailed_until`.

### Unjail after downtime

```bash
worrelld tx slashing unjail \
  --from wallet \
  --chain-id worrell-testnet-1 \
  --gas auto \
  --gas-adjustment 1.5 \
  --gas-prices 0.025uworrell \
  --yes
```

---

## Useful Commands

### Service Management

```bash
sudo systemctl start worrelld
sudo systemctl stop worrelld
sudo systemctl restart worrelld
sudo systemctl status worrelld
```

### Logs & Sync

```bash
# Live logs
sudo journalctl -u worrelld -f --no-pager -o cat

# Logs from last hour
sudo journalctl -u worrelld --since "1 hour ago"

# Sync status
worrelld status 2>&1 | jq '.sync_info | {latest_block_height, catching_up}'

# Connected peers
worrelld status 2>&1 | jq '.node_info.moniker'
```

### Wallet & Balance

```bash
# List wallets
worrelld keys list

# Show address
worrelld keys show wallet -a

# Check balance
worrelld query bank balances $(worrelld keys show wallet -a)
```

### Staking

```bash
# Delegate tokens
worrelld tx staking delegate \
  $(worrelld keys show wallet --bech val -a) 1000000uworrell \
  --from wallet --chain-id worrell-testnet-1 \
  --gas auto --gas-adjustment 1.5 --gas-prices 0.025uworrell -y

# Redelegate tokens
worrelld tx staking redelegate \
  $(worrelld keys show wallet --bech val -a) \
  <NEW_VALOPER> 1000000uworrell \
  --from wallet --chain-id worrell-testnet-1 \
  --gas auto --gas-adjustment 1.5 --gas-prices 0.025uworrell -y

# Undelegate tokens
worrelld tx staking unbond \
  $(worrelld keys show wallet --bech val -a) 1000000uworrell \
  --from wallet --chain-id worrell-testnet-1 \
  --gas auto --gas-adjustment 1.5 --gas-prices 0.025uworrell -y
```

### Rewards

```bash
# Withdraw all rewards
worrelld tx distribution withdraw-all-rewards \
  --from wallet --chain-id worrell-testnet-1 \
  --gas auto --gas-adjustment 1.5 --gas-prices 0.025uworrell -y

# Withdraw commission
worrelld tx distribution withdraw-rewards \
  $(worrelld keys show wallet --bech val -a) \
  --commission \
  --from wallet --chain-id worrell-testnet-1 \
  --gas auto --gas-adjustment 1.5 --gas-prices 0.025uworrell -y
```

### Governance

```bash
# List proposals
worrelld query gov proposals

# Vote on a proposal
worrelld tx gov vote 1 yes \
  --from wallet --chain-id worrell-testnet-1 \
  --gas auto --gas-adjustment 1.5 --gas-prices 0.025uworrell -y
```

Governance parameters:

| Parameter | Standard | Expedited |
|---|---|---|
| Minimum deposit | 1,500 WORRELL | 7,500 WORRELL |
| Voting period (mainnet) | 5 days | 24 hours |
| Quorum | 33.4% | 33.4% |
| Pass threshold | 50% | 66.7% |
| Veto threshold | 33.4% | 33.4% |

### Validator Operations

```bash
# Edit validator
worrelld tx staking edit-validator \
  --new-moniker "NEW_MONIKER" \
  --identity "YOUR_KEYBASE_ID" \
  --website "https://your-website.com" \
  --security-contact "your@email.com" \
  --details "Your validator description" \
  --from wallet --chain-id worrell-testnet-1 \
  --gas auto --gas-adjustment 1.5 --gas-prices 0.025uworrell -y

# Unjail validator
worrelld tx slashing unjail \
  --from wallet --chain-id worrell-testnet-1 \
  --gas auto --gas-adjustment 1.5 --gas-prices 0.025uworrell -y

# Check signing info
worrelld query slashing signing-info $(worrelld tendermint show-address)
```

---

## Firewall

The P2P port must be reachable from outside; keep RPC/API/gRPC restricted to localhost or trusted IPs unless you intend to serve public endpoints behind a reverse proxy.

```bash
# P2P — must be open to the public
sudo ufw allow 26656/tcp comment "worrell P2P"

# RPC — open only if you serve public endpoints
sudo ufw allow 26657/tcp comment "worrell RPC"

# REST API
sudo ufw allow 1317/tcp comment "worrell REST"

# gRPC
sudo ufw allow 9090/tcp comment "worrell gRPC"
```

(Adjust the port numbers if you applied the custom prefix in Step 8.)

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
