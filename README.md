# Optimism Layer 2 Sandbox - Complete Configuration Guide

[![Optimism](https://img.shields.io/badge/Optimism-L2-red.svg)](https://optimism.io/)
[![Network](https://img.shields.io/badge/Network-Sandbox-yellow.svg)]()
[![Chain ID](https://img.shields.io/badge/L2%20Chain%20ID-20254-blue.svg)]()

A comprehensive guide to understanding and deploying an Optimism Layer 2 network configuration over a self-hosted Proof of Stake Ethereum development network.

## 📑 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Network Configuration](#network-configuration)
- [Configuration Files Deep Dive](#configuration-files-deep-dive)
- [Component Configurations](#component-configurations)
- [Smart Contract Addresses](#smart-contract-addresses)
- [Network Topology](#network-topology)
- [Deployment Sequence](#deployment-sequence)
- [Monitoring & Operations](#monitoring--operations)
- [Troubleshooting](#troubleshooting)

## 🏗️ Architecture Overview

This Optimism Layer 2 setup consists of four main components working together:

```
┌─────────────────────────────────────────────────────────────────┐
│                         L1 Ethereum (Devnet)                   │
│                    Chain ID: 20253                             │
│         ┌─────────────────┐    ┌─────────────────┐             │
│         │  Execution EL   │    │   Beacon CL     │             │
│         │  Port: 8545     │    │   Port: 3500    │             │
│         │ IP: 3.108.65.88 │    │ IP: 3.108.65.88 │             │
│         └─────────────────┘    └─────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼ (Deposits & Batch Submissions)
┌─────────────────────────────────────────────────────────────────┐
│                    Optimism Layer 2 Stack                      │
│                       Chain ID: 20254                          │
│                                                                 │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│  │  op-geth    │◄──┤  op-node    │──►│ op-proposer │          │
│  │ (Execution) │   │ (Rollup)    │   │ (Proposals) │          │
│  │ Port: 9545  │   │ Port: 8547  │   │ Port: 8560  │          │
│  └─────────────┘   └─────────────┘   └─────────────┘          │
│         ▲                                                      │
│         │                                                      │
│  ┌─────────────┐                                               │
│  │ op-batcher  │                                               │
│  │ (Batching)  │                                               │
│  │ Port: 8548  │                                               │
│  └─────────────┘                                               │
└─────────────────────────────────────────────────────────────────┘
```

## 🌐 Network Configuration

### Chain Identifiers
| Parameter | Value | Description |
|-----------|-------|-------------|
| **L1 Chain ID** | `20253` | Custom Ethereum devnet identifier |
| **L2 Chain ID** | `20254` | Optimism Layer 2 network identifier |
| **Network ID** | `20254` | P2P network identifier (matches L2 chain ID) |

### Core Network Parameters
| Parameter | Value | Unit | Description |
|-----------|-------|------|-------------|
| **Block Time** | `2` | seconds | Target L2 block production interval |
| **Max Sequencer Drift** | `600` | seconds | Maximum time sequencer can drift from L1 |
| **Sequencer Window** | `3600` | seconds | Time window for sequencer batch submission |
| **Channel Timeout** | `300` | seconds | Data channel timeout period |
| **Gas Limit** | `60,000,000` | gas | Maximum gas per L2 block |

### EIP-1559 Fee Configuration
| Parameter | Value | Description |
|-----------|-------|-------------|
| **Elasticity** | `6` | Block size elasticity multiplier |
| **Denominator** | `50` | Base fee adjustment denominator |
| **Canyon Denominator** | `250` | Post-Canyon upgrade denominator |

## 📁 Configuration Files Deep Dive

### 1. `intent.toml` - Deployment Intent

Defines the high-level configuration for deploying the Optimism stack:

```toml
configType = "custom"                    # Custom deployment configuration
l1ChainID = 20253                       # Target L1 chain
fundDevAccounts = true                  # Fund development accounts
l1ContractsLocator = "tag://op-contracts/v4.0.0"  # L1 contract version
l2ContractsLocator = "tag://op-contracts/v4.0.0"  # L2 contract version
```

**Key Roles Defined:**
- **SuperchainProxyAdminOwner**: `0xB67b4ac6Dcef5031AcA1D59E980e16753b4c97F8`
- **SuperchainGuardian**: `0xB67b4ac6Dcef5031AcA1D59E980e16753b4c97F8`
- **ProtocolVersionsOwner**: `0xB67b4ac6Dcef5031AcA1D59E980e16753b4c97F8`

### 2. `rollup.json` - Rollup Protocol Configuration

Core rollup protocol parameters that define network behavior:

```json
{
  "block_time": 2,                              // 2-second block intervals
  "max_sequencer_drift": 600,                   // 10-minute max drift
  "seq_window_size": 3600,                      // 1-hour sequencer window
  "channel_timeout": 300,                       // 5-minute channel timeout
  "l1_chain_id": 20253,                        // L1 network
  "l2_chain_id": 20254,                        // L2 network
}
```

**Critical Contract Addresses:**
- **Batch Inbox**: `0x00287e94920f8343e1ece955bbe2bbed4175fef5`
- **Deposit Contract**: `0x7f95bf377ee75b1d38cbd1bede64128bde3614d3`
- **System Config**: `0x30169b84ed6bb6afec4f12330832c8fb781ad290`

### 3. `genesis.json` - L2 Genesis Block

Defines the initial state of the Layer 2 network:

```json
{
  "config": {
    "chainId": 20254,                           // L2 chain identifier
    "optimism": {
      "eip1559Elasticity": 6,                   // Fee elasticity
      "eip1559Denominator": 50,                 // Base fee denominator
      "eip1559DenominatorCanyon": 250           // Post-Canyon denominator
    }
  },
  "gasLimit": "0x3938700",                      // 60M gas limit (hex)
  "coinbase": "0x4200000000000000000000000000000000000011"  // Fee recipient
}
```

### 4. `state.json` - Deployment State

Contains deployed contract addresses and system state:

**Operator Roles:**
- **L1 Proxy Admin Owner**: `0xb67b4ac6dcef5031aca1d59e980e16753b4c97f8`
- **L2 Proxy Admin Owner**: `0xb67b4ac6dcef5031aca1d59e980e16753b4c97f8`
- **System Config Owner**: `0xb67b4ac6dcef5031aca1d59e980e16753b4c97f8`
- **Unsafe Block Signer**: `0xd602d2aea60259815465ece963b6013aa94edf56`
- **Batcher**: `0x13fcfe478bcc9b6562f85ec2ab6f79c60ee6b478`
- **Proposer**: `0x4eaa6339f2df32858680b900c7bdd029cf87001b`
- **Challenger**: `0xb67b4ac6dcef5031aca1d59e980e16753b4c97f8`

## ⚙️ Component Configurations

### 🟡 op-geth (Execution Layer)

**Purpose**: Executes transactions and maintains the L2 state.

**Key Configuration Parameters:**

| Parameter | Value | Description |
|-----------|-------|-------------|
| `--datadir` | `./op-geth-data` | Data directory for blockchain data |
| `--http.port` | `9545` | HTTP RPC endpoint port |
| `--ws.port` | `9546` | WebSocket RPC endpoint port |
| `--authrpc.port` | `9551` | Authenticated RPC port (Engine API) |
| `--port` | `9303` | P2P networking port |
| `--networkid` | `20254` | Network identifier |
| `--syncmode` | `full` | Full synchronization mode |
| `--gcmode` | `archive` | Archive mode (keeps all state) |
| `--maxpeers` | `0` | No peer connections (isolated) |

**Security Features:**
- JWT authentication for Engine API (`--authrpc.jwtsecret`)
- Disabled transaction pool gossip (`--rollup.disabletxpoolgossip`)
- No peer discovery (`--nodiscover`)

```bash
./geth \
  --datadir=./op-geth-data \
  --http --http.addr=0.0.0.0 --http.port=9545 \
  --http.vhosts="*" --http.corsdomain="*" \
  --http.api=eth,net,web3,debug,txpool,admin,personal \
  --ws --ws.addr=0.0.0.0 --ws.port=9546 \
  --ws.origins="*" --ws.api=eth,net,web3,debug,txpool,admin \
  --authrpc.addr=0.0.0.0 --authrpc.port=9551 \
  --authrpc.vhosts="*" --authrpc.jwtsecret=./jwt.txt \
  --port=9303 --networkid="20254" \
  --syncmode=full --gcmode=archive \
  --nodiscover --maxpeers=0 \
  --rollup.disabletxpoolgossip=true
```

### 🔵 op-node (Rollup Node)

**Purpose**: Manages L2 block derivation from L1 and coordinates the rollup protocol.

**Key Configuration Parameters:**

| Parameter | Value | Description |
|-----------|-------|-------------|
| `--l1` | `http://3.108.65.88:8545` | L1 execution layer endpoint |
| `--l1.beacon` | `http://3.108.65.88:3500` | L1 beacon chain endpoint |
| `--l2` | `http://3.110.183.232:9551` | L2 execution layer endpoint |
| `--rpc.port` | `8547` | RPC server port |
| `--sequencer.enabled` | `true` | Enable sequencer mode |
| `--sequencer.max-safe-lag` | `3600` | Max safe lag (1 hour) |
| `--verifier.l1-confs` | `4` | L1 confirmation depth |

**Sequencer Configuration:**
- **Enabled**: Active sequencer mode for transaction ordering
- **Max Safe Lag**: 1-hour maximum behind L1 before stopping
- **L1 Confirmations**: Waits for 4 L1 confirmations

```bash
./op-node \
  --l1=http://3.108.65.88:8545 \
  --l1.beacon=http://3.108.65.88:3500 \
  --l2=http://3.110.183.232:9551 \
  --l2.jwt-secret=$JWT_SECRET \
  --rollup.config=./rollup.json \
  --sequencer.enabled=true \
  --sequencer.stopped=false \
  --sequencer.max-safe-lag=3600 \
  --verifier.l1-confs=4 \
  --p2p.disable \
  --rpc.addr=0.0.0.0 --rpc.port=8547 \
  --rpc.enable-admin
```

### 🟢 op-proposer (Output Proposals)

**Purpose**: Submits L2 output roots to L1 for dispute resolution and withdrawals.

**Key Configuration Parameters:**

| Parameter | Value | Description |
|-----------|-------|-------------|
| `--rollup-rpc` | `http://3.110.185.186:8547` | op-node RPC endpoint |
| `--l1-eth-rpc` | `http://3.108.65.88:8545` | L1 Ethereum RPC |
| `--proposal-interval` | `600s` | Submit proposals every 10 minutes |
| `--game-factory-address` | `0x33cb848948e3b731f2106d9bc64e03ae1c8be445` | Fault proof factory |
| `--game-type` | `1` | Fault proof game type |
| `--private-key` | `0x239a5b...` | Proposer private key |

**Fault Proof System:**
- **Game Factory**: Manages dispute games for invalid proposals
- **Game Type 1**: Standard fault proof game
- **10-minute intervals**: Regular output proposal submission

```bash
./op-proposer \
  --poll-interval=20s \
  --rpc.port=8560 --rpc.enable-admin \
  --rollup-rpc=http://3.110.185.186:8547 \
  --l1-eth-rpc=http://3.108.65.88:8545 \
  --private-key=0x239a5b34386a7a59f7dee33b117fb955978471ff8bb1a3fc86417155584cd05f \
  --game-factory-address=0x33cb848948e3b731f2106d9bc64e03ae1c8be445 \
  --game-type=1 \
  --proposal-interval=600s \
  --num-confirmations=1 \
  --wait-node-sync=true
```

### 🟠 op-batcher (Transaction Batcher)

**Purpose**: Batches L2 transactions and submits them to L1 for data availability.

**Key Configuration Parameters:**

| Parameter | Value | Description |
|-----------|-------|-------------|
| `--l2-eth-rpc` | `http://3.110.183.232:9545` | L2 execution RPC |
| `--rollup-rpc` | `http://3.110.185.186:8547` | op-node RPC |
| `--poll-interval` | `1s` | Check for new batches every second |
| `--max-channel-duration` | `25` | Maximum channel duration |
| `--data-availability-type` | `calldata` | Use calldata for DA |
| `--batch-type` | `1` | Span batch type |
| `--private-key` | `0xde7a7c...` | Batcher private key |

**Batching Strategy:**
- **Real-time**: 1-second polling for immediate batching
- **Calldata DA**: Uses L1 calldata for data availability
- **Span Batches**: Efficient batch compression (type 1)
- **No Throttling**: Maximum throughput configuration

```bash
./op-batcher \
  --l2-eth-rpc=http://3.110.183.232:9545 \
  --rollup-rpc=http://3.110.185.186:8547 \
  --poll-interval=1s \
  --sub-safety-margin=6 \
  --max-channel-duration=25 \
  --l1-eth-rpc=http://3.108.65.88:8545 \
  --private-key=0xde7a7c43cf2901ef41de757659396fb94fb2362192fff3c48bf59d701bffdc59 \
  --batch-type=1 \
  --data-availability-type=calldata \
  --throttle-threshold=0 \
  --rpc.addr=0.0.0.0 --rpc.port=8548
```

## 🏢 Smart Contract Addresses

### System Contracts

| Contract | Address | Purpose |
|----------|---------|---------|
| **Batch Inbox** | `0x00287e94920f8343e1ece955bbe2bbed4175fef5` | Receives transaction batches from batcher |
| **Deposit Contract** | `0x7f95bf377ee75b1d38cbd1bede64128bde3614d3` | Handles L1→L2 deposits |
| **System Config** | `0x30169b84ed6bb6afec4f12330832c8fb781ad290` | System-wide configuration |
| **Protocol Versions** | `0x8048c7ea3a0ef76e7b3316e53a88db4303302c35` | Protocol version management |
| **Game Factory** | `0x33cb848948e3b731f2106d9bc64e03ae1c8be445` | Fault proof dispute games |

### Predeploys (L2 System Contracts)

| Contract | Address | Purpose |
|----------|---------|---------|
| **Sequencer Fee Vault** | `0x4200000000000000000000000000000000000011` | Collects sequencer fees |
| **Base Fee Vault** | `0x420000000000000000000000000000000000001A` | Collects base fees |
| **L1 Fee Vault** | `0x420000000000000000000000000000000000001B` | Collects L1 data fees |

## 🌍 Network Topology

### Server Distribution

| Server IP | Components | Ports | Role |
|-----------|------------|-------|------|
| **3.108.65.88** | L1 Ethereum (EL + CL) | 8545, 3500 | L1 Infrastructure |
| **3.110.183.232** | op-geth | 9545, 9546, 9551 | L2 Execution |
| **3.110.185.186** | op-node | 8547 | L2 Consensus |
| **13.204.80.36** | op-proposer | 8560 | L2 Operators |
| **13.233.33.156** | op-batcher | 8548 | L2 Operators |

### Port Mapping

| Port | Service | Protocol | Purpose |
|------|---------|----------|---------|
| **8545** | L1 Ethereum | HTTP RPC | L1 execution layer API |
| **3500** | L1 Beacon | HTTP | L1 consensus layer API |
| **9545** | op-geth | HTTP RPC | L2 execution layer API |
| **9546** | op-geth | WebSocket | L2 real-time API |
| **9551** | op-geth | Auth RPC | Engine API (JWT protected) |
| **8547** | op-node | HTTP RPC | Rollup node API |
| **8560** | op-proposer | HTTP RPC | Proposer management API |
| **8548** | op-batcher | HTTP RPC | Batcher management API |

## 🚀 Deployment Sequence

### Prerequisites
1. **L1 Network**: Ensure L1 Ethereum devnet is running
2. **JWT Secret**: Generate shared JWT secret for authentication
3. **Private Keys**: Fund proposer and batcher accounts with L1 ETH
4. **Binaries**: Build or download op-geth, op-node, op-proposer, op-batcher

### Startup Order

**⚠️ CRITICAL: Components must start in this exact order:**

1. **L1 Ethereum Network** (if not running)
   ```bash
   # Ensure L1 EL and CL are operational
   curl -s http://3.108.65.88:8545 -X POST -H "Content-Type: application/json" \
     --data '{"jsonrpc":"2.0","method":"eth_blockNumber","id":1}'
   ```

2. **op-geth** (L2 Execution Layer)
   ```bash
   # Must start first to provide execution environment
   ./geth [configuration flags]
   ```

3. **op-node** (L2 Rollup Node)
   ```bash
   # Starts after op-geth is ready
   ./op-node [configuration flags]
   ```

4. **op-batcher** (Transaction Batcher)
   ```bash
   # Can start after op-node is syncing
   ./op-batcher [configuration flags]
   ```

5. **op-proposer** (Output Proposer)
   ```bash
   # Starts last, after network is stable
   ./op-proposer [configuration flags]
   ```

### Health Verification

After each component starts, verify it's healthy:

```bash
# op-geth health
curl -X POST http://localhost:9545 -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","id":1}'

# op-node health  
curl -X POST http://localhost:8547 -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"optimism_syncStatus","id":1}'

# op-batcher health
curl http://localhost:8548/health

# op-proposer health
curl http://localhost:8560/health
```

## 📊 Monitoring & Operations

### Key Metrics to Monitor

| Metric | Component | Expected Value | Alert Threshold |
|--------|-----------|----------------|-----------------|
| **Block Height** | op-geth | Increasing ~30/min | Stalled >2 min |
| **Sync Status** | op-node | `{"sync_status": "synced"}` | Not synced >5 min |
| **Batch Submissions** | op-batcher | Every 1-5 minutes | No batches >10 min |
| **Proposals** | op-proposer | Every 10 minutes | No proposals >15 min |
| **L1 Confirmations** | op-node | 4 confirmations | >10 confirmations |

### Log Analysis

**op-geth Logs:**
```bash
# Check for successful block production
tail -f op-geth-data/logs/geth.log | grep "Imported new chain segment"

# Monitor for sync issues
tail -f op-geth-data/logs/geth.log | grep -E "(ERROR|WARN)"
```

**op-node Logs:**
```bash
# Check rollup synchronization
./op-node [flags] 2>&1 | grep "Sync progress"

# Monitor L1 connectivity
./op-node [flags] 2>&1 | grep -E "(L1|beacon)"
```

### Performance Benchmarks

| Component | CPU Usage | Memory Usage | Disk I/O | Network |
|-----------|-----------|--------------|----------|---------|
| **op-geth** | 2-4 cores | 4-8 GB | High (archive) | Moderate |
| **op-node** | 1-2 cores | 1-2 GB | Low | High |
| **op-proposer** | <1 core | <500 MB | Low | Low |
| **op-batcher** | <1 core | <500 MB | Low | Moderate |

## 🔧 Troubleshooting

### Common Issues & Solutions

| Issue | Symptoms | Root Cause | Solution |
|-------|----------|------------|----------|
| **JWT Auth Failure** | `connection refused` on 9551 | Mismatched JWT secrets | Ensure same jwt.txt across components |
| **Sync Stalled** | op-node not advancing | L1 connectivity issues | Check L1 RPC endpoints |
| **Batch Failures** | No L1 batch transactions | Insufficient L1 ETH | Fund batcher account |
| **Proposal Failures** | No output proposals | Insufficient L1 ETH or wrong factory | Fund proposer, verify contract address |
| **High Memory Usage** | OOM errors | Archive mode + traffic | Increase RAM or use pruning |

### Debug Commands

```bash
# Check op-geth sync status
curl -X POST http://localhost:9545 -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","id":1}'

# Check op-node rollup status
curl -X POST http://localhost:8547 -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"optimism_rollupConfig","id":1}'

# Check L1 balance of batcher
curl -X POST http://3.108.65.88:8545 -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x13FcfE478bCc9b6562f85eC2Ab6F79C60EE6B478","latest"],"id":1}'

# Check L1 balance of proposer  
curl -X POST http://3.108.65.88:8545 -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x4EaA6339f2df32858680B900c7BdD029Cf87001b","latest"],"id":1}'
```

### Performance Optimization

1. **op-geth Optimization:**
   - Use SSD storage for better I/O
   - Increase cache size: `--cache=4096`
   - Consider pruning: `--gcmode=full` instead of `archive`

2. **op-node Optimization:**
   - Increase L1 RPC timeout: `--l1.rpc-max-batch-size=100`
   - Tune sequencer lag: `--sequencer.max-safe-lag=1800`

3. **Network Optimization:**
   - Use dedicated network connections
   - Monitor bandwidth usage
   - Consider CDN for RPC endpoints

---

## 🔒 Security Notice

**⚠️ DEVELOPMENT ENVIRONMENT ONLY**

This configuration is designed for development and testing purposes. The following security considerations apply:

- **Private keys are exposed** and should not be used in production
- **All services bind to 0.0.0.0** allowing external access
- **No firewall restrictions** are configured
- **Minimal authentication** is implemented

For production deployment, implement proper security measures including key management, network restrictions, and monitoring.

---

**📚 Additional Resources:**
- [Optimism Documentation](https://docs.optimism.io/)
- [OP Stack Architecture](https://stack.optimism.io/)
- [Fault Proofs](https://docs.optimism.io/stack/protocol/fault-proofs/overview)
- [Optimism GitHub](https://github.com/ethereum-optimism/optimism)
