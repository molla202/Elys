## ÖDÜLSÜZDÜR
## gereksinimler 4cpu 8ram
## exp. ( https://explorer.stavr.tech/elys-testnet  )
## Discord https://discord.gg/HPz8J9eC
### güncelleme geldi snapsız hata verir
# Elys
![1500x500](https://user-images.githubusercontent.com/91562185/231207195-fff4a84b-36d3-4af5-85dd-1b9675417730.jpg)

## Güncelleme ve kütüphane kurulumunu yapıyoruz.
```
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
```
# install dependencies, if needed
sudo apt update

sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```

## GO 
```
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl https://dl.google.com/go/go1.19.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version
```
## Ayarlama yapılacak yerler var not defterine yapıstırıp yapın sona komple 

```
# ayarlamaları yapalım cüzdan adınızı falan yazın validator adınızı yazın aynı zamanda port yazıyor hangisini istiyorsanız onu yazın suan 38
echo "export WALLET="cüzdan-adınız"" >> $HOME/.bash_profile
echo "export MONIKER="validator-adınız"" >> $HOME/.bash_profile
echo "export ELYS_CHAIN_ID="elystestnet-1"" >> $HOME/.bash_profile
echo "export ELYS_PORT="38"" >> $HOME/.bash_profile
source $HOME/.bash_profile

go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest

# binary indiriyoruz
cd $HOME
rm -rf elys
git clone https://github.com/elys-network/elys.git
cd elys
git checkout v0.9.0
make install

# ayarlamaları yapalım
elysd config node tcp://localhost:${ELYS_PORT}657
elysd config keyring-backend os
elysd config chain-id elystestnet-1
elysd init validator-adınız --chain-id elystestnet-1

# genesis ve addrbook indiriyoruz
wget -O $HOME/.elys/config/genesis.json https://testnet-files.itrocket.net/elys/genesis.json
wget -O $HOME/.elys/config/addrbook.json https://testnet-files.itrocket.net/elys/addrbook.json

# seeds ve peers indiriyoruz
SEEDS="ae7191b2b922c6a59456588c3a262df518b0d130@elys-testnet-seed.itrocket.net:38656"
PEERS="0977dd5475e303c99b66eaacab53c8cc28e49b05@elys-testnet-peer.itrocket.net:38656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.elys/config/config.toml

# app.toml da port değiştiriyoruz çakışmasınlar :D
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${ELYS_PORT}317\"%;
s%^address = \":8080\"%address = \":${ELYS_PORT}080\"%;
s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${ELYS_PORT}090\"%; 
s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${ELYS_PORT}091\"%; 
s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${ELYS_PORT}545\"%; 
s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${ELYS_PORT}546\"%" $HOME/.elys/config/app.toml

# config.toml port ayarı
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ELYS_PORT}658\"%; 
s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${ELYS_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ELYS_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ELYS_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ELYS_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ELYS_PORT}660\"%" $HOME/.elys/config/config.toml

# pruning yapıyore
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.elys/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.elys/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.elys/config/app.toml

# gas ayarı ve index ayarı
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0uelys"/g' $HOME/.elys/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.elys/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.elys/config/config.toml

mkdir -p ~/.elys/cosmovisor/genesis/bin $ mkdir -p ~/.elys/cosmovisor/upgrades

cp ~/go/bin/elysd ~/.elys/cosmovisor/genesis/bin/

# servis dosyası oluşturuyore
sudo tee /etc/systemd/system/elysd.service > /dev/null <<EOF
[Unit] 
Description=Elys Network node 
After=network.target

[Service] 
Type=simple 
Restart=on-failure 
RestartSec=5 
User=elys 
ExecStart=$HOME/.elys/cosmovisor/genesis/bin/ run start
LimitNOFILE=65535
Environment="DAEMON_NAME=elysd"
Environment="DAEMON_HOME=$HOME/.elys"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install] 
WantedBy=multi-user.target
```
```
# snap çakalım hemen olsun
elysd tendermint unsafe-reset-all --home $HOME/.elys
curl https://testnet-files.itrocket.net/elys/snap_elys.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.elys
```
```
# servisi başlatıp loglara bakıyoruz
sudo systemctl daemon-reload
sudo systemctl enable elysd
sudo systemctl restart elysd && sudo journalctl -u elysd -f
```

```
# validator oluşturma
elysd tx staking create-validator \
  --amount 1000000uelys \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2"   --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(elysd tendermint show-validator) \
  --moniker $MONIKER \
  --chain-id elystestnet-1 \
  --gas auto --gas-adjustment 1.5
  ```
# Cüzdan oluşturmak
```
elysd keys add $WALLET
```
# Cüzdan import
```
elysd keys add $WALLET --recover
```
# Silmek için
```
sudo systemctl stop elysd
sudo systemctl disable elysd
sudo rm -rf /etc/systemd/system/elysd.service
sudo rm $(which elysd)
sudo rm -rf $HOME/.elys
sed -i "/ELYS_/d" $HOME/.bash_profile
```


