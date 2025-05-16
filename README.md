<h1 align="center"> Lumera </h1>







 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>


### Public RPC



### Explorer

### Faucet

https://faucet.testnet.lumera.io/

## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	4|
| RAM	| 8+ GB |
| Storage	| 400 GB SSD |




### 🚧Gerekli kurulumlar
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### 🚧Go kurulumu
```
ver="1.22.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```
### 🚧Dosyaları çekelim
```
cd $HOME
curl -LO https://github.com/LumeraProtocol/lumera/releases/download/v1.0.1/lumera_v1.0.1_linux_amd64.tar.gz
tar -xvf lumera_v1.0.1_linux_amd64.tar.gz
rm lumera_v1.0.1_linux_amd64.tar.gz
rm install.sh
chmod +x lumerad
```
```
mkdir -p $HOME/.lumera/cosmovisor/genesis/bin
mv $HOME/lumerad $HOME/.lumera/cosmovisor/genesis/bin
mv libwasmvm.x86_64.so $HOME/.lumera/
```
```
echo 'export LD_LIBRARY_PATH=$HOME/.lumera:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```
```
sudo ln -s $HOME/.lumera/cosmovisor/genesis $HOME/.lumera/cosmovisor/current -f
sudo ln -s $HOME/.lumera/cosmovisor/current/bin/lumerad /usr/local/bin/lumerad -f
```
### 🚧Cosmovisor kuralım
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### 🚧Servis oluşturalım
```
sudo tee /etc/systemd/system/lumerad.service > /dev/null << EOF
[Unit]
Description=lumerad node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.lumera"
Environment="DAEMON_NAME=junctiond"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.lumera/cosmovisor/current/bin"
Environment="LD_LIBRARY_PATH=$HOME/.lumera"

[Install]
WantedBy=multi-user.target
EOF
```
### 🚧Etkinleştirelim
```
sudo systemctl daemon-reload
sudo systemctl enable lumerad
```
### 🚧İnit
```
lumerad init "MONIKER" --chain-id lumera-testnet-1
```
### 🚧Genesis ve addrbook
```
curl -L https://raw.githubusercontent.com/Core-Node-Team/Lumera/refs/heads/main/addrbook.json > $HOME/.lumera/config/addrbook.json
curl -L https://raw.githubusercontent.com/Core-Node-Team/Lumera/refs/heads/main/genesis.json > $HOME/.lumera/config/genesis.json
```
### 🚧Port
```
echo "export LUM_PORT="28"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
sed -i.bak -e "s%:1317%:${LUM_PORT}317%g;
s%:8080%:${LUM_PORT}080%g;
s%:9090%:${LUM_PORT}090%g;
s%:9091%:${LUM_PORT}091%g;
s%:8545%:${LUM_PORT}545%g;
s%:8546%:${LUM_PORT}546%g;
s%:6065%:${LUM_PORT}065%g" $HOME/.lumera/config/app.toml
```
### Port
```
sed -i.bak -e "s%:26658%:${LUM_PORT}658%g;
s%:26657%:${LUM_PORT}657%g;
s%:6060%:${LUM_PORT}060%g;
s%:26656%:${LUM_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${LUM_PORT}656\"%;
s%:26660%:${LUM_PORT}660%g" $HOME/.lumera/config/config.toml
```
### 🚧Seed ve Peer
```
peers="$(curl -sS https://lumera-testnet-rpc.rpcdot.com:443/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | sed -z 's|\n|,|g;s|.$||')"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lumera/config/config.toml

```

### 🚧Pruning
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.lumera/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.lumera/config/app.toml
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.lumera/config/config.toml
```
### 🚧Gas ve index ayarı
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025ulume\"/" $HOME/.lumera/config/app.toml
```
### Snap 
```
soon
```
### 🚧Başlatalım
```
sudo systemctl daemon-reload && sudo systemctl start lumerad && sudo journalctl -u lumerad -f --no-hostname -o cat
```
### Log
```
sudo journalctl -u lumerad -f --no-hostname -o cat
```
### Cüzdan olusturma
```
lumerad keys add cüzdan-adi-yaz
```
### Cüzdan import
```
lumerad keys add cüzdan-adi-yaz --recover
```
### Validator oluştur

cd $HOME
# Create validator.json file
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(lumerad comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000ulume\",
    \"moniker\": \"test\",
    \"identity\": \"AAA\",
    \"website\": \"AAA\",
    \"security\": \"AAA\",
    \"details\": \"AAA\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validatorlu.json



lumerad tx staking create-validator $HOME/.lumera/validatorlu.json \
--from wallet \
--chain-id lumera-testnet-1 \
--gas-prices=0.025ulume \
--gas-adjustment=1.5 \
--gas=auto 


### unjail
```
lumerad tx slashing unjail --from wallet --chain-id lumera-testnet-1 --gas-prices=0.025ulume --gas-adjustment=1.5 --gas=auto
```
### kendine stake
```
lumerad tx staking delegate $(lumerad keys show wallet --bech val -a) 1000000ulume --from wallet --chain-id lumera-testnet-1 --gas-prices=0.025ulume --gas-adjustment=1.5 --gas=auto
```

### Delete node
```
sudo systemctl stop lumerad
sudo systemctl disable lumerad
sudo rm /etc/systemd/system/lumerad.service
sudo systemctl daemon-reload
rm -f $(which lumerad)
rm -rf .lumera
```





