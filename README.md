<h1 align="center"> Aligned Layer

![image](https://github.com/molla202/AlignedLayer/assets/91562185/cb9fca1a-a370-4aee-b41d-546d27c4523b)



</h1>


 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>
 * [Aligned Layer Website](https://alignedlayer.com/)<br>
 * [Blockchain Explorer](https://explorer.corenodehq.com/Alignedlayer-Testnet)<br>
 * [Discord](https://discord.gg/zHsXryD7)<br>
 * [Twitter](https://twitter.com/alignedlayer)<br>

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

### 🚧 Go kurulumu
```
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### Dosyaları çekelim
```
cd $HOME
wget https://github.com/yetanotherco/aligned_layer_tendermint/releases/download/v0.1.0/alignedlayerd
chmod +x alignedlayerd
sudo mv alignedlayerd /usr/local/bin/
cd $HOME
alignedlayerd version
```
### Varyasyonlar
```
NODE_HOME=$HOME/.alignedlayer
CHAIN_BINARY=alignedlayerd
CHAIN_ID=alignedlayer
PEER_ADDRESSES=("91.107.239.79" "116.203.81.174" "88.99.174.203" "128.140.3.188")
```
### İnit
NOT: nodename kendi node adınızı ayzabilirsiniz.
```
alignedlayer init NodeName \
    --chain-id alignedlayer --overwrite
```
### Genesis Addrbook ve Ayarlar
```
curl -Ls https://raw.githubusercontent.com/molla202/AlignedLayer/main/genesis.json > $HOME/.alignedlayer/config/genesis.json
curl -Ls https://raw.githubusercontent.com/molla202/AlignedLayer/main/addrbook.json > $HOME/.alignedlayer/config/addrbook.json


for ADDR in "${PEER_ADDRESSES[@]}"; do
    GENESIS=$(curl -f "$ADDR:26657/genesis" | jq '.result.genesis')
    if [ -n "$GENESIS" ]; then
        echo "$GENESIS" > $NODE_HOME/config/genesis.json;
        break;
    fi
done

for ADDR in "${PEER_ADDRESSES[@]}"; do
    PEER_ID=$(curl -s "$ADDR:26657/status" | jq -r '.result.node_info.id')
    if [ -n "$PEER_ID" ]; then
        PEERS+=("$PEER_ID@$ADDR:26656")
    fi
done

PEER_LIST=$(IFS=,; echo "${PEERS[*]}")

$CHAIN_BINARY config set config p2p.persistent_peers "$PEER_LIST" --skip-validate
$CHAIN_BINARY config set config rpc.laddr "tcp://0.0.0.0:26657" --skip-validate

sed -i -e "s|^seeds *=.*|seeds = \"d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.nodex.one:24210\"|" $HOME/.alignedlayer/config/config.toml
sed -i -e 's|^persistent_peers *=.*|persistent_peers ="dc2011a64fc5f888a3e575f84ecb680194307b56@148.251.235.130:20656,a1a98d9caf27c3363fab07a8e57ee0927d8c7eec@128.140.3.188:26656,1beca410dba8907a61552554b242b4200788201c@91.107.239.79:26656,f9000461b5f535f0c13a543898cc7ac1cd10f945@88.99.174.203:26656, 32fbefec592ac2ff9ecb3cad69bafaaad01e771a@148.251.235.130:20656,81138177a67195791bbe782fe1ed49f25e582bac@91.107.239.79:26656,c5d0498e345725365c1016795eecff4a67e4c4c9@88.99.174.203:26656,14af04afc663427604e8dd53f4023f7963a255cb@116.203.81.174:26656,9c89e77d51561c8b23957eee85a81ccc99fa7d6b@128.140.3.188:26656,c355b86c882d05a83f84afba379291d7b954b28f@65.108.236.43:26656,b499b9eb88c1c78ae25fdc7c390090f7542160eb@167.235.12.38:26656,18e1adeadb8cc596375e4212288fcd00690df067@213.199.48.195:26656,6d7adb46e588bea496f33758e0448bf84e308b39@143.244.178.205:26656,de193ba0ae387fc7892c2ead7458202f1c035d69@38.242.137.235:26656"|' $HOME/.alignedlayer/config/config.toml
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001stake\"|" $HOME/.alignedlayer/config/app.toml

sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.alignedlayer/config/app.toml
```
### Servis
```
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null <<EOF
[Unit]
Description=alignedlayerd
After=network-online.target
[Service]
User=root
ExecStart=$(which alignedlayerd) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
### Başlatalım
```
cd $HOME
sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd
sudo systemctl restart alignedlayerd
```
```
sudo journalctl -u alignedlayerd -f --no-hostname -o cat
```


















