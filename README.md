# Celestia Node & Validator kurulumu

[![Go Reference](https://pkg.go.dev/badge/github.com/celestiaorg/celestia-node.svg)](https://pkg.go.dev/github.com/celestiaorg/celestia-node)
[![GitHub release (latest by date including pre-releases)](https://img.shields.io/github/v/release/celestiaorg/celestia-node)](https://github.com/celestiaorg/celestia-node/releases/latest)
[![test](https://github.com/celestiaorg/celestia-node/actions/workflows/test.yml/badge.svg)](https://github.com/celestiaorg/celestia-node/actions/workflows/test.yml)
[![lint](https://github.com/celestiaorg/celestia-node/actions/workflows/lint.yml/badge.svg)](https://github.com/celestiaorg/celestia-node/actions/workflows/lint.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/celestiaorg/celestia-node)](https://goreportcard.com/report/github.com/celestiaorg/celestia-node)
[![codecov](https://codecov.io/gh/celestiaorg/celestia-node/branch/main/graph/badge.svg?token=CWGA4RLDS9)](https://codecov.io/gh/celestiaorg/celestia-node)

![image](https://user-images.githubusercontent.com/101149671/170825153-0af8d236-45b7-4382-9f68-4e544c23f4e1.png)

Celestia Ağındaki Düğüm (Node) Türleri  (`light` | `full` | `bridge`).

Yukarıda açıklanan celestia düğümü türleri, celestia veri kullanılabilirliği (DA) ağını içerir.

DA ağı, konsensüs ağından gelen blokları dinleyerek ve onları veri kullanılabilirliği örneklemesi (DAS) için sindirilebilir hale getirerek celestia çekirdekli konsensüs ağını sarar.

Detaylı Bilği için [Bknz](https://blog.celestia.org/celestia-mvp-release-data-availability-sampling-light-clients) DAS ve Celestia zincir verilerine nasıl güvenli ve ölçeklenebilir erişim sağladığı hakkında daha fazla bilgi edinmek istiyorsanız.

## Yazılım Gereksinimleri

| Gereksinim  | Not            |
|-------------|----------------|
| Go Sürümü   | 1.17+          |

## Sistem Gereksinimleri 


Düğüm türü başına sistem gereksinimleri için resmi belgeler sayfasına bakın:
* [Bridge](https://docs.celestia.org/nodes/bridge-validator-node#hardware-requirements)
* [Light](https://docs.celestia.org/nodes/light-node#hardware-requirements)
* [Full](https://docs.celestia.org/nodes/full-node#hardware-requirements)

## Validator Node Kurulumu (Mamaki)
Mamaki Testnetinde Validator Node kurulumu, Sunucunuza baglanıp adım adım ilerleyin.
[Explorer](https://celestia.explorers.guru/). 

# Validator Node Gereksinimleri 

|    Bellek   |       Cpu      |      Disk      |   Ağ           |
|-------------|----------------|----------------|----------------|
|     8GB     |   Quad-Core    |     250GB      |  1Gbps/100Mbps |

Bunlar Önerilen sistem gereksinimleridir.
Farklı sistemlerde de çalıştırmayı deneyebilirsiniz. Sunucunuza baglanıp Komutları Sırayla Uygulayın.
"//" ile başlayan satırlar açıklama satırıdır. Komut degildir.
Kurulum Ubuntu 20.04 içindir.

# Gerekli Paketlerin kurulumu
Önce Gerekli Kütüphane ve Güncellemeler ile başlayalım
```sh
//Sistem Güncellemesi
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar screen wget clang pkg-config libssl-dev jq build-essential \
git make ncdu -y

//Go Kurulumu
ver="1.17.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile

```
# Celestia Kurulumu
```sh
cd $HOME
screen -S celestia
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app/
APP_VERSION=$(curl -s \
  https://api.github.com/repos/celestiaorg/celestia-app/releases/latest \
  | jq -r ".tag_name")
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
```
# Node Budama ve Moniker Ayarları
Node'umuzun gerekli ayarları ile devam edelim
```sh
//<node-ismi> kendinize göre güncelleyin
celestia-appd init <node-ismi> --chain-id mamaki

//Budama ayarları
pruning="custom"
pruning_keep_recent="100"
pruning_interval="10"

sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$pruning_keep_recent\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$pruning_interval\"/" $HOME/.celestia-app/config/app.toml
```
# Cüzdan işlemler
Node'umuza Cüzdan bilgilerini girelim. Yeni cüzdan açacak ise cüzdanınızın kurtarma bilgilerini not edin.
Faucet Token Almayı unutmayın. [Discord Faucet](https://discord.gg/JeRdw5veKu).
```sh
//Yeni Cüzdan için
celestia-appd keys add <cüzdan-ismi>
//Eski Cüzdan Geri yükleme
celestia-appd keys add walletname --recover
```
# Genesis & Peer ayarları
Node kurulumumuza genesis ve peers ayarları ile devam edelim doğru bir baglantı için önemli.

```sh
wget -O $HOME/.celestia-app/config/genesis.json "https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/genesis.json"

BOOTSTRAP_PEERS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/bootstrap-peers.txt | tr -d '\n') && echo $BOOTSTRAP_PEERS
sed -i.bak -e "s/^bootstrap-peers *=.*/bootstrap-peers = \"$BOOTSTRAP_PEERS\"/" $HOME/.celestia-app/config/config.toml

PEERS="7145da826bbf64f06aa4ad296b850fd697a211cc@176.57.189.212:26656, f7b68a491bae4b10dbab09bb3a875781a01274a5@65.108.199.79:20356, 853a9fbb633aed7b6a8c759ba99d1a7674b706a3@38.242.216.151:26656, fbddf6bf8d172a96678cfcd04a887cb54b1fc9b7@70.34.211.176:26656, 96995456b7fe3ab6524fc896dec76d9ba79d420c@212.125.21.178:26656, 268694eaf9446b2052b1161979bf2e09f3e45e81@173.212.254.166:26656, 28aaa8865f3e9bba69f257b08d5c28091b5b3167@176.57.150.79:26656"
 
sed -i.bak -e "s/^persistent-peers *=.*/persistent-peers = \"$PEERS\"/" $HOME/.celestia-app/config/config.toml

sed -i.bak -e "s/^timeout-commit *=.*/timeout-commit = \"25s\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^skip-timeout-commit *=.*/skip-timeout-commit = false/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^mode *=.*/mode = \"validator\"/" $HOME/.celestia-app/config/config.toml
```
# Sistem Dosyası Oluşturma
Node'umuzun daha sağlıklı çalıştıgına emin olmak için Sistem dosyası oluşturmamız gerek. 
Bu sayede aksi bir durumda yeniden başlıcaktır.
```sh
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-appd.service
[Unit]
Description=celestia-appd Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/celestia-appd start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
Bu komutdan sonra Servis dosyası oluşacaktır
```sh
cat /etc/systemd/system/celestia-appd.service
```
# Node'umuzu Başlatalım
```sh
cd $HOME/.celestia-app
celestia-appd tendermint unsafe-reset-all --home "$HOME/.celestia-app"
sudo systemctl enable celestia-appd
sudo systemctl start celestia-appd
sudo journalctl -u celestia-appd.service -f
```
Şuan screen ekranı içerisinde Node'umuzun Çalıştırdık ve Gerçek zamanlı izlemeyi açtık. 
CTRL+A+D ile Screen'den Çıkabiliriz daha sonra tekrar baglandıgımızda. screen -r ile girebiliriz

# Validator oluşturma
Node başarıyla eşleştikten sonra validatör oluşturmanız gerek.
```sh
celestia-appd tx staking create-validator \
    --amount=<stake-miktarı> \
    --pubkey=$(celestia-appd tendermint show-validator) \
    --moniker=MonikerAdi \
    --chain-id=mamaki \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1000000 \
    --from=<cüzdan-ismi>
    
//Örnek Düzenlenmiş komut
celestia-appd tx staking create-validator \
    --amount=1000000utia \
    --pubkey=$(celestia-appd tendermint show-validator) \
    --moniker=MonikerAdi \
    --chain-id=mamaki \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1000000 \
    --from=rueswallet
    
//Ekstra token Delegate için
celestia-appd tx staking delegate <validator-adres> 1000000utia --chain-id mamaki --fees=1utia --from <cüzdan-ismi>
//Jailled olursanız Unjail işlemi için
celestia-appd tx slashing unjail --from=<cüzdan-ismi> --chain-id mamaki

```
Düğüm kurma ve gereken donanım gereksinimleri hakkında daha fazla bilgi için şu adresteki Orjinal Belgeleri ziyaret edin: <https://docs.celestia.org>.

Yazı Tamamen Açık kaynaktır. İstediginiz gibi kullanabilirsiniz.
## API docs

Celestia düğümü genel API'si belgelenmiştir. [oku](https://docs.celestia.org/developers/node-api/).


Daha fazla bilgiyi Burdan bulabilirsiniz [oku](https://github.com/celestiaorg/celestia-node/blob/main/docs/adr/adr-003-march2022-testnet.md#legend).


