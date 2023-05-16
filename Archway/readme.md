<p align="center">
  <img height="100" height="auto" src="https://github.com/sychonix/testnet/blob/main/Archway/archway.jpg">
</p>

# Archway | Chain ID : constantine-3 | Version v0.5.2

### Install dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

### Install Go
```
ver="1.20.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

### Build Binary
```
cd $HOME
git clone https://github.com/archway-network/archway.git
cd archway
git checkout v0.5.1
make install
```

### Initialize the node

```
archwayd init $MONIKER --chain-id constantine-3
archwayd config chain-id constantine-3
archwayd config keyring-backend test
```
```
archwayd config node tcp://localhost:26657
```

### Custom Port ( Optional)
```
PORT=34
archwayd config node tcp://localhost:${PORT}657
```
```
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.archway/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.archway/config/app.toml
```

### Genesis
```
wget -O $HOME/.archway/config/genesis.json https://raw.githubusercontent.com/archway-network/networks/main/constantine-3/genesis.json
```

### Addrbook
```

```

### Seed & Peer
```
SEEDS="3c5bc400c786d8e57ae2b85639273d1aec79829a@34.31.130.235:26656"
sed -i -e "s|^seeds *=.*|seeds = \"$SEEDS\"|" $HOME/.archway/config/config.toml
peers="b2d39b9b7991c0a9a7678994f5afe04a1b3df923@34.122.164.239:26656,930bf53d3858340d52bb7e541617740d91477ff0@335.239.130.141:26656,434015482b70fff4e6bc96299dee7b84aca01343@35.223.36.227:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.archway/config/config.toml
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uconst\"|" $HOME/.archway/config/app.toml
```

### Prunning
```
sed -i \
-e 's|^pruning *=.*|pruning = "custom"|' \
-e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
-e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
-e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
$HOME/.archway/config/app.toml
```
### Indexer Off (optional)
```
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.archway/config/config.toml
```
### Service
```
sudo tee /etc/systemd/system/archwayd.service << EOF
[Unit]
Description=archway-testnet
After=network-online.target
#
[Service]
User=$USER
ExecStart=$(which archwayd) start
RestartSec=3
Restart=on-failure
LimitNOFILE=65535
#
[Install]
WantedBy=multi-user.target
EOF
```

### Start
```
sudo systemctl daemon-reload
sudo systemctl enable archwayd
sudo systemctl restart archwayd
sudo journalctl -u archwayd -f -o cat
```

### Sync Info
```
archwayd status 2>&1 | jq .SyncInfo
```

### Check Log Node
```
sudo journalctl -u archwayd -f -o cat
```

### Wallet
```
archwayd keys add wallet
```
### Recover
```
archwayd keys add wallet --recover
```

### Balances
```
archwayd  q bank balances <address>
```

### Validator
```
archwayd tx staking create-validator \
--amount 999500uconst \
--pubkey $(archwayd tendermint show-validator) \
--moniker "your_name" \
--identity "your_keybase" \
--details "your_details" \
--website "your_website" \
--chain-id constantine-3 \
--commission-rate 0.1 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
-y
```

### Unjail
```
archwayd tx slashing unjail --from wallet --chain-id constantine-3 --gas-adjustment=1.4 --gas auto -y
```

### Staking
```
archwayd tx staking delegate <TO_VALOPER_ADDRESS> 1000000000uconst --from wallet --chain-id constantine-3 --gas-prices 0.1uconst --gas-adjustment 1.4 --gas auto -y 
```

### Stop
```
sudo systemctl stop archwayd
```

### Restart
```
sudo systemctl restart archwayd
```

### Delete
```
sudo systemctl stop archwayd
sudo systemctl disable archwayd
sudo rm /etc/systemd/system/archwayd.service
sudo systemctl daemon-reload
rm -f $(which archwayd)
rm -rf .archway
rm -rf archway
```




