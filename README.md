


# FileBrowser Quantum + Cloudflare Tunnel (Docker Compose)

Deployment [FileBrowser Quantum](https://github.com/gtsteffaniak/filebrowser) menggunakan Docker Compose, dengan opsi akses dari internet lewat Cloudflare Tunnel. Cocok untuk homelab: clone, isi konfigurasi, jalankan.

## Daftar Isi

- [Developers](#developers)
- [Stack & Versi](#stack--versi)
- [Requirements](#requirements)
- [Struktur Repo](#struktur-repo)
- [Setup](#setup)
- [Run](#run)
- [Configuration](#configuration)
- [Security Warning](#security-warning)
- [Access](#access)
- [Setup Cloudflare Tunnel (Opsional)](#setup-cloudflare-tunnel-opsional)
- [Troubleshooting](#troubleshooting)
- [Lisensi](#lisensi)

## Developers

| Nama | GitHub | Email |
|---|---|---|
| I Putu Budha Aditya | [@budhaaditya12323-boop](https://github.com/budhaaditya12323-boop) | budhaaditya12323@gmail.com |

## Stack & Versi

Proyek ini memakai Docker Compose tanpa Dockerfile kustom, dengan database SQLite yang disimpan langsung di mesin host.

| Komponen | Versi / Tag | Keterangan |
|---|---|---|
| FileBrowser Quantum | `v2.0.1` | Aplikasi utama |
| Docker Image | `gtstef/filebrowser:stable` | Image resmi tanpa Dockerfile kustom |
| Cloudflared | `cloudflare/cloudflared:latest` | Opsional, untuk akses dari internet |
| Docker Compose | v2 (plugin `docker compose`) | Orkestrasi deployment |
| Database | `./db/filebrowser.sqlite` | SQLite, disimpan di mesin host |

> Catatan: tag `stable` bergerak mengikuti rilis terbaru, jadi isinya bisa berbeda dari `v2.0.1` di kemudian hari. Untuk hasil yang konsisten, pin ke versi tertentu di `compose.yaml`.

## Requirements

- **Docker Engine** dan **Docker Compose v2** (perintah `docker compose`, bukan `docker-compose`). Cek dengan:
  ```bash
  docker --version
  docker compose version
  ```
- **Git** untuk clone repo.
- **Akun Cloudflare + domain yang dikelola di Cloudflare** (hanya jika ingin akses dari internet lewat tunnel).

## Struktur Repo

```
.
├── compose.yaml
├── .env.example
├── .gitignore
├── config/
│   └── config.example.yaml
├── data/        # file yang dikelola FileBrowser (isi diabaikan Git)
├── db/          # database SQLite (isi diabaikan Git)
├── cache/       # cache sementara (isi diabaikan Git)
└── README.md
```

Folder `data/`, `db/`, dan `cache/` disediakan lewat `.gitkeep` agar sudah ada saat clone (menghindari folder dibuat oleh root). Isinya tidak ikut Git.

## Setup

1. **Clone repo**
```bash
   git clone https://github.com/budhaaditya12323-boop/personal-cloud-filebrowse
   cd ./personal-cloud-filebrowse
```

2. **Siapkan file dan folder**
```bash
   cp .env.example .env
   cp ./filebrowser/config/config.example.yaml ./filebrowser/config/config.yaml
   mkdir -p ./filebrowser/cache
```
   - `.env` dan `config.yaml` wajib dibuat, walau tidak memakai Cloudflare Tunnel.
   - Jalankan `mkdir` **tanpa `sudo`** dan **sebelum** `docker compose up`. Jika folder belum ada, Docker membuatnya sebagai root dan FileBrowser gagal start dengan error `permission denied`.

3. **Edit `.env`**: isi `CF_TUNNEL_TOKEN` dengan token tunnel, atau biarkan bernilai `-` jika tidak memakai Cloudflare Tunnel.

4. **Edit `filebrowser/config/config.yaml`**: ganti password admin dan sesuaikan pengaturan lain seperlunya (auth, sumber/path, dsb). Bagian yang wajib diubah diberi komentar di file contohnya.

## Run

```bash
# Hanya akses lokal (LAN)
docker compose up -d

# Dengan Cloudflare Tunnel (akses dari internet)
docker compose --profile tunnel up -d
```

Perintah berguna:

```bash
docker compose ps                              # status container
docker compose logs -f                         # lihat log
docker compose pull && docker compose up -d    # update image
docker compose down                            # hentikan (data tetap aman di folder host)
```

## Configuration

### Environment variable (`.env`)

| Variabel | Wajib | Default | Keterangan |
|---|---|---|---|
| `CF_TUNNEL_TOKEN` | Ya (isi `-` jika tanpa tunnel) | `-` | Token Cloudflare Tunnel |

### File konfigurasi (`config/config.yaml`)

| Pengaturan | Wajib | Keterangan |
|---|---|---|
| Password admin | Ya | Ganti placeholder dari `config.example.yaml` sebelum menjalankan pertama kali |
| Pengaturan lain (auth, sumber/path) | Tidak | Sesuaikan kebutuhan. Lihat [dokumentasi FileBrowser Quantum](https://github.com/gtsteffaniak/filebrowser) untuk daftar opsi lengkap |

> Akun admin awal hanya dibuat saat database **pertama kali dibuat**. Mengubah `config.yaml` setelahnya tidak otomatis mengganti password yang sudah tersimpan di `db/`. Ganti lewat antarmuka FileBrowser, atau hapus isi `db/` (menghapus semua akun) untuk memulai dari awal.

### Volume

| Path host | Path di container | Fungsi |
|---|---|---|
| `./data` | `/srv` | File yang dikelola FileBrowser |
| `./config` | `/config` | `config.yaml` |
| `./db` | `/home/filebrowser/data` | Database SQLite |
| `./cache` | `/tmp/filebrowser` | Cache sementara |

### Variabel yang tetap di `compose.yaml`

`FILEBROWSER_CONFIG` dan `FILEBROWSER_DATABASE_PATH` adalah path di dalam container yang terikat dengan bagian `volumes`. Tidak perlu diubah.

## Security Warning

> **Jangan gunakan password default atau lemah, terutama jika aplikasi diakses dari internet.**

- Ganti password admin di `config/config.yaml` sebelum menjalankan pertama kali. Jika placeholder dibiarkan, siapa pun yang membaca repo ini tahu kredensialnya.
- Jangan commit `.env` dan `config/config.yaml`. Keduanya sudah ada di `.gitignore`.
- Jika memakai Cloudflare Tunnel, aplikasi bisa diakses siapa saja yang tahu domainnya. Pertimbangkan [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/policies/access/) sebagai lapisan login tambahan.
- Jika file database pernah ter-commit ke Git, anggap kredensial di dalamnya bocor dan ganti semua password.

## Access

| Cara | Alamat | Keterangan |
|---|---|---|
| Di mesin yang sama | `http://localhost:8080` | Selalu bisa |
| LAN | `http://<IP-server>:8080` | Port `8080` terbuka di jaringan lokal |
| Internet | `https://<subdomain-kamu>` | Lewat Cloudflare Tunnel, lihat bagian di bawah |

**Hardening opsional:** jika hanya mengakses lewat tunnel, ubah port di `compose.yaml` menjadi `127.0.0.1:8080:80` agar tidak terbuka di LAN.

## Setup Cloudflare Tunnel (Opsional)

Lewati bagian ini jika hanya butuh akses lokal.

1. Siapkan akun Cloudflare dan domain yang sudah dikelola di Cloudflare.
2. Buka dashboard **Cloudflare Zero Trust**, buat tunnel baru bertipe **Cloudflared**, lalu salin **token**-nya. Menu bisa berubah, ikuti [dokumentasi resmi](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/).
3. Tambahkan **Public Hostname** pada tunnel:
   - Subdomain/domain: sesuai keinginan, misalnya `files.domainmu.com`
   - Service type: `HTTP`
   - URL: `filebrowser:80`

   Gunakan nama service dan port **di dalam jaringan Docker**, bukan `localhost:8080`.
4. Ganti nilai `CF_TUNNEL_TOKEN` di `.env` dengan token tersebut.
5. Jalankan dengan profil tunnel:
   ```bash
   docker compose --profile tunnel up -d
   ```

## Troubleshooting

**`docker compose up` gagal karena variabel belum diisi**
Pastikan file `.env` sudah dibuat dari `.env.example` (`cp .env.example .env`). Tanpa file `.env`, `CF_TUNNEL_TOKEN` tidak terdefinisi dan Compose berhenti dengan error.

**Container filebrowser gagal start atau tidak menemukan konfigurasi**
Pastikan `config/config.yaml` sudah dibuat (`cp config/config.example.yaml config/config.yaml`) dan sintaks YAML-nya valid. Cek detail dengan `docker compose logs filebrowser`.

**Error `permission denied` pada `data/`, `db/`, atau `config/`**
Container berjalan dengan user yang berbeda dari pemilik folder. Sesuaikan kepemilikan folder dengan user di container (`sudo chown -R <uid>:<gid> data db config cache`), atau tambahkan `user: "1000:1000"` pada service filebrowser.

**Port `8080` sudah dipakai**
Ubah sisi kiri pemetaan port di `compose.yaml`, misalnya `"8081:80"`.

**Container `cloudflared` restart terus**
Biasanya token kosong, salah, atau masih bernilai `-`. Periksa `CF_TUNNEL_TOKEN` di `.env` dan lihat log dengan `docker compose logs cloudflared`.

**Domain terbuka tapi muncul error 502 / Bad Gateway**
Public Hostname di dashboard Cloudflare belum mengarah ke `filebrowser:80`, atau container filebrowser belum berjalan. Cek `docker compose ps`.

**Password di `config.yaml` sudah diubah tapi login masih memakai password lama**
Akun admin awal hanya dibuat saat database pertama kali dibuat. Ganti password lewat antarmuka FileBrowser, atau hapus isi `db/` untuk mereset (semua akun ikut terhapus).

**Setelah update, aplikasi bermasalah**
Kembali ke tag versi sebelumnya di `compose.yaml`, lalu jalankan `docker compose up -d`. Backup folder `db/` dan `config/` sebelum update besar.

## Lisensi

- [FileBrowser Quantum](https://github.com/gtsteffaniak/filebrowser): Apache License 2.0

