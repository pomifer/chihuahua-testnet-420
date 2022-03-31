# chihuahua-testnet-420

This is a testnet for chihuahua that will start on v1.1.1 and then upgrade to v2.0.0-rc-2.


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
peers="26ef762af837984e37c2bee3ee355203cfe7f248@164.92.147.131:26656"
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
