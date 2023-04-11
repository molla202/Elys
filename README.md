## ÖDÜLSÜZDÜR
## gereksinimler 4cpu 8ram
## exp. ( https://explorer.stavr.tech/elys-testnet  )
## Discord https://discord.gg/HPz8J9eC
# Elys
![1500x500](https://user-images.githubusercontent.com/91562185/231207195-fff4a84b-36d3-4af5-85dd-1b9675417730.jpg)

## Güncelleme ve kütüphane kurulumunu yapıyoruz.
```
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## Go kurulumu
```
ver="1.19.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

## Dosyaları çekiyoruz
```
git clone https://github.com/elys-network/elys
cd elys
git checkout v0.2.3
make install
```
## Düzenliyoruz(düzenlenmesi  gereken yeri düzenleyin)
```
elysd init HA-burayı-silip-kendi-adınızı-yazın-hemşerim --chain-id elystestnet-1
elysd config chain-id elystestnet-1
```
## Cüzdan oluşturuyoruz(yada --recover kodu ile import ediyoruz)
```
elysd keys add cüzdan-adı
```
```
elysd keys add cüzdan-adı --recover
```
## Adress book ve Genesis indiriyoruz efem
```
curl https://anode.team/Elys/test/genesis.json > ~/.elys/config/genesis.json
curl https://anode.team/Elys/test/addrbook.json > ~/.elys/config/addrbook.json
```

## Peers, seed ekleyelim
```
SEEDS="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:22056"
PEERS="d9f2e28e398d42fe7ca8ed322ee168b3e867bc95@65.108.199.222:34656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.elys/config/config.toml
```
## Gas ayarı çekelim
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uelys\"/" $HOME/.elys/config/app.toml
```
## Servis dosyası olusturalım
```
sudo tee /etc/systemd/system/elysd.service > /dev/null <<EOF
[Unit]
Description=Elys Network
After=network-online.target

[Service]
User=$USER
ExecStart=$(which elysd) start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

## Ve ekşın :D
```
sudo systemctl daemon-reload && sudo systemctl enable elysd
sudo systemctl restart elysd && journalctl -fu elysd -o cat
```
## Validator olusturma kodlarımız
```
elysd tx staking create-validator \
  --amount=1000000uelys \
  --pubkey=$(elysd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="elystestnet-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --fees="0uelys" \
  --from=<wallet_name>
```
## Lazımsa State hızlısından State-Sync
```
SNAP_RPC=https://elys.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

wget https://anode.team/unsafe-reset-all.sh && chmod u+x unsafe-reset-all.sh && ./unsafe-reset-all.sh elysd .elys

peers="d9f2e28e398d42fe7ca8ed322ee168b3e867bc95@65.108.199.222:34656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.elys/config/config.toml

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.elys/config/config.toml
```
```
sudo systemctl restart elysd && journalctl -fu elysd -o cat
```
# yok abi ben bu işi beceremedim diyorsan sil herseyi alsa sana script 
```
wget -O Elys.sh https://anode.team/Elys/test/Elys.sh && chmod u+x Elys.sh && ./Elys.sh
```
