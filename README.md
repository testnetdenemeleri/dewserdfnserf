# Drosera Node Kurulum Rehberi (SystemD, Docker'sız)

Bu rehber, sistem gereksinimi düşük olan @DroseraNetwork node'unu kolayca kurabilmeniz için hazırlanmıştır. Boşta duran VPS’lerinizde veya ücretsiz sunucularda çalıştırabilirsiniz.

## Sistem Gereksinimleri

| Gereksinim              | Detaylar                                 |
|------------------------|------------------------------------------|
| CPU Mimarisi           | amd64 veya arm64                         |
| RAM                    | En az 4 GB RAM                          |
| Depolama               | En az 20 GB SSD                         |
| İşletim Sistemi        | Ubuntu 20.04 veya üzeri                 |
| Cüzdan Gereksinimi     | Holesky ağı üzerinde ETH bulunan EVM cüzdan |

## Sunucu Önerileri

**Ücretsiz:**
- https://www.digitalocean.com/ → Kredi veriyor, kredi ile sunucu kiralayabilirsiniz.

**Ücretli:**
- https://contabo.com/en/vps/cloud-vps-4c/ → 6$ civarı, en ucuzunu seçebilirsiniz.

## Kurulum Öncesi Gereklilikler

- Yeni bir MetaMask cüzdanı oluşturun.
- Bu cüzdana Holesky ETH yükleyin.

Faucet bağlantısı: https://www.alchemy.com/faucets

## Sunucuya Bağlanma

```bash
ssh root@[Sunucu_IP]
```

## Gerekli Güncellemeleri ve Araçları Yükleyin

```bash
sudo apt-get update && sudo apt-get upgrade -y && \
sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## CLI Kurulumları

```bash
curl -L https://app.drosera.io/install | bash
source /root/.bashrc
droseraup

curl -L https://foundry.paradigm.xyz | bash
source /root/.bashrc
foundryup

curl -fsSL https://bun.sh/install | bash
source /root/.bashrc
```

## Trap (Tuzak) Oluşturma

```bash
mkdir my-drosera-trap && cd my-drosera-trap
git config --global user.email "mail-adresini-gir"
git config --global user.name "github-kullanici-adi"
```

## Template'i Çekip Derleme

```bash
forge init -t drosera-network/trap-foundry-template
bun install
forge build
```

## Deploy Etme

```bash
DROSERA_PRIVATE_KEY="kendi-privatekey"
drosera apply
```

## Kontrol Et ve Bloom Boost İşlemi

1. https://app.drosera.io adresine girin
2. Cüzdanınızı bağlayın
3. "Traps Owned" sekmesinden trap'e tıklayın
4. "Send Bloom Boost" butonuna basın ve bir miktar Holesky ETH yatırın

## Whitelist Ayarı

```bash
cd ~/my-drosera-trap
nano drosera.toml
```

```toml
private_trap = true
whitelist = ["0xSeninPublicCuzdanAdresin"]
```

```bash
DROSERA_PRIVATE_KEY=senin_privatekeyin drosera apply
```

## Operatör CLI Kurulumu

```bash
cd ~
curl -LO https://github.com/drosera-network/releases/releases/download/v1.16.2/drosera-operator-v1.16.2-x86_64-unknown-linux-gnu.tar.gz
tar -xvf drosera-operator-*.tar.gz
sudo cp drosera-operator /usr/bin
drosera-operator --version
```

## Alchemy RPC Alma

1. https://dashboard.alchemy.com adresine git
2. Ethereum → Holesky ağı için bir uygulama oluştur
3. Uygulama sayfasından özel RPC linkini kopyala

## SystemD Servisi Oluşturma

```bash
sudo tee /etc/systemd/system/drosera.service > /dev/null <<EOF
[Unit]
Description=drosera node service
After=network-online.target

[Service]
User=$USER
Restart=always
RestartSec=15
LimitNOFILE=65535
ExecStart=$(which drosera-operator) node \
  --db-file-path $HOME/.drosera.db \
  --network-p2p-port 31313 \
  --server-port 31314 \
  --eth-rpc-url Oluşturduğun_RPC \
  --eth-backup-rpc-url https://1rpc.io/holesky \
  --drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8 \
  --eth-private-key Private_Keyin \
  --listen-address 0.0.0.0 \
  --network-external-p2p-address VPS_IP_Adresin \
  --disable-dnr-confirmation true

[Install]
WantedBy=multi-user.target
EOF
```

## Node'u Başlatma ve Kontrol Etme

```bash
sudo systemctl daemon-reload
sudo systemctl enable drosera
sudo systemctl start drosera
sudo systemctl status drosera
journalctl -u drosera.service -f
```

## Opt-in ve Çalışırlığı Doğrulama

1. Trap sayfasından "Opt-in" butonuna tıklayın
2. Operator Status yeşil olmalı
3. Rewards kısmı işlemeye başlamalı

---

Herhangi bir sorunuz olursa: https://t.me/ViriyaChat