<div align="center">

# ⛓️ Worrell Testnet — Full Node & Validator Setup Guide

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

## 📚 Guide

| Type | Language | Link |
|------|----------|------|
| Full Node & Validator Setup | 🇬🇧 English | [testnet-en.md](testnet-en.md) |

---

## 📋 Overview

Worrell is a sovereign proof-of-stake blockchain built on the Cosmos SDK (v0.53.6) and CometBFT, purpose-built for payments and energy infrastructure rather than as a general-purpose chain. It targets fast, low-fee value transfer (min gas price `0.025uworrell`, ~5–6s block time) and is designed as settlement/coordination infrastructure for energy markets and machine-to-machine payments, with IBC ready to be enabled by governance once the network stabilizes.

The testnet (`worrell-testnet-1`) is live and open to node operators — up to 100 validator seats, 80% of the fixed 1,000,000,000 WORRELL genesis supply allocated to the community (treasury, airdrop, incentives, reserve), with founder tokens locked under a 4-year vesting schedule with a 1-year cliff.

- Official Website: [worrellchain.com](https://worrellchain.com)
- Node Source: [github.com/worrellchain/worrell](https://github.com/worrellchain/worrell)
- Network Configs / Genesis: [github.com/worrellchain/networks](https://github.com/worrellchain/networks)
- Validator Guide (upstream): [worrellchain/worrell — docs/RUNNING-A-NODE.md](https://github.com/worrellchain/worrell/blob/main/docs/RUNNING-A-NODE.md)
- Explorer: [test.anode.team/worrell](https://test.anode.team/worrell) (community-run by ANODE.TEAM)
- Public REST API: [worrell.api.t.anode.team](https://worrell.api.t.anode.team) (community-run by ANODE.TEAM)
- Faucet: `POST http://164.68.98.186:4500` with `{"address":"worrell1..."}` (500 WORRELL, rate-limited to once/hour per address)
- Validator Announcements: Telegram [t.me/worrellvalidators](https://t.me/worrellvalidators)
- Support: [GitHub Discussions](https://github.com/worrellchain/worrell/discussions) · [hello@worrellchain.com](mailto:hello@worrellchain.com)

### Network Facts

| Field | Value |
|---|---|
| Chain ID | `worrell-testnet-1` |
| Binary | `worrelld` (Cosmos SDK v0.53.6, CometBFT) |
| Token | WORRELL (base denom `uworrell`, 6 decimals) |
| Bech32 prefix | `worrell` |
| Min gas price | `0.025uworrell` |
| Genesis sha256 | `a81c507b12ba0678c3172394ff4bb03e1c3db60050cc5568c127a24ec19378fd` |
| Persistent peer | `bb9164c1bd9ed9ff2c0fd9e09b23285698e231de@164.68.98.186:26656` |
| Max validators | 100 |
| Min commission (global) | 5% |
| Unbonding period | 1 hour (testnet) / 21 days (mainnet) |

> ⚠️ **Bu bir 3. parti (community) referans dokümandır**, Worrell'in resmi ekibiyle bağlantılı değildir. Kaynaklar: [worrellchain/networks](https://github.com/worrellchain/networks) ve [worrellchain.com](https://worrellchain.com) (resmi), ayrıca bir validatörün hazırladığı [docs.oshvank.xyz/docs/testnet/Worrel](https://docs.oshvank.xyz/docs/testnet/Worrel) çapraz kontrol için kullanıldı. Chain hâlâ genç ve aktif geliştirme altında — genesis, peer listesi, sürüm ve endpoint'ler zamanla değişebilir; kritik komutlardan önce [worrellchain/networks](https://github.com/worrellchain/networks) reposundan güncel bilgiyi teyit et.

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
