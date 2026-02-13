# FortyTwo Network Relay Node Kurulum Rehberi (Ubuntu 24.04)

Resmi dokümantasyon: https://docs.fortytwo.network/docs/relay-quick-start

## 🖥️ Sistem Gereksinimleri

### Donanım
- **CPU:** x86_64 mimarisinde işlemci
- **RAM:** Minimum 4GB (8GB önerilir)
- **Disk:** Minimum 20GB boş alan
- **İnternet:** Statik genel IP adresi (zorunlu)

### Yazılım
- **İşletim Sistemi:** Ubuntu 24.04 LTS
- **Docker:** En son sürüm
- **Git:** En son sürüm
- **Web3 Cüzdan:** EVM uyumlu cüzdan private key'i

### Ağ Gereksinimleri
- **Statik genel IP** zorunludur
- **24/7 çalışma süresi** önerilir
- **Açık portlar:** 42042 (TCP/UDP), 4001 (TCP/UDP), 42420 (TCP)

## 🔐 Ön Hazırlık

###  IP Adresi Kontrolü
```bash
# Sunucunuzun genel IP adresini öğrenin
curl -4 ifconfig.me
```
Bu IP adresini not edin, kurulum sırasında kullanacaksınız.

## 📦 Kurulum Adımları

### Adım 1: Sistemi Güncelleme

```bash
# Paket listelerini güncelle
sudo apt update && sudo apt upgrade -y

### Adım 2: Docker Kurulumu (Docker kuruluysa version kontrol et ve gerekliyse güncelle)

```bash
# Eski Docker sürümlerini kaldır
sudo apt remove docker docker-engine docker.io containerd runc

# Gerekli paketleri yükle
sudo apt install -y ca-certificates curl gnupg lsb-release

# Docker GPG anahtarını ekle
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Docker repository'sini ekle
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker'ı yükle
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Kullanıcıyı docker grubuna ekle
sudo usermod -aG docker $USER

# Grup değişikliklerini uygula
newgrp docker

# Docker servisini başlat ve etkinleştir
sudo systemctl start docker
sudo systemctl enable docker

# Kurulumu doğrula
docker --version
docker compose version
```

**Beklenen çıktı:**
```
Docker version 24.x.x, build xxxxxxx
Docker Compose version v2.x.x
```

### Adım 3: Git Kurulumu (Varsa geç)

```bash
# Git'i yükle
sudo apt install -y git

# Kurulumu doğrula
git --version
```

### Adım 4: Repository'yi Klonlama

```bash
# Kurulum dizini oluştur
mkdir -p ~/FortytwoRelay && cd ~/FortytwoRelay

# Official repository'yi klonla
git clone https://github.com/Fortytwo-Network/fortytwo-relay-setup
cd fortytwo-relay-setup

# Repository içeriğini kontrol et
ls -la
```

**Aşağıdaki dosyaları görmelisin**
```
.env.example
docker-compose.yml
Dockerfile
README.md
```

### Adım 5: Gerekli Klasörleri Oluşturma

```bash
# IPFS veri klasörlerini oluştur
sudo mkdir -p /data/ipfs
sudo mkdir -p /data/ipfs_ipns

# Tam izin ver (host network mode için gerekli)
sudo chmod -R 777 /data/ipfs
sudo chmod -R 777 /data/ipfs_ipns

# Klasörleri doğrula
ls -la /data/
```

## ⚙️ Yapılandırma

### 1. Environment Dosyasını Oluşturma

```bash
cd ~/FortytwoRelay/fortytwo-relay-setup

# .env dosyasını oluştur
cp .env.example .env

# Düzenle
nano .env
```

### 2. .env Dosyası Yapılandırması

**.env dosyasının içeriği:**

```bash
# Whitelisted wallet private key (0x ile başlamalı)
FT_ACCOUNT_PRIVATE_KEY=0xYOUR_PRIVATE_KEY_HERE

# RPC servis portu
FT_RPC_SERVICE_PORT=42420

# Node dinleme portu
FT_NODE_LISTENER_PORT=42042

# IPFS swarm portu
IPFS_SWARM_PORT=4001

# IPFS veri yolları
IPFS_DATA_PATH=/data/ipfs
IPFS_MOUNT_PATH=/data/ipfs_ipns

# Sunucunuzun genel IP adresi
HOST_PUBLIC_IP=YOUR_PUBLIC_IP_HERE
```

**Örnek:**
```bash
FT_ACCOUNT_PRIVATE_KEY=Maildeki Cüzdanının Privkeyi
FT_RPC_SERVICE_PORT=42420
FT_NODE_LISTENER_PORT=42042
IPFS_SWARM_PORT=4001
IPFS_DATA_PATH=/data/ipfs
IPFS_MOUNT_PATH=/data/ipfs_ipns
HOST_PUBLIC_IP=Sunucunun İp adresi
```

**Kaydetmek için:** `Ctrl + O`, `Enter`, ardından `Ctrl + X`


## 🔥 Firewall Yapılandırması

### Ubuntu UFW Ayarları

```bash
# UFW'yi yükle (yoksa)
sudo apt install -y ufw

