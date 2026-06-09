# 🔐 Secure Dark Auction with Multi-Party Computation

A privacy-preserving sealed-bid double auction (dark pool) built on top of the [MP-SPDZ](https://github.com/data61/MP-SPDZ) framework, developed as part of the Communications Security course at Instituto Superior Técnico, Universidade de Lisboa (2025/2026).

Three parties jointly execute the auction without any single party learning the others' private orders — only the final market outcome is revealed.

---

## Table of Contents

- [Overview](#overview)
- [MPC Protocols](#mpc-protocols)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Running the Auction](#running-the-auction)

---

## Overview

A **dark auction** (or dark pool call auction) is a financial market mechanism where buy and sell orders are submitted secretly and matched simultaneously at a single uniform clearing price. This project implements such a mechanism using **Secure Multi-Party Computation (MPC)**, so that no party — not even the one running the computation — can learn the private orders of others.

**Key properties:**
- Three parties each hold private orders for three assets: BTC, ETH, and SOL
- Orders are never revealed; only the clearing price, traded volume, and individual fills are disclosed
- The computation is distributed across all three parties using MP-SPDZ

---

## MPC Protocols

The project supports four MP-SPDZ protocols with different security and performance trade-offs:

| Protocol | Binary | Security Model | Trust Assumption | Main Technique |
|---|---|---|---|---|
| **MASCOT** | `mascot-party.x` | Malicious | Dishonest majority | OT-based preprocessing with authenticated triples |
| **Shamir** | `shamir-party.x` | Semi-honest | Honest majority | Shamir secret sharing |
| **Semi** | `semi-party.x` | Semi-honest | Dishonest majority | OT-based triple generation (no MACs) |
| **Replicated Ring** | `replicated-ring-party.x` | Semi-honest | Honest majority (3-party) | Replicated secret sharing mod 2^k |

---

## Project Structure

```
.
├── Dockerfile                  # Container with MP-SPDZ and all protocols
├── docker-compose.yml          # 3-party Docker Compose setup
├── Config/
│   └── IPs                     # Party IP addresses for the Docker network
├── inputs/
│   ├── party0.txt              # Human-readable orders for party 0
│   ├── party1.txt              # Human-readable orders for party 1
│   └── party2.txt              # Human-readable orders for party 2
├── Inputs/
│   ├── Input-P0-0              # MP-SPDZ input file for party 0 (auto-generated)
│   ├── Input-P1-0              # MP-SPDZ input file for party 1 (auto-generated)
│   └── Input-P2-0              # MP-SPDZ input file for party 2 (auto-generated)
├── scripts/
│   ├── generate_inputs.py      # Converts human-readable inputs to MP-SPDZ format
│   ├── test_run.sh             # Quick single-protocol test runner
│   ├── run_auction.sh          # Multi-protocol compile + run script
│   └── benchmark.sh            # Automated benchmarking across protocols
└── Programs/
    └── Source/
        └── dark-auction.mpc    # ← Your MPC auction program goes here
```
---

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/)

### 1. Clone the repository

```bash
git clone https://github.com/ribeirocn/mpc-project
cd mpc-project
```

### 2. Start the containers

```bash
docker compose up -d --build party0 party1 party2
```

### 3. Edit input files

Each party's orders are in `inputs/party{id}.txt`. The format is:

```
# bid_price bid_qty ask_price ask_qty
# Use 0 for price/qty to indicate no order on that side
106 5 0   0    # bid: buy 5 units at up to 106
0   0 103 2    # ask: sell 2 units at no less than 103
```

After editing, regenerate the MP-SPDZ input files:

```bash
python3 scripts/generate_inputs.py --convert-only
```

---

## Running the Auction

### Compile your MPC program

```bash
docker compose exec party0 bash -c \
  'cd /mp-spdz && python3 compile.py dark-auction'
```

### Run with a specific protocol

Open one terminal per party (or use the helper scripts):

```bash
# Party 0
docker compose exec party0 bash -c \
  'cd /mp-spdz && ./mascot-party.x -N 3 -p 0 -ip Config/IPs dark-auction'

# Party 1
docker compose exec party1 bash -c \
  'cd /mp-spdz && ./mascot-party.x -N 3 -p 1 -ip Config/IPs dark-auction'

# Party 2
docker compose exec party2 bash -c \
  'cd /mp-spdz && ./mascot-party.x -N 3 -p 2 -ip Config/IPs dark-auction'
```

### Use the helper scripts

```bash
bash scripts/test_run.sh        # Quick test with a single protocol
bash scripts/run_auction.sh     # Compile and run across multiple protocols
bash scripts/benchmark.sh       # Automated benchmarking
```
