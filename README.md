# chihuahua-testnet-420

This is a testnet for chihuahua that will start on v1.1.1 and then upgrade to v2.0.0-rc-1.


## Installation Steps

### Install Prerequisites 

The following are necessary to build chihuahua from source. 

#### 1. Basic Packages
```bash:
# update the local package list and install any available upgrades 
sudo apt-get update && sudo apt upgrade -y 
# install toolchain and ensure accurate time synchronization 
sudo apt-get install make build-essential gcc git jq chrony -y
```

#### 2. Install Go
Follow the instructions [here](https://golang.org/doc/install) to install Go.

Alternatively, for Ubuntu LTS, you can do:
```bash:
wget https://golang.org/dl/go1.17.6.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.17.6.linux-amd64.tar.gz
```

Unless you want to configure in a non standard way, then set these in the `.profile` in the user's home (i.e. `~/`) folder.

```bash:
cat <<EOF >> ~/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source ~/.profile
go version
```
Output should be: `go version go1.17.5 linux/amd64`

### 3. Install Chihuahua from source

```bash:
git clone https://github.com/ChihuahuaChain/chihuahua.git
cd chihuahua
git fetch --tags
git checkout v1.1.1
make install
```
Note: there is no tag to build off of, just use master for now

### Init chain
```bash:
chihuahuad init $MONIKER_NAME --chain-id chihuahua-testnet-420
```

### Download Genesis
```bash:
wget -O ~/.chihuahua/config/genesis.json https://raw.githubusercontent.com/pomifer/chihuahua-testnet-420/main/genesis.json
```

### Add Seeds & Persistent Peers
```bash:
seeds="4936e377b4d4f17048f8961838a5035a4d21240c@chihuahua-seed-01.mercury-nodes.net:29540"
peers="b140eb36b20f3d201936c4757d5a1dcbf03a42f1@216.238.79.138:26656,19900e1d2b10be9c6672dae7abd1827c8e1aad1e@161.97.96.253:26656,c382a9a0d4c0606d785d2c7c2673a0825f7c53b2@88.99.94.120:26656,a5dfb048e4ed5c3b7d246aea317ab302426b37a1@137.184.250.180:26656,3bad0326026ca4e29c64c8d206c90a968f38edbe@128.199.165.78:26656,89b576c3eb72a4f0c66dc0899bec7c21552ea2a5@23.88.7.73:29538,38547b7b6868f93af1664d9ab0e718949b8853ec@54.184.20.240:30758,a9640eb569620d1f7be018a9e1919b0357a18b8c@38.146.3.160:26656,7e2239a0d4a0176fe4daf7a3fecd15ac663a8eb6@144.91.126.23:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.chihuahua/config/config.toml
```

### Update minimum-gas-prices in app.toml
```bash:
minimum-gas-prices = 0.025uhuahua
```

### Add/recover keys
```bash:
# To create new keypair - make sure you save the mnemonics!
chihuahuad keys add <key-name> 

# Restore existing odin wallet with mnemonic seed phrase. 
# You will be prompted to enter mnemonic seed. 
chihuahuad keys add <key-name> --recover
```

## Create a service
This will restart the process in case of process crash or server restart.

Create a `/etc/systemd/system/chihuahuad.service` file as `root` with: 
```
[Unit]
Description=Chihuahua Node
After=network-online.target

[Service]
User=<your-user>
ExecStart=/home/<your-user>/go/bin/chihuahuad start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```
Note: replace the 2 `<your-user>` with your user/path and remove the `User=` line in case of `root`
```
# Enable on server restart 
sudo systemctl enable chihuahuad

# Start the service
sudo systemctl start chihuahuad

# Show logs
sudo journalctl -u chihuahuad.service -f
```

## Instructions for post-genesis validators

### Create the validator

Note that proposal #1 agrees that all validators set commission to at
least 5%!

```bash:
chihuahuad tx staking create-validator \
  --from "<key-name>" \
  --amount "10000000uhuahua" \
  --pubkey "$(chihuahuad tendermint show-validator)" \
  --chain-id "chihuahua-testnet-420" \
  --moniker "<moniker>" \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.10 \
  --min-self-delegation 1 \
  --details "<details>" \
  --security-contact "<contact>" \
  --website "<website>" \
  --gas-prices "0.025uhuahua"
```

### Backup critical files
```bash:
priv_validator_key.json
```
