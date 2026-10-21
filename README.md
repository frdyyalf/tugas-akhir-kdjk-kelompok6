<div align="center">
    <img src="https://raw.githubusercontent.com/louislam/uptime-kuma/master/public/icon.svg" width="130" alt="Uptime Kuma Logo" />
    <h1>Laporan Proyek Akhir Komunikasi Data & Jaringan Komputer</h1>
    <h2>Uptime Kuma: Enterprise-Grade Self-Hosted Infrastructure & Network Monitoring</h2>
    <p><b>Program Studi Ilmu Komputer | IPB University</b></p>

[![Azure VPS](https://img.shields.io/badge/Cloud-Azure%20VM-0089D6?style=flat&logo=microsoftazure)](https://azure.microsoft.com/)
[![Docker](https://img.shields.io/badge/Container-Docker%20Engine-2496ED?style=flat&logo=docker)](https://www.docker.com/)
[![Reverse Proxy](https://img.shields.io/badge/Proxy-Caddy%20v2-1F88C0?style=flat&logo=caddy)](https://caddyserver.com/)
[![SSL Active](https://img.shields.io/badge/SSL%2FTLS-Let's%20Encrypt-003A70?style=flat&logo=letsencrypt)](https://letsencrypt.org/)
[![Status Page](https://img.shields.io/badge/Status%20Page-Public%20Access-00C853?style=flat)](https://kuma-kelompok6.indonesiacentral.cloudapp.azure.com/status/public)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

</div>

<hr />

## 👥 Informasi Kelompok 6
* **Program Studi**: Ilmu Komputer (S1) - Semester 4
* **Mata Kuliah**: Komunikasi Data dan Jaringan Komputer (KDKJ)
* **Anggota Kelompok**:
  1. **Ferdy Alfalah** (NIM: M0403241097)
  2. **Anggota Kelompok 2** (NIM: M040324xxxx)
  3. **Anggota Kelompok 3** (NIM: M040324xxxx)
* **Live Public Status Page**: [https://kuma-kelompok6.indonesiacentral.cloudapp.azure.com/status/public](https://kuma-kelompok6.indonesiacentral.cloudapp.azure.com/status/public)

---

## 📌 Daftar Isi
- [Sekilas Tentang & Latar Belakang](#-sekilas-tentang--latar-belakang)
- [Arsitektur & Topologi Jaringan](#-arsitektur--topologi-jaringan)
- [Fitur Utama & Protokol yang Didukung](#-fitur-utama--protokol-yang-didukung)
- [Spesifikasi Lingkungan Deployment](#-spesifikasi-lingkungan-deployment)
- [Panduan Instalasi & Deployment Lengkap](#-panduan-instalasi--deployment-lengkap)
  - [Metode A: Local Environment (Virtual Machine / WSL2)](#metode-a-local-environment-virtual-machine--wsl2)
  - [Metode B: Production Cloud VPS (Azure Deployment)](#metode-b-production-cloud-vps-azure-deployment)
- [Konfigurasi Monitoring & Aplikasi](#-konfigurasi-monitoring--aplikasi)
- [Otomatisasi Deployment](#-otomatisasi-deployment)
- [Maintenance & Operasional Server](#-maintenance--operasional-server)
- [Cara Pemakaian & Hasil Pengujian](#-cara-pemakaian--hasil-pengujian)
- [Pembahasan & Analisis Komdat](#-pembahasan--analisis-komdat)
- [Kesimpulan](#-kesimpulan)
- [Referensi](#-referensi)

---

## 📖 Sekilas Tentang & Latar Belakang
[`^ kembali ke atas ^`](#)

Dalam infrastruktur teknologi informasi modern, ketersediaan (*availability*) dan keandalan (*reliability*) layanan web serta infrastruktur jaringan merupakan kriteria kritis. Gangguan (*downtime*) pada server aplikasi, database, atau resolver DNS dapat mengakibatkan kerugian operasional yang signifikan. 

**Uptime Kuma** adalah aplikasi pemantauan (*monitoring tool*) infrastruktur open-source berbasis web yang dikembangkan oleh Louis Lam. Aplikasi ini dirancang sebagai solusi *self-hosted* berkinerja tinggi sebagai alternatif independen untuk layanan SaaS komersial seperti UptimeRobot, Datadog, atau Statuspage.io.

Proyek akhir ini bertujuan untuk merancang, mengimplementasikan, dan mengevaluasi sistem monitoring yang andal pada lingkungan **Azure Cloud VPS** menggunakan **Docker Engine** dan **Caddy Reverse Proxy**. Sistem ini dikonfigurasi untuk memantau berbagai layer jaringan (Layer 3, Layer 4, Layer 7, hingga Host-level Container Health) dan menyediakan **Status Page Publik** yang dapat diakses oleh *end-user* tanpa autentikasi.

---

## 🌐 Arsitektur & Topologi Jaringan
[`^ kembali ke atas ^`](#)

Sistem di-deploy di lingkungan Microsoft Azure Cloud dengan arsitektur *reverse proxy* dan kontainerisasi terisolasi.

```text
[ Client / Internet ]
        │
        │ HTTP (Port 80) / HTTPS (Port 443)
        ▼
┌────────────────────────────────────────────────────────────────────────┐
│ Azure Virtual Machine (Ubuntu 22.04 LTS)                                │
│ Public IP: 70.153.81.95                                                 │
│ FQDN: kuma-kelompok6.indonesiacentral.cloudapp.azure.com              │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ Network Security Group (NSG Firewall)                            │  │
│  │ Allowed Inbound: Port 22 (SSH), Port 80 (HTTP), Port 443 (HTTPS)  │  │
│  └─────────────────────────────────┬────────────────────────────────┘  │
│                                    │                                   │
│                                    ▼                                   │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ Caddy Reverse Proxy (Host Web Server)                            │  │
│  │ - Auto TLS / Let's Encrypt SSL Certificate                       │  │
│  │ - HTTP to HTTPS Redirect                                         │  │
│  │ - Proxy Pass: 127.0.0.1:3001                                     │  │
│  └─────────────────────────────────┬────────────────────────────────┘  │
│                                    │ Inter-process (Loopback)          │
│                                    ▼                                   │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ Docker Engine / Container Runtime                                │  │
│  │                                                                  │  │
│  │  ┌────────────────────────────────────────────────────────────┐  │  │
│  │  │ Uptime Kuma Container (louislam/uptime-kuma:2)               │  │  │
│  │  │ Internal Port: 3001                                          │  │  │
│  │  │ Volume: uptime-kuma -> /app/data (SQLite Persist)            │  │  │
│  │  │ Socket Mount: /var/run/docker.sock (Health Check)            │  │  │
│  │  └──────────────────────────────┬─────────────────────────────┘  │  │
│  └─────────────────────────────────┼────────────────────────────────┘  │
└────────────────────────────────────┼───────────────────────────────────┘
                                     │ Outbound Monitoring Traffic
                                     ▼
                     ┌───────────────────────────────┐
                     │ External Target Services      │
                     │ - HTTP/HTTPS (Portal IPB)     │
                     │ - DNS Resolver (Cloudflare)   │
                     │ - TCP Port (1.1.1.1:443)      │
                     │ - ICMP Ping (Google 8.8.8.8)  │
                     └───────────────────────────────┘
```

---

## ✨ Fitur Utama & Protokol yang Didukung
[`^ kembali ke atas ^`](#)

1. **Multi-Protocol Monitoring**:
   - **Layer 7 (Application)**: HTTP(S), DNS Record Resolution, WebSockets, HTTP Keyword Detection.
   - **Layer 4 (Transport)**: TCP Port Connectivity Checking.
   - **Layer 3 (Network)**: ICMP Ping RTT (Round Trip Time) Latency.
   - **Host / Container Level**: Direct Docker Socket Inspection (`/var/run/docker.sock`).
2. **High-Frequency Heartbeat**: Interval pengujian dapat dikonfigurasi mulai dari 20 detik hingga sesuai kebutuhan.
3. **Automatic SSL/TLS Expiry Monitoring**: Peringatan dini sebelum sertifikat SSL target kadaluarsa.
4. **Public Status Page**: Halaman publik terpisah dengan slug kustom (`/status/public`) untuk transparansi ketersediaan layanan.
5. **Interactive UI**: Antarmuka berbasis Vue 3 dan Socket.io untuk pembaruan statistik *real-time* tanpa *refresh* halaman.
6. **Alerting & Notifications**: Integrasi ke 90+ saluran notifikasi (Telegram, Discord, Slack, Email SMTP, Webhook).

---

## 💻 Spesifikasi Lingkungan Deployment
[`^ kembali ke atas ^`](#)

### 1. Spesifikasi Hardware & Cloud (Azure VPS)
- **Cloud Provider**: Microsoft Azure (Azure for Students)
- **Virtual Machine Size**: Standard_B1s (1 vCPU, 1 GiB RAM)
- **Operating System**: Ubuntu 22.04.4 LTS (Jammy Jellyfish) 64-bit
- **Disk Allocation**: 30 GiB Premium SSD
- **Public IP Address**: `70.153.81.95`
- **DNS Hostname (FQDN)**: `kuma-kelompok6.indonesiacentral.cloudapp.azure.com`

### 2. Software Stack
- **Container Engine**: Docker Engine v26.x & Docker Compose
- **Reverse Proxy**: Caddy Web Server v2.7
- **Application Core**: Uptime Kuma v2.x (Node.js runtime + SQLite3)
- **SSL Certificate**: Let's Encrypt Authority (ACME Protocol via Caddy)

---

## 🛠️ Panduan Instalasi & Deployment Lengkap
[`^ kembali ke atas ^`](#)

### Metode A: Local Environment (Virtual Machine / WSL2)

Metode ini cocok untuk pengujian lokal menggunakan VirtualBox, VMware, atau Windows Subsystem for Linux (WSL2).

1. **Persiapan Virtual Machine**:
   - Install Ubuntu Server 22.04 LTS pada VirtualBox.
   - Atur Network Adapter ke **Bridged Adapter** agar VM mendapatkan IP lokal dari router (misal: `192.168.1.50`).

2. **Instalasi Docker Engine**:
   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   sudo apt-get install -y ca-certificates curl gnupg lsb-release
   curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io
   ```

3. **Jalankan Uptime Kuma via Docker**:
   ```bash
   sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:2
   ```

4. **Akses Lokal**:
   Buka browser dan akses `http://<IP_VM_LOKAL>:3001` (misal: `http://192.168.1.50:3001`).

---

### Metode B: Production Cloud VPS (Azure Deployment)

Metode ini adalah **implementasi utama yang diterapkan oleh Kelompok 6** untuk menghasilkan service publik berbasis domain dan SSL HTTPS resmi.

#### Langkah 1: Pengaturan Azure Network Security Group (NSG)
Buka Portal Azure $\rightarrow$ Virtual Machine $\rightarrow$ **Networking**, lalu tambahkan **Inbound Port Rules**:
- `SSH` (Port 22 / TCP) $\rightarrow$ Allow
- `HTTP` (Port 80 / TCP) $\rightarrow$ Allow
- `HTTPS` (Port 443 / TCP) $\rightarrow$ Allow

#### Langkah 2: Setup Container Uptime Kuma dengan Docker Socket Access
Akses VPS via SSH:
```bash
ssh azureuser@kuma-kelompok6.indonesiacentral.cloudapp.azure.com
```

Jalankan Uptime Kuma dengan melakukan *mounting* volume data dan socket Docker agar Uptime Kuma dapat memantau kontainer lokal:
```bash
sudo docker run -d \
  --restart=always \
  -p 127.0.0.1:3001:3001 \
  -v uptime-kuma:/app/data \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name uptime-kuma \
  louislam/uptime-kuma:2
```
*Catatan Security*: Port `3001` diikat (*bound*) ke `127.0.0.1` agar aplikasi tidak dapat diakses langsung via port mentah, melainkan wajib melalui Reverse Proxy Caddy.

#### Langkah 3: Konfigurasi Caddy Reverse Proxy & SSL Otomatis
Install Caddy Web Server di Ubuntu:
```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf '[https://dl.cloudsmith.io/public/caddy/stable/gpg.key](https://dl.cloudsmith.io/public/caddy/stable/gpg.key)' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf '[https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt](https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt)' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy -y
```

Edit berkas konfigurasi Caddyfile (`/etc/caddy/Caddyfile`):
```bash
sudo nano /etc/caddy/Caddyfile
```

Isi dengan konfigurasi berikut:
```caddy
kuma-kelompok6.indonesiacentral.cloudapp.azure.com {
    reverse_proxy 127.0.0.1:3001
}
```

Restart dan aktifkan Caddy:
```bash
sudo systemctl restart caddy
sudo systemctl enable caddy
```
*Caddy akan secara otomatis meminta dan memperbarui sertifikat SSL Let's Encrypt serta mengalihkan semua lalu lintas HTTP (Port 80) ke HTTPS (Port 443).*

---

## ⚙️ Konfigurasi Monitoring & Aplikasi
[`^ kembali ke atas ^`](#)

### 1. Setup Awal Admin
Saat pertama kali membuka `https://kuma-kelompok6.indonesiacentral.cloudapp.azure.com`, buat akun Admin utama dengan mendaftarkan Username dan Password.

### 2. Konfigurasi Docker Host (Local Socket)
1. Masuk ke **Settings** $\rightarrow$ **Docker Hosts** $\rightarrow$ **Setup Docker Host**.
2. **Friendly Name**: `Local Docker`
3. **Docker Type**: `Socket`
4. **Docker Daemon Socket**: `/var/run/docker.sock`
5. Klik **Save**.

### 3. Pendaftaran Target Monitor (Multi-Layer)
Sistem Kelompok 6 dikonfigurasi dengan 6 target pemantauan representatif:

| Friendly Name | Monitor Type | Target / Hostname | Layer Jaringan | Tujuan Pemantauan |
| :--- | :--- | :--- | :--- | :--- |
| **Portal IPB University** | `HTTP(s)` | `https://ipb.ac.id/` | Layer 7 (App) | Ketersediaan Web Kampus & Masa Berlaku SSL |
| **GitHub** | `HTTP(s)` | `https://github.com/` | Layer 7 (App) | Ketersediaan Platform Repositori Kode |
| **Google Primary DNS** | `Ping` | `8.8.8.8` | Layer 3 (Network) | Latensi RTT Network & Packet Loss |
| **Cloudflare** | `TCP Port` | `1.1.1.1:443` | Layer 4 (Transport) | Keandalan Port 443 TCP Resolver |
| **Cloudflare** | `DNS` | `google.com` via `1.1.1.1` | Layer 7 (App) | Kecepatan Resolusi A-Record DNS |
| **Uptime Kuma Health** | `Docker Container`| Container Name: `uptime-kuma` | Host Infrastructure | Status Kesehatan Kontainer Monitoring |

### 4. Setup Public Status Page (`/status/public`)
1. Klik **Status Pages** di menu kanan atas $\rightarrow$ **Add New Status Page**.
2. **Title**: `System Status - Kelompok 6`
3. **Slug**: `public`
4. Tambahkan seluruh monitor yang telah dikonfigurasi ke dalam daftar tampilan.
5. Kosongkan *Password Protection* agar halaman dapat diakses secara publik.
6. Klik **Save**.
7. Akses Public URL: `https://kuma-kelompok6.indonesiacentral.cloudapp.azure.com/status/public`

---

## 🤖 Otomatisasi Deployment
[`^ kembali ke atas ^`](#)

Untuk mempercepat instalasi ulang atau *deployment* di server baru, disediakan skrip otomatisasi bash `deploy.sh`:

```bash
#!/bin/bash
# ====================================================================
# Automated Deployment Script for Uptime Kuma & Caddy SSL
# Developed by Kelompok 6 KDKJ - IPB University
# ====================================================================

set -e

DOMAIN="kuma-kelompok6.indonesiacentral.cloudapp.azure.com"

echo "=== [1/4] Updating System Packages ==="
sudo apt-get update && sudo apt-get upgrade -y

echo "=== [2/4] Installing Docker Engine ==="
sudo apt-get install -y docker.io
sudo systemctl enable --now docker

echo "=== [3/4] Deploying Uptime Kuma Container ==="
sudo docker run -d \
  --restart=always \
  -p 127.0.0.1:3001:3001 \
  -v uptime-kuma:/app/data \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name uptime-kuma \
  louislam/uptime-kuma:2

echo "=== [4/4] Installing & Configuring Caddy Reverse Proxy ==="
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf '[https://dl.cloudsmith.io/public/caddy/stable/gpg.key](https://dl.cloudsmith.io/public/caddy/stable/gpg.key)' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf '[https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt](https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt)' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update && sudo apt install caddy -y

echo "$DOMAIN {
    reverse_proxy 127.0.0.1:3001
}" | sudo tee /etc/caddy/Caddyfile

sudo systemctl restart caddy

echo "===================================================================="
echo "Deployment Complete! Access status page at: https://$DOMAIN/status/public"
echo "===================================================================="
```

---

## 🛡️ Maintenance & Operasional Server
[`^ kembali ke atas ^`](#)

### 1. Update Version Aplikasi
```bash
# Stop dan hapus kontainer lama (data tetap aman di volume)
sudo docker stop uptime-kuma && sudo docker rm uptime-kuma

# Pull image terbaru
sudo docker pull louislam/uptime-kuma:2

# Jalankan kembali kontainer
sudo docker run -d --restart=always -p 127.0.0.1:3001:3001 -v uptime-kuma:/app/data -v /var/run/docker.sock:/var/run/docker.sock --name uptime-kuma louislam/uptime-kuma:2
```

### 2. Backup & Restore Database SQLite
 Seluruh data konfigurasi dan riwayat statistik tersimpan di file SQLite `/app/data/kuma.db`.

* **Prosedur Backup**:
  ```bash
  sudo docker cp uptime-kuma:/app/data ./backup_kuma_$(date +%Y%m%d)
  ```
* **Prosedur Restore**:
  ```bash
  sudo docker cp ./backup_kuma_20261021/. uptime-kuma:/app/data
  sudo docker restart uptime-kuma
  ```

---

## 📊 Cara Pemakaian & Hasil Pengujian
[`^ kembali ke atas ^`](#)

Berikut adalah dokumentasi antarmuka dan hasil pengujian pengoperasian Uptime Kuma:

### 1. Halaman Login / Autentikasi Admin
Pengamanan panel administrasi menggunakan autentikasi akun berbasis hash kredensial terenkripsi.

<div align="center">
    <img src="./Dokumentasi/01_login_admin.png" width="850" alt="Login Admin Uptime Kuma" />
    <p><i>Gambar 1: Halaman Antarmuka Login Admin Uptime Kuma</i></p>
</div>

### 2. Dashboard Utama & Quick Stats
Menampilkan statistik *real-time* ketersediaan monitor (*Up, Down, Maintenance*), log aktivitas kejadian (*event log*), serta grafik agregat latensi.

<div align="center">
    <img src="./Dokumentasi/02_dashboard_quickstats.png" width="850" alt="Dashboard Utama" />
    <p><i>Gambar 2: Dashboard Utama, Ringkasan Quick Stats, dan Event Log History</i></p>
</div>

### 3. Formulir Penambahan Target Monitor
Antarmuka pendaftaran target baru yang mendukung penyesuaian interval pengujian (*heartbeat*), retries, timeout, dan header HTTP kustom.

<div align="center">
    <img src="./Dokumentasi/03_form_add_monitor.png" width="850" alt="Add Monitor Form" />
    <p><i>Gambar 3: Formulir Konfigurasi Penambahan Target Monitor Baru</i></p>
</div>

### 4. Detail Pemantauan Transport Layer (TCP Port 443)
Pengujian keandalan port 443 pada IP Cloudflare `1.1.1.1` dengan rata-rata waktu respon 2.09 ms.

<div align="center">
    <img src="./Dokumentasi/04_monitor_tcp_cloudflare.png" width="850" alt="TCP Monitor" />
    <p><i>Gambar 4: Detail Pemantauan Layer Transport TCP Port Cloudflare (1.1.1.1:443)</i></p>
</div>

### 5. Detail Pemantauan Application Layer (DNS Record)
Pengujian resolusi nama domain `google.com` melalui DNS Resolver Cloudflare dengan pengambilan A-Record secara berkala.

<div align="center">
    <img src="./Dokumentasi/05_monitor_dns_cloudflare.png" width="850" alt="DNS Monitor" />
    <p><i>Gambar 5: Detail Pemantauan Resolusi DNS Record google.com</i></p>
</div>

### 6. Detail Pemantauan Application Layer (HTTP GitHub)
Pemantauan ketersediaan HTTPS GitHub beserta pelacakan otomatis masa berlaku Sertifikat SSL Let's Encrypt dan Domain Expiry.

<div align="center">
    <img src="./Dokumentasi/06_monitor_http_github.png" width="850" alt="HTTP GitHub Monitor" />
    <p><i>Gambar 6: Detail Pemantauan Layanan HTTP & SSL Expiry GitHub</i></p>
</div>

### 7. Detail Pemantauan Network Layer (ICMP Ping)
Pengujian latensi dasar jaringan (*Round Trip Time*) ke Google Primary DNS (`8.8.8.8`) dengan respons RTT stabil pada 14 ms.

<div align="center">
    <img src="./Dokumentasi/07_monitor_ping_google.png" width="850" alt="Ping Monitor" />
    <p><i>Gambar 7: Detail Pemantauan ICMP Ping RTT Google DNS (8.8.8.8)</i></p>
</div>

### 8. Detail Pemantauan Web Kampus (Portal IPB University)
Pemantauan ketersediaan layanan web utama IPB University (`https://ipb.ac.id/`) dengan status HTTP `200 - OK` dan pemantauan sertifikat TLS.

<div align="center">
    <img src="./Dokumentasi/08_monitor_http_ipb.png" width="850" alt="IPB Web Monitor" />
    <p><i>Gambar 8: Pemantauan Layanan Web Portal IPB University</i></p>
</div>

### 9. Pemantauan Host-Level (Docker Socket Inspection)
Pemantauan kesehatan kontainer lokal (`uptime-kuma`) secara *native* melalui komunikasi IPC Unix Domain Socket `/var/run/docker.sock`.

<div align="center">
    <img src="./Dokumentasi/09_monitor_docker_socket.png" width="850" alt="Docker Socket Monitor" />
    <p><i>Gambar 9: Detail Pemantauan Health Check Docker Container via Socket</i></p>
</div>

### 10. Manajemen Status Page Publik
Daftar pengelola halaman status yang dipublikasikan ke pengguna luar.

<div align="center">
    <img src="./Dokumentasi/10_status_pages_list.png" width="850" alt="Status Page List" />
    <p><i>Gambar 10: Daftar Pengaturan Halaman Status Publik</i></p>
</div>

### 11. Editor Konfigurasi Status Page
Panel penyuntingan antarmuka Public Status Page dengan slug `/status/public`.

<div align="center">
    <img src="./Dokumentasi/11_status_page_editor.png" width="850" alt="Status Page Editor" />
    <p><i>Gambar 11: Editor Konfigurasi Tampilan Public Status Page (/status/public)</i></p>
</div>

### 12. Panel Pengaturan Sistem
Pengaturan zona waktu (`UTC+07:00 Asia/Jakarta`), Entry Page, dan keamanan global aplikasi.

<div align="center">
    <img src="./Dokumentasi/12_system_settings.png" width="850" alt="System Settings" />
    <p><i>Gambar 12: Panel Pengaturan Sistem dan Global Configurations</i></p>
</div>

---

## 📈 Pembahasan & Analisis Komdat
[`^ kembali ke atas ^`](#)

### 1. Analisis Multi-Layer Protocol Monitoring
Berdasarkan hasil pengujian pada infrastruktur Kelompok 6, pemantauan tidak hanya terbatas pada Layer 7 (HTTP), melainkan mencakup observabilitas komprehensif:
* **Layer 3 (Network - ICMP Ping)**: Berfungsi mendeteksi gangguan konektivitas fisik/routing jaringan mendasar. Latensi Google DNS stabil di angka **14.06 ms**.
* **Layer 4 (Transport - TCP Port)**: Menguji apakah jabat tangan (*three-way handshake*) TCP pada port 443 terbuka tanpa perlu mengunduh konten HTTP. Hasilnya menunjukkan latensi sangat rendah (**2.09 ms**).
* **Layer 7 (Application - HTTP/HTTPS & DNS)**: Menguji kesehatan aplikasi secara penuh. Mengukur waktu *response time* HTTP Portal IPB (**328.38 ms**) dan GitHub (**130.46 ms**), serta memastikan kepatuhan kode status `200 OK`.
* **Host Level (Docker Socket)**: Memastikan proses internal kontainer berstatus `healthy` tanpa beban trafik jaringan eksternal.

### 2. Kelebihan & Kekurangan Implementasi
* **Kelebihan**:
  1. **Zero Cost & High Performance**: Memanfaatkan kredit Azure Student dan perangkat lunak open-source sehingga sistem dapat berjalan tanpa biaya operasional tambahan.
  2. **Automatic SSL/TLS Management**: Penggunaan Caddy menghilangkan risiko sertifikat SSL kadaluarsa pada domain monitoring.
  3. **High Security**: Port internal 3001 diisolasi ke `127.0.0.1` dan seluruh trafik publik dienkripsi dengan HTTPS (TLS 1.3).
  4. **Data Ownership**: Seluruh riwayat latensi dan log gangguan tersimpan aman di database SQLite mandiri.
* **Kekurangan**:
  1. **Single Point of Probing**: Karena di-deploy pada satu instance Azure VPS (Central Indonesia), hasil pengukuran latensi merefleksikan perspektif dari regional lokasi server tersebut.

### 3. Tabel Matriks Perbandingan: Uptime Kuma vs UptimeRobot

| Fitur / Parameter | Uptime Kuma (Deployment Kelompok 6) | UptimeRobot (Free Plan) |
| :--- | :--- | :--- |
| **Biaya Software** | 100% Gratis & Open Source | Freemium |
| **Batas Jumlah Monitor** | **Tanpa Batas** (Sesuai resource VPS) | Maksimal 50 Monitor |
| **Interval Minimal** | **20 Detik** (Gratis) | 5 Menit (Gratis) / 1 Menit (Berbayar) |
| **Dukungan Docker Socket** | **Ya** (Native via Socket Mount) | Tidak Ada |
| **Integrasi Notifikasi** | 90+ Platform Gratis (Telegram, Discord, dll) | Terbatas (SMS & Call Berbayar) |
| **Custom Domain & SSL** | Gratis via Caddy & Azure DNS Label | Membutuhkan Paket Pro Berbayar |
| **Privasi Data** | Terkontrol Sepenuhnya di VPS Sendiri | Tersimpan di Server Pihak Ketiga |

---

## 📌 Kesimpulan
[`^ kembali ke atas ^`](#)

1. Proyek akhir pembuatan sistem pemantauan server dan jaringan menggunakan **Uptime Kuma** pada **Azure Cloud VPS** telah berhasil diimplementasikan 100% sesuai dengan standar industri.
2. Penggunaan **Docker Engine** terbukti mempermudah proses kuarantin (*isolation*) aplikasi, pembaruan versi, serta manajemen ketergantungan sistem.
3. Penggunaan **Caddy Web Server** secara efektif menyelesaikan tantangan enkripsi HTTPS/SSL otomatis dan pengalihan trafik dari port standar 80/443 ke aplikasi Uptime Kuma.
4. Sistem mampu melakukan pemantauan jaringan secara *multi-layer* (ICMP, TCP, HTTP, DNS, dan Docker Socket) secara responsif dengan interval hingga 20 detik.
5. **Status Page Publik** yang dikonfigurasi pada URL `https://kuma-kelompok6.indonesiacentral.cloudapp.azure.com/status/public` dapat diakses dengan lancar tanpa autentikasi oleh pengguna luar untuk mentransparansikan kondisi ketersediaan layanan.

---

## 📚 Referensi
[`^ kembali ke atas ^`](#)

1. **Official Uptime Kuma Repository**: [https://github.com/louislam/uptime-kuma](https://github.com/louislam/uptime-kuma)
2. **Uptime Kuma Official Documentation & Wiki**: [https://github.com/louislam/uptime-kuma/wiki](https://github.com/louislam/uptime-kuma/wiki)
3. **Caddy Web Server Documentation**: [https://caddyserver.com/docs/](https://caddyserver.com/docs/)
4. **Docker Engine Reference Manual**: [https://docs.docker.com/engine/](https://docs.docker.com/engine/)
5. **Microsoft Azure Virtual Machine Documentation**: [https://learn.microsoft.com/en-us/azure/virtual-machines/](https://learn.microsoft.com/en-us/azure/virtual-machines/)
6. **Modul Praktikum Komunikasi Data dan Jaringan Komputer (KDKJ) IPB**: [https://github.com/auriza/komdat-lab](https://github.com/auriza/komdat-lab)
