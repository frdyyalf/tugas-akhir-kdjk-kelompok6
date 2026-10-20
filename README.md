<div align="center">
    <img src="https://raw.githubusercontent.com/louislam/uptime-kuma/master/public/icon.svg" width="128" alt="Uptime Kuma Logo" />
    <h1>Laporan Proyek Akhir KDKJ: Uptime Kuma</h1>
    <p><b>Aplikasi Web Self-Hosted Monitoring Server dan Jaringan</b></p>
</div>

<hr />

[Sekilas Tentang](#sekilas-tentang) | [Instalasi](#instalasi) | [Konfigurasi](#konfigurasi) | [Maintenance](#maintenance) | [Otomatisasi](#otomatisasi) | [Cara Pemakaian](#cara-pemakaian) | [Pembahasan](#pembahasan) | [Referensi](#referensi)

---

# Sekilas Tentang
[`^ kembali ke atas ^`](#)

**Uptime Kuma** adalah aplikasi pemantauan (*monitoring tool*) jaringan dan server open-source berbasis web yang dikembangkan oleh Louis Lam. Aplikasi ini dirancang sebagai alternatif mandiri (*self-hosted*) untuk layanan komersial seperti UptimeRobot.

Uptime Kuma memungkinkan administrator jaringan dan pengembang sistem untuk memantau ketersediaan (*uptime/downtime*), waktu respon (*latency*), status sertifikat SSL/TLS, serta kesehatan berbagai protokol jaringan (HTTP/HTTPS, TCP, Ping, DNS, WebSocket, Docker Container, dan lain-lain) secara *real-time* melalui antarmuka yang modern, reaktif, dan intuitif.

### Fitur Utama:
- **Dukungan Banyak Protokol**: Memantau HTTP(s), Ping, TCP Port, DNS Record, Keyword HTTP, JSON Query, WebSocket, Steam Game Server, hingga Docker Container.
- **Interval Pemantauan Cepat**: Pemantauan frekuensi tinggi hingga interval 20 detik.
- **Notifikasi Multi-Channel**: Terintegrasi dengan lebih dari 90+ layanan notifikasi seperti Telegram, Discord, Slack, Email (SMTP), Gotify, Pushover, dan Webhook.
- **Status Page Terkustomisasi**: Membuat halaman status publik yang terpisah untuk menginformasikan kondisi infrastruktur kepada pengguna akhir (*end-user*).
- **Pemantauan Sertifikat SSL/TLS**: Peringatan otomatis sebelum sertifikat keamanan domain kadaluarsa.
- **Keamanan Lengkap**: Mendukung *Two-Factor Authentication* (2FA) dan konfigurasi *Proxy*.

---

# Instalasi
[`^ kembali ke atas ^`](#)

Panduan instalasi ini dirancang secara umum sehingga dapat diterapkan di berbagai lingkungan, baik pada **Virtual Machine (VM) lokal** (VirtualBox/VMware), **VPS Cloud** (Azure, AWS, DigitalOcean), maupun **Server Bare Metal** berbasis Linux.

#### Kebutuhan Sistem Minimum:
- **Sistem Operasi**: Linux Major Distros (Ubuntu 20.04/22.04 LTS, Debian, CentOS, atau Arch Linux).
- **CPU**: Minimal 1 Core.
- **RAM**: Minimal 1 GB.
- **Disk Space**: Minimal 5 GB.

---

### Opsi 1: Instalasi via Docker (Sangat Direkomendasikan)

Metode kontainerisasi menggunakan Docker adalah cara paling cepat, bersih, dan mudah di-maintain.

1. **Pasang Docker Engine di Server**:
```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y docker.io
sudo systemctl enable --now docker
```

2. **Jalankan Container Uptime Kuma**:
```bash
sudo docker run -d \
  --restart=always \
  -p 3001:3001 \
  -v uptime-kuma:/app/data \
  --name uptime-kuma \
  louislam/uptime-kuma:2
```
*Keterangan Parameter:*
- `-d`: Menjalankan kontainer di latar belakang (*detached mode*).
- `--restart=always`: Kontainer otomatis menyala kembali saat server/VM booting.
- `-p 3001:3001`: Meneruskan port 3001 dari host server ke port 3001 kontainer.
- `-v uptime-kuma:/app/data`: Memetakan *volume* persisten agar data pemantauan tidak hilang saat kontainer diperbarui.

---

### Opsi 2: Instalasi Non-Docker (Node.js & PM2)

Jika server tidak mendukung Docker, Uptime Kuma dapat dijalankan langsung menggunakan Node.js dan PM2 Process Manager.

Prasyarat: Node.js >= 20.4, Git, dan PM2.

```bash
# 1. Clone repositori resmi
git clone [https://github.com/louislam/uptime-kuma.git](https://github.com/louislam/uptime-kuma.git)
cd uptime-kuma

# 2. Jalankan setup lingkungan
npm run setup

# 3. Install PM2 dan jalankan server di latar belakang
npm install pm2 -g && pm2 install pm2-logrotate
pm2 start server/server.js --name uptime-kuma
pm2 startup && pm2 save
```

---

### Opsi Deployment Akses Publik (Internet)

Agar Uptime Kuma dapat diakses dari internet secara global, terdapat dua opsi umum:
1. **VPS Cloud Publik (Berbayar / Kredit Mahasiswa)**:
   Mendeploy di cloud provider (seperti Azure for Students, DigitalOcean, atau AWS) dengan mengalokasikan *Public IP* statis dan membuka port `3001` pada firewall/Network Security Group.
2. **Cloudflare Tunnel (Gratis & Tanpa Port Forwarding)**:
   Jika aplikasi di-deploy di VM lokal, gunakan Cloudflare Tunnel untuk mengekspos port local 3001 ke URL HTTPS publik secara gratis:
```bash
sudo docker run -d --network=host --name cloudflared cloudflare/cloudflared:latest tunnel --url http://localhost:3001
```

---

# Konfigurasi
[`^ kembali ke atas ^`](#)

### 1. Konfigurasi Notifikasi (Contoh: Telegram Bot)
Agar administrator langsung mendapatkan peringatan otomatis saat layanan mengalami gangguan (*down*):
1. Masuk ke dashboard Uptime Kuma -> Pilih **Pengaturan** (*Settings*) -> **Notifikasi**.
2. Klik **Setel Notifikasi**.
3. Pilih **Telegram** sebagai *Notification Type*.
4. Masukkan **Bot Token** (diperoleh dari `@BotFather`) dan **Chat ID** tujuan.
5. Klik **Tes** untuk menguji pengiriman pesan, lalu klik **Simpan**.

### 2. Konfigurasi Reverse Proxy & Nginx (Opsional)
Untuk penggunaan domain kustom (misal: `https://monitoring.domain.com`), disarankan menggunakan Nginx sebagai *Reverse Proxy*:

```nginx
server {
    listen 80;
    server_name monitoring.domainkamu.com;

    location / {
        proxy_pass [http://127.0.0.1:3001](http://127.0.0.1:3001);
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

---

# Maintenance
[`^ kembali ke atas ^`](#)

### 1. Pembaruan Versi Aplikasi (Update)
Pembaruan aplikasi versi Docker dapat dilakukan tanpa merusak data pemantauan:

```bash
# Hentikan kontainer lama
sudo docker stop uptime-kuma && sudo docker rm uptime-kuma

# Unduh image versi terbaru
sudo docker pull louislam/uptime-kuma:2

# Jalankan kembali kontainer dengan volume data yang sama
sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:2
```

### 2. Cadangan Data (Backup & Restore)
Seluruh konfigurasi dan riwayat statistik disimpan dalam database SQLite di dalam volume `uptime-kuma`.

* **Backup Data**:
```bash
sudo docker cp uptime-kuma:/app/data ./backup_kuma_data
```
* **Restore Data**:
```bash
sudo docker cp ./backup_kuma_data/. uptime-kuma:/app/data
sudo docker restart uptime-kuma
```

---

# Otomatisasi
[`^ kembali ke atas ^`](#)

Pengguna dapat memanfaatkan *Shell Script* sederhana untuk mengotomatiskan seluruh proses instalasi Docker dan deployment Uptime Kuma dalam satu langkah:

File `deploy.sh`:
```bash
#!/bin/bash
echo "=== Memulai Otomatisasi Deployment Uptime Kuma ==="
sudo apt-get update -y
sudo apt-get install -y docker.io
sudo systemctl enable --now docker

sudo docker run -d \
  --restart=always \
  -p 3001:3001 \
  -v uptime-kuma:/app/data \
  --name uptime-kuma \
  louislam/uptime-kuma:2

echo "=== Deployment Selesai! Akses aplikasi pada http://localhost:3001 ==="
```

---

# Cara Pemakaian
[`^ kembali ke atas ^`](#)

Berikut adalah panduan antarmuka dan penggunaan fitur-fitur utama Uptime Kuma berdasarkan hasil pengujian:

### 1. Halaman Login / Autentikasi
Saat diakses pertama kali, sistem meminta pembuatan akun **Admin** utama untuk mengamankan panel *dashboard*.

![Halaman Login Admin](./Dokumentasi/Screenshot%202026-10-19%20213107.png)

### 2. Dashboard Pemantauan Layanan Spasial (Detail Monitor)
Panel ini menampilkan grafik latensi (*response time*) secara *real-time*, waktu aktif (*uptime percentage*), status sertifikat SSL, serta opsi untuk menjeda, mengubah, atau menghapus monitor.

* Contoh Pemantauan Layanan HTTP (`Github Service`):
  ![Detail Monitor HTTP](./Dokumentasi/Screenshot%202026-10-19%20214032.png)

* Contoh Pemantauan Ping Network (`Google DNS`):
  ![Detail Monitor Ping](./Dokumentasi/Screenshot%202026-10-19%20214042.png)

* Contoh Pemantauan Layanan Web Kampus (`Portal IPB University`):
  ![Detail Monitor Web Kampus](./Dokumentasi/Screenshot%202026-10-19%20214052.png)

### 3. Log Statistik & Riwayat Kejadian (Incident History)
Halaman statistik merangkum total monitor yang Aktif, Tidak Aktif, maupun Pemeliharaan, beserta tabel log kejadian (*event log*) yang mencatat respon server (misal: `200 - OK` atau `ECONNREFUSED`).

![Log Statistik Sistem](./Dokumentasi/Screenshot%202026-10-19%20214006.png)

### 4. Menambahkan Pemantauan Baru (Add New Monitor)
Formulir untuk mendaftarkan target pemantauan baru. Pengguna dapat memilih tipe monitor (HTTP/Ping/DNS/TCP), memasukkan URL/IP, serta menentukan interval pemeriksaan (*heartbeat interval*).

![Tambah Monitor Baru](./Dokumentasi/Screenshot%202026-10-19%20213845.png)

### 5. Manajemen & Tampilan Halaman Status Publik (Status Page)
Uptime Kuma menyediakan fitur *Status Page* publik yang memungkinkan pengguna luar melihat kondisi kesehatan infrastruktur tanpa perlu login.

* **Daftar Halaman Status**:
  ![Daftar Status Page](./Dokumentasi/Screenshot%202026-10-19%20213756.png)

* **Tampilan Status Page (`/status/compare`)**:
  ![Tampilan Publik Status Page](./Dokumentasi/Screenshot%202026-10-19%20214128.png)

* **Pengaturan & Edit Status Page**:
  ![Edit Status Page](./Dokumentasi/Screenshot%202026-10-19%20214250.png)

### 6. Panel Pengaturan Sistem (Settings)
Menu konfigurasi umum untuk mengatur Zona Waktu (*Timezone*), Notifikasi, Proxy Terbalik, Keamanan (2FA), serta Kunci API Steam.

![Pengaturan Sistem](./Dokumentasi/Screenshot%202026-10-19%20214354.png)

---

# Pembahasan
[`^ kembali ke atas ^`](#)

### Kelebihan Uptime Kuma:
1. **Ringan dan Sangat Cepat**: Dibangun menggunakan Node.js, Vue 3, dan Bootstrap 5 dengan komunikasi WebSocket (Socket.io), sehingga antarmuka sangat responsif tanpa perlu *refresh* halaman manual.
2. **Gratis dan Tanpa Batasan (100% Open Source)**: Tidak ada batasan jumlah monitor atau frekuensi pemantauan (interval bisa diatur hingga 20 detik tanpa biaya).
3. **Integrasi Notifikasi Sangat Luas**: Mendukung 90+ platform notifikasi populer secara *native*.
4. **Desain UI/UX Modern**: Tampilan dashboard sangat bersih, intuitif, serta mendukung mode gelap (*Dark Mode*).

### Kekurangan Uptime Kuma:
1. **Penyimpanan Skala Kecil**: Secara *default* menggunakan SQLite. Sangat efisien untuk pemantauan ratusan target, namun kurang cocok untuk mengelola puluhan ribu monitor secara bersamaan pada infrastruktur enterprise raksasa.
2. **Belum Mendukung Multi-Node Native**: Pemantauan dikirim dari satu lokasi server tempat Uptime Kuma di-deploy (tidak memiliki probe pemantauan terdistribusi dari berbagai benua secara bawaan seperti layanan berbayar).

---

### Perbandingan Aplikasi Sejenis: **Uptime Kuma vs UptimeRobot**

| Fitur / Parameter | Uptime Kuma (Self-Hosted) | UptimeRobot (SaaS/Komersial) |
| :--- | :--- | :--- |
| **Model Biaya** | Gratis sepenuhnya (*Open Source*) | Freemium (Berbayar untuk fitur lengkap) |
| **Batas Jumlah Monitor** | **Tidak Terbatas** (Tergantung resource server) | Gratis maks. 50 monitor (Selebihnya berbayar) |
| **Interval Pemantauan Min.**| **Hingga 20 Detik** (Gratis) | 5 Menit (Gratis) / 1 Menit (Berbayar) |
| **Privasi Data** | Data tersimpan mandiri di server sendiri | Data disimpan di server pihak ketiga |
| **Lokasi Pemantauan** | Single-location (Lokasi server tempat deploy) | Multi-location (Probe dari berbagai negara) |
| **Saluran Notifikasi** | 90+ Integrasi gratis (Telegram, Discord, dll) | Terbatas di versi gratis (SMS/Call berbayar) |
| **Kemudahan Setup** | Perlu instalasi Docker/Server sederhana | Langsung pakai (Daftar akun di web) |

---

# Referensi
[`^ kembali ke atas ^`](#)

1. **GitHub Repository Official**: [Louislam / Uptime Kuma](https://github.com/louislam/uptime-kuma)
2. **Dokumentasi Resmi & Wiki**: [Uptime Kuma Wiki](https://github.com/louislam/uptime-kuma/wiki)
3. **Docker Hub**: [Official Image Uptime Kuma](https://hub.docker.com/r/louislam/uptime-kuma)
4. **Dokumentasi Kurikulum KDKJ IPB**: [Projek Akhir Komdat-Lab](https://github.com/auriza/komdat-lab/blob/master/projek.md)