# Gerekli portları aç
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 42042/tcp   # FT Node Listener
sudo ufw allow 42042/udp   # FT Node Listener
sudo ufw allow 4001/tcp    # IPFS Swarm
sudo ufw allow 4001/udp    # IPFS Swarm
sudo ufw allow 42420/tcp   # FT RPC Service

# UFW'yi etkinleştir
sudo ufw enable

# Durumu kontrol et
sudo ufw status verbose
```

**Beklenen çıktı:**
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
42042/tcp                  ALLOW       Anywhere
42042/udp                  ALLOW       Anywhere
4001/tcp                   ALLOW       Anywhere
4001/udp                   ALLOW       Anywhere
42420/tcp                  ALLOW       Anywhere
```

## 🚀 Node'u Başlatma

### 1. Son Güncellemeleri Çekin

```bash
cd ~/FortytwoRelay/fortytwo-relay-setup

# Repository'yi güncelle
git pull
```

### 2. Docker Container'ları Build Edin

```bash
# Cache kullanmadan temiz build
docker compose build 
```

**Beklenen çıktı:**
```
[+] Building 45.2s (12/12) FINISHED
...
=> => writing image sha256:...
```

### 3. Container'ları Başlatın

```bash
# Arka planda başlat
docker compose up -d
```

**Beklenen çıktı:**
```
[+] Running 2/2
 ✔ Container fortytwo-ipfs-kubo  Started
 ✔ Container fortytwo-relay      Started
```

### 4. Container Durumunu Kontrol Edin

```bash
# Çalışan container'ları listele
docker ps
```

**Beklenen çıktı:**
```
CONTAINER ID   IMAGE              STATUS                   PORTS
xxxxx          ipfs/kubo:v0.39.0  Up 2 minutes (healthy)
xxxxx          fortytwo-relay     Up 2 minutes
```

---

## ✅ Doğrulama ve Test

### 1. Relay Node Loglarını İzleyin

```bash
# Canlı log izleme
docker logs -f fortytwo-relay

# Son 50 satır
docker logs --tail 50 fortytwo-relay
```

## Eğer docker log komutları çalışmassa:
```bash
docker ps

# docker listesinden CONTAINER ID bul
docker logs CONTAINER ID -f
```

**BAŞARILI çıktı örneği:**
```
UTC 2026-02-14 12:00:00  INFO Fortytwo Protocol Relay is starting up
UTC 2026-02-14 12:00:00  INFO Fortytwo Protocol Relay current version: 0.1.11
UTC 2026-02-14 12:00:00  INFO Operator Wallet Address: 0x...
UTC 2026-02-14 12:00:00  INFO Address 0x... is whitelisted
UTC 2026-02-14 12:00:00  INFO Check health
UTC 2026-02-14 12:00:00  INFO Relay address is public. Current address /ip4/38.49.212.137/tcp/42042
```


## 📞 Destek ve Topluluk

### Resmi Kaynaklar
- **Resmi Dokümanlar:** https://docs.fortytwo.network/
- **Discord:** https://discord.com/invite/fortytwo
- **Website:** https://fortytwo.network/
- **GitHub:** https://github.com/Fortytwo-Network

### Sorun Bildirimi
Sorun bildirmeden önce:
1. Logları toplayın: `docker logs fortytwo-relay > relay-logs.txt`
2. Sistem bilgilerini toplayın: `uname -a > system-info.txt`
3. UFW durumunu kaydedin: `sudo ufw status verbose > ufw-status.txt`
4. Discord'da #support kanalında paylaşın

---

## 📄 Lisans ve Sorumluluk Reddi

Bu rehber topluluk tarafından hazırlanmıştır. Resmi FortyTwo Network dokümantasyonunu referans alarak oluşturulmuştur.

⚠️ **Sorumluluk Reddi:**
- Private key'lerinizi güvenli tutun
- Node işletimi risklerini anlayın
- Resmi Terms & Conditions'ı okuyun: https://docs.fortytwo.network/docs/legal-node-operation-tc

---

## 📚 Ek Kaynaklar

### Faydalı Linkler
- [Relay Requirements](https://docs.fortytwo.network/docs/relay-requirements)
- [Environment Setup](https://docs.fortytwo.network/docs/relay-env-setup)
- [Ports Verification](https://docs.fortytwo.network/docs/ports)
- [Relay Support](https://docs.fortytwo.network/docs/support-relay)

### Video Tutorials (Eğer varsa)
- YouTube kanalına bakın
- Discord'da #tutorials kanalını takip edin

---

## ✨ Katkıda Bulunma

Bu rehberi geliştirmek isterseniz:
1. Fork edin
2. İyileştirmelerinizi yapın
3. Pull request gönderin

---

**Son Güncelleme:** 2026-02-14  
**Versyon:** 1.0.0  
**Desteklenen Relay Version:** 0.1.11+

---

**🚀 Başarılar! Relay Node'unuz artık FortyTwo Network'e katkıda bulunuyor!**
