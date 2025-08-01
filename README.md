# **Complete A to Z Guide: Running Drosera on VPS Using Termius (Mobile)**

This guide covers everything from connecting to your VPS via Termius to fully setting up Drosera, including Docker, trap configuration, operator registration, and getting testnet ETH.

---

## **📱 Step 1: Install Termius (Android/iOS)**
Download Termius for mobile SSH access:
- **Android**: [Termius on Google Play](https://play.google.com/store/apps/details?id=com.server.auditor.ssh.client)
- **iOS**: [Termius on App Store](https://apps.apple.com/us/app/termius-ssh-shell-console-terminal/id549039908)

---

## **🔌 Step 2: Connect to Your VPS via Termius**
1. Open Termius → Tap **"Hosts"** → **"+"** (Add new host).
2. Fill in:
   - **Alias**: `Drosera VPS` (any name)
   - **Hostname**: Your VPS IP address
   - **Username**: `root` (or your sudo user)
   - **Authentication**: Password or SSH key
3. Tap **"Save"** → **"Connect"**.

---

## **⚙️ Step 3: Initial VPS Setup**
### **Update & Install Dependencies**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl ufw git wget jq build-essential lz4 tmux htop
```

### **Configure Firewall**
```bash
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 31313/tcp # Drosera P2P
sudo ufw allow 31314/tcp # Drosera HTTP
sudo ufw enable
```

---

## **🐳 Step 4: Install Docker**
```bash
# Remove old Docker versions
sudo apt remove docker.io docker-doc docker-compose podman-docker containerd runc -y

# Install Docker
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify Docker
sudo docker run hello-world
```

---

## **🕷️ Step 5: Set Up Drosera Trap**
### **Install Required Tools**
```bash
# Drosera CLI
curl -L https://app.drosera.io/install | bash
source ~/.bashrc
droseraup

# Foundry (Solidity)
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup

# Bun (JavaScript runtime)
curl -fsSL https://bun.sh/install | bash
source ~/.bashrc
```

### **Initialize Trap Project**
```bash
mkdir ~/my-drosera-trap && cd ~/my-drosera-trap
git config --global user.email "your_email@example.com"
git config --global user.name "your_username"
forge init -t drosera-network/trap-foundry-template
bun install
forge build
```

### **Configure Trap**
```bash
nano drosera.toml
```
Paste (replace `YOUR_OPERATOR_WALLET`):
```toml
ethereum_rpc = "https://ethereum-hoodi-rpc.publicnode.com"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"

[traps.helloworld]
path = "out/HelloWorldTrap.sol/HelloWorldTrap.json"
response_contract = "0x183D78491555cb69B68d2354F7373cc2632508C7"
response_function = "helloworld(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["YOUR_OPERATOR_WALLET_ADDRESS"]
```

### **Apply Configuration**
```bash
DROSERA_PRIVATE_KEY=your_private_key_here drosera apply
```
### **🔥 Bloom Boost**
Increase trap priority:
```bash
drosera bloomboost --trap-address YOUR_TRAP_ADDRESS --eth-amount 0.1
```
#### **💧Get Hoodi ETH (Testnet Tokens)**
1. **Add Hoodi Network to MetaMask:**
   - Chain ID: `560048`
   - RPC URL: `https://ethereum-hoodi-rpc.publicnode.com`
   - Explorer: `https://explorer.hoodi.drosera.io`

2. **Use Faucet:**
   - Visit [Hoodi Faucet](https://faucet.hoodi.drosera.io/)
   - Connect wallet → Request test ETH
  
     
---

## **🤖 Step 6: Set Up Drosera Operator (Docker)**
### **Create Docker Compose File**
```bash
mkdir ~/Drosera-Network && cd ~/Drosera-Network
nano docker-compose.yaml
```
Paste:
```yaml
version: '3.8'
services:
  drosera-operator:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-operator
    ports:
      - "31313:31313"
      - "31314:31314"
    environment:
      - DRO__DB_FILE_PATH=/data/drosera.db
      - DRO__DROSERA_ADDRESS=0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
      - DRO__LISTEN_ADDRESS=0.0.0.0
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__BACKUP_RPC_URL=https://rpc.hoodi.ethpandaops.io
      - DRO__ETH__PRIVATE_KEY=${ETH_PRIVATE_KEY}
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${VPS_IP}
      - DRO__SERVER__PORT=31314
      - RUST_LOG=info,drosera_operator=debug
    volumes:
      - drosera_data:/data
    restart: always
volumes:
  drosera_data:
```

### **Create .env File**
```bash
nano .env
```
Add:
```env
ETH_PRIVATE_KEY=your_private_key_here
VPS_IP=your_vps_public_ip
```

### **Start Operator**
```bash
docker compose up -d
docker compose logs -f  # View logs
```

---
