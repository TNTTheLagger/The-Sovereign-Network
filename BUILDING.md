# Building The Sovereign Network

## Prerequisites

### Ubuntu / Debian

```bash
sudo apt update && sudo apt install -y \
    build-essential \
    clang \
    libclang-dev \
    libsnappy-dev \
    liblz4-dev \
    libzstd-dev \
    zlib1g-dev \
    libbz2-dev \
    cmake \
    pkg-config \
    libssl-dev
```

If RocksDB compilation still fails, install the system package:

```bash
sudo apt install -y librocksdb-dev
export ROCKSDB_LIB_DIR=/usr/lib
```

### Fedora / RHEL

```bash
sudo dnf install -y \
    gcc-c++ \
    clang \
    clang-devel \
    snappy-devel \
    lz4-devel \
    libzstd-devel \
    zlib-devel \
    bzip2-devel \
    cmake \
    openssl-devel
```

### macOS

```bash
brew install cmake snappy lz4 zstd rocksdb openssl
```

## Building

```bash
cargo build --release
```

## Running

**Use `--testnet` for local development and testing.** Production mode requires a configured validator identity and network connectivity that will likely fail on first run.

```bash
# Recommended for development/testing
./target/release/zhtp --testnet

# With verbose logging
RUST_LOG=debug ./target/release/zhtp --testnet
```

## Platform Notes

### macOS: Bluetooth Discovery

Bluetooth mesh discovery requires special permissions on macOS. If you encounter permission errors or want to disable Bluetooth scanning:

```bash
# Run without Bluetooth discovery
./target/release/zhtp --testnet --disable-bluetooth
```

Alternatively, grant Bluetooth permissions to Terminal/iTerm in:
**System Preferences → Privacy & Security → Bluetooth**

### Linux: Bluetooth Discovery

Bluetooth requires `CAP_NET_ADMIN` capability or running with elevated privileges:

```bash
# Option 1: Run with sudo (not recommended for production)
sudo ./target/release/zhtp --testnet

# Option 2: Grant capability (recommended)
sudo setcap 'cap_net_admin,cap_net_raw+eip' ./target/release/zhtp
./target/release/zhtp --testnet

# Option 3: Disable Bluetooth
./target/release/zhtp --testnet --disable-bluetooth
```

## Troubleshooting

### RocksDB compilation fails on Ubuntu

Install all dependencies listed above. If errors persist:

```bash
# Use system RocksDB
sudo apt install -y librocksdb-dev
export ROCKSDB_LIB_DIR=/usr/lib
cargo clean
cargo build --release
```

### Missing `libclang`

```bash
# Ubuntu/Debian
sudo apt install libclang-dev

# Fedora
sudo dnf install clang-devel

# macOS
brew install llvm
```

### OpenSSL not found

```bash
# Ubuntu/Debian
sudo apt install libssl-dev pkg-config

# Fedora
sudo dnf install openssl-devel

# macOS
brew install openssl
export OPENSSL_DIR=$(brew --prefix openssl)
```

## Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 4 GB | 8 GB |
| Storage | 20 GB SSD | 100 GB NVMe |
| Network | 10 Mbps | 100 Mbps |

- **SSD required** - RocksDB performs poorly on HDD
- **Testnet**: 2 GB RAM may work with `--testnet`
- **Storage grows** with DHT data - plan for expansion

## Security (Home Node Operators)

Running a node on a home computer requires basic security hygiene.

### Network Security

**Firewall - only expose required ports:**

```bash
# Ubuntu/Debian
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 33445/udp  # ZHTP mesh
sudo ufw allow 33446/tcp  # QUIC transport
sudo ufw enable
```

**Bind admin interfaces to localhost:**

```bash
./zhtp --testnet --rpc-bind 127.0.0.1
```

### Run as Dedicated User

```bash
sudo useradd -r -s /bin/false zhtp-node
sudo mkdir -p /opt/zhtp
sudo chown -R zhtp-node:zhtp-node /opt/zhtp
sudo -u zhtp-node ./zhtp --testnet
```

### System Hardening

```bash
# Keep system updated
sudo apt update && sudo apt upgrade -y

# Install fail2ban for SSH protection
sudo apt install fail2ban -y
sudo systemctl enable fail2ban

# Disable SSH password auth (use keys)
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### What NOT to Do

- Don't run as root
- Don't store personal crypto wallets on the node machine
- Don't expose SSH (port 22) without fail2ban
- Don't use the node for daily browsing/email
- Don't share your node's identity keys
- Don't ignore system updates

### Key Management

- Backup validator keys offline (USB drive, paper)
- Never store keys on machines with personal data
- Use separate physical device if possible (NUC, Raspberry Pi)

### Monitoring

```bash
# Check connections
sudo netstat -tlnp | grep zhtp

# Monitor logs
journalctl -u zhtp -f
```
