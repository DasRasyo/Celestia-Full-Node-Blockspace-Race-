# Celestia Full Storage Node (blockspace-race) Kurulum Rehberi

![Ekran görüntüsü 2023-03-28 040506](https://user-images.githubusercontent.com/94050636/228100509-865911b0-e167-40e5-9049-7538e0017276.png)


## Minimum Önerilen Sistem Gereksinimleri

Memory: 8 GB RAM <br/>
CPU: Quad-Core <br/>
Disk: 250 GB SSD Storage <br/>
Bandwidth: 1 Gbps for Download/100 Mbps for Upload <br/>


## Sistem güncellemerini yaparak başlıyoruz.

```
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential \
git make ncdu -y
```

## Go kurulumu ile devam ediyoruz.

```
ver="1.19.4"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

go versiyon komutunun çıktısı şu şekilde olmalıdır:

```
go version go1.19.4 linux/amd64
```

## celestia-node Kurulumu

```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.8.1 
make build 
make install 
make cel-key
```

Celestia versiyon kontrolü:

```
celestia version
```

Çıktı şu şekilde olmalı:

```
Semantic version: v0.8.1 
Commit: 2718b1dfb7ee4fbcc8614601dc7d58019bfb1437
```

## Init işlemini yapalım

NOT: Aşağıdaki komutu girdiğinizde my_celes_key adında cüzdan oluşacak. Cüzdan adresi ve mnemonicler çıkacak. Bunları kaydetmeyi unutmayın!!!

```
celestia full init --p2p.network blockspacerace
```

## Cüzdan adresi ve adını şu komut ile görüntüleyebilirsiniz:

```
./cel-key list --node.type full --p2p.network blockspacerace
```

## Servis oluşturalım

```
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-fulld.service
[Unit]
Description=celestia-fulld Full Node
After=network-online.target

[Service]
User=$USER
ExecStart=/usr/local/bin/celestia full start --core.ip https://rpc-blockspacerace.pops.one --core.rpc.port 26657 --core.grpc.port 9090 --keyring.accname my_celes_key --metrics.tls=false --metrics --metrics.endpoint otel.celestia.tools:4318 --gateway --gateway.addr localhost --gateway.port 26659 --p2p.network blockspacerace
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

Not: ExecStart komutundaki core ip, rpc port, grpc port, [Celestia dökümanlarında](https://docs.celestia.org/nodes/blockspace-race/#rpc-endpoints) sağlanan endpointlerden alınmıştır ve alternatifleri vardır. İsterseniz burayı değiştirip farklı endpointler kullanabilirsiniz.

Aşağıdaki komutlarla node'umuzu başlatıyoruz.

```
systemctl enable celestia-fulld
systemctl start celestia-fulld
```

Log kontrolü

```
journalctl -u celestia-fulld.service -f
```

## Node ID öğrenme

Not: Aşağıdaki iki komutu girerek node ID'mizi öğreneceğiz. Bu node ID ile Celestia Full Storage Node'umuzu [Tiascan](https://tiascan.com/full-storage) üzerinden uptime gibi çeşitli verilerle birlikte takip edebileceğiz. Ayrıca Knack Portal'da tasklarda node ID'mizi girmemiz gerekecek. Bu ID ile node'umuz takip edilecek, çalışma süresi izlenecek.

```
AUTH_TOKEN=$(celestia full auth admin --p2p.network blockspacerace)


curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```

Bu iki komutu girdiğinizde şuna benzer bir çıktı almalısınız;

```
{"jsonrpc":"2.0","result":{"ID":"12D3KooWFDtXXXXXXXXXXXXXXX"
```

12D3 ile başlayan node'umuzun ID'sidir. [Tiascan](https://tiascan.com/full-storage) üzerinden aratabilirsiniz.

## Yedeklenmesi Gereken Dosyalar

WinSCP veya aynı işleve sahip bir araç ile .celestia-full-blockspacerace-0 klasörünü altında bulunan keys klasörünü yedeklemeniz gerekmektedir. Key bilgilerimiz burada. Knack portala node bilgilerinizi kaydetmeden önce bu klasörü mutlaka yedekleyin!

## BAŞARILAR!
