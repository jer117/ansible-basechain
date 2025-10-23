# Base Node Ansible Deployment

This Ansible playbook deploys a Base node using Docker containers. Base is a secure, low-cost, developer-friendly Ethereum L2 built on Optimism's OP Stack.

## Quick Start

1. **Configure your L1 endpoints** in `host_vars/{{ inventory_hostname }}.yml`:
   ```yaml
   base_node_l1_eth_rpc: "https://your-l1-eth-rpc-endpoint"
   base_node_l1_beacon: "https://your-l1-beacon-endpoint"
   ```

2. **Choose your network and client**:
   ```yaml
   base_node_network: "mainnet"  # or "sepolia" for testnet
   base_node_client: "geth"      # geth, reth, or nethermind
   ```

3. **Deploy the Base node**:
   ```bash
   # Deploy Base node only
   ansible-playbook -i inventory main.yml --tags base-node
   
   # Deploy with monitoring
   ansible-playbook -i inventory main.yml --tags base-node,monitoring
   ```

## Supported Clients

- **Geth** (default) - Ethereum Go client
- **Reth** - High-performance Rust client with Flashblocks support
- **Nethermind** - .NET Ethereum client

## Network Configuration

### Mainnet
- Network: `base-mainnet`
- Sequencer: `https://mainnet-sequencer.base.org`
- Default cache: 20GB

### Sepolia Testnet
- Network: `base-sepolia`
- Sequencer: `https://sepolia-sequencer.base.org`
- Default cache: 10GB

## Configuration Options

### Required Settings

Set these in your `host_vars/{{ inventory_hostname }}.yml`:

```yaml
# L1 Configuration (REQUIRED)
base_node_l1_eth_rpc: "https://your-l1-eth-rpc-endpoint"
base_node_l1_beacon: "https://your-l1-beacon-endpoint"
base_node_l1_rpc_kind: "debug_geth"  # or alchemy, quicknode, infura, etc.
```

### Optional Settings

```yaml
# Network and Client
base_node_network: "mainnet"  # mainnet or sepolia
base_node_client: "geth"      # geth, reth, or nethermind
base_node_type: "vanilla"     # vanilla or archive

# Performance Tuning (for Geth)
base_node_geth_cache: "20480"        # 20GB cache
base_node_geth_cache_database: "20"   # 4GB database cache
base_node_geth_cache_gc: "12"        # GC cache
base_node_geth_cache_snapshot: "24"  # Snapshot cache
base_node_geth_cache_trie: "44"      # Trie cache

# Advanced Features
base_node_trusted_rpc: false          # Enable trusted RPC mode
base_node_snap_sync: false           # Enable snap sync (experimental)
base_node_ethstats_enabled: false    # Enable EthStats monitoring
base_node_ethstats_url: ""          # EthStats URL
```

## Ports

The Base node exposes the following ports:

### Execution Node
- **8545**: RPC endpoint
- **8546**: WebSocket endpoint
- **7301**: Metrics endpoint
- **30303**: P2P (TCP/UDP)

### OP Node
- **7545**: RPC endpoint
- **9222**: P2P (TCP/UDP)
- **7300**: Metrics endpoint
- **6060**: pprof endpoint

## Hardware Requirements

### Minimum Requirements
- Modern Multicore CPU
- 32GB RAM (64GB Recommended)
- NVMe SSD drive
- Storage: (2 * [current chain size](https://base.org/stats) + [snapshot size](https://basechaindata.vercel.app) + 20% buffer)

### Production Hardware Specifications

#### Geth Full Node
- **Instance**: AWS i4i.12xlarge
- **Storage**: RAID 0 of all local NVMe drives
- **Filesystem**: ext4

#### Reth Archive Node
- **Instance**: AWS i7ie.6xlarge
- **Storage**: RAID 0 of all local NVMe drives
- **Filesystem**: ext4

## Usage Examples

### Deploy Mainnet Node with Geth
```bash
# Set in host_vars/{{ inventory_hostname }}.yml
base_node_network: "mainnet"
base_node_client: "geth"
base_node_l1_eth_rpc: "https://your-mainnet-l1-rpc"
base_node_l1_beacon: "https://your-mainnet-l1-beacon"

# Deploy
ansible-playbook -i inventory main.yml --tags base-node
```

### Deploy Sepolia Testnet with Reth
```bash
# Set in host_vars/{{ inventory_hostname }}.yml
base_node_network: "sepolia"
base_node_client: "reth"
base_node_l1_eth_rpc: "https://your-sepolia-l1-rpc"
base_node_l1_beacon: "https://your-sepolia-l1-beacon"

# Deploy
ansible-playbook -i inventory main.yml --tags base-node
```

### Deploy with Custom Performance Settings
```bash
# Set in host_vars/{{ inventory_hostname }}.yml
base_node_geth_cache: "40960"        # 40GB cache
base_node_geth_cache_database: "40"  # 8GB database cache
base_node_geth_cache_gc: "24"        # Larger GC cache
base_node_geth_cache_snapshot: "48" # Larger snapshot cache
base_node_geth_cache_trie: "88"     # Larger trie cache

# Deploy
ansible-playbook -i inventory main.yml --tags base-node
```

## Monitoring

The Base node includes built-in metrics endpoints:

- **Execution node metrics**: `http://localhost:7301/metrics`
- **OP node metrics**: `http://localhost:7300/metrics`

You can integrate with Prometheus and Grafana using the existing monitoring tasks:

```bash
ansible-playbook -i inventory main.yml --tags base-node,monitoring
```

## Troubleshooting

### Check Container Status
```bash
docker ps | grep base-node
```

### View Logs
```bash
# Execution node logs
docker logs base-node-execution

# OP node logs
docker logs base-node-op-node
```

### Check RPC Endpoints
```bash
# Test execution node RPC
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  http://localhost:8545

# Test OP node RPC
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  http://localhost:7545
```

### Common Issues

1. **L1 RPC endpoint issues**: Ensure your L1 endpoints are accessible and have sufficient rate limits
2. **Memory issues**: Adjust `base_node_docker_memory_limit` in `vars/main.yml`
3. **Storage issues**: Ensure sufficient disk space for the blockchain data
4. **Network issues**: Check firewall settings for the required ports

## Support

For support, please join the [Base Discord](https://discord.gg/buildonbase) and post in the `🛠｜node-operators` channel.

## Disclaimer

THE NODE SOFTWARE IS PROVIDED "AS IS" WITHOUT WARRANTY OF ANY KIND. We make no guarantees about asset protection or security. Usage is subject to applicable laws and regulations.

For more information, visit [docs.base.org](https://docs.base.org/).
