# Campus Polling System

**Mata Kuliah:** Cloud Computing – Telkom University (UTS)  
**Studi Kasus:** #7 – Sistem Polling / Voting Kampus

## Anggota Kelompok

| Nama | NIM |
|---|---|
| Renizar Wendari | 101012500073 |
| Ghania Fitriasy Syifa | 101012580015 |
| Habib Abdurrahman | 101012580024 |
| Bagus Bintang Fauzi | 101012580026 |

## 1. Deskripsi Project

Campus Polling System merupakan aplikasi web sederhana untuk melakukan voting terhadap dua pilihan, yaitu Opsi A dan Opsi B.

Aplikasi dibangun menggunakan Python sebagai backend REST API, HTML dan Vanilla JavaScript sebagai frontend, serta Nginx sebagai web server dan reverse proxy.

Seluruh layanan dijalankan menggunakan Docker Compose dengan Custom Bridge Network. Penyimpanan data menggunakan Docker Named Volume agar data suara tetap tersimpan meskipun container backend dibuat ulang.

Proses Continuous Integration dan Continuous Deployment (CI/CD) diotomatisasi menggunakan Jenkins yang terintegrasi dengan GitHub Webhook.

**Repository GitHub:**  
https://github.com/renizar/campus-polling-system

## 2. Teknologi yang Digunakan

| Teknologi | Fungsi |
|---|---|
| Python | Backend REST API |
| HTML | Struktur halaman frontend |
| Vanilla JavaScript | Interaksi dan komunikasi dengan API |
| Nginx | Web server dan reverse proxy |
| Docker | Containerization aplikasi |
| Docker Compose | Mengelola beberapa container |
| Docker Bridge Network | Komunikasi antar-container |
| Docker Named Volume | Penyimpanan data persisten |
| GitHub | Version control dan repository |
| Jenkins | Otomatisasi CI/CD |
| GitHub Webhook | Memicu pipeline saat ada push |

## 3. Diagram Arsitektur

```text
                    USER / BROWSER
                          |
                          v
                  localhost:8083
                          |
                          v
        +-------------------------------------+
        |       DOCKER NETWORK: app-net       |
        |                                     |
        |  +-------------------------------+  |
        |  | FRONTEND                      |  |
        |  | Nginx (nginx:alpine)          |  |
        |  |                               |  |
        |  | index.html + JavaScript       |  |
        |  |                               |  |
        |  | /api/* -> backend:5000        |  |
        |  +---------------+---------------+  |
        |                  |                  |
        |                  v                  |
        |  +-------------------------------+  |
        |  | BACKEND                       |  |
        |  | Python REST API               |  |
        |  | app.py                        |  |
        |  | Port 5000                     |  |
        |  +---------------+---------------+  |
        |                  |                  |
        +------------------|------------------+
                           |
                           v
                 DOCKER NAMED VOLUME
                    polling-db-data
                           |
                           v
              /var/lib/backend_data/votes.json
```

### Komponen Infrastruktur

- **Frontend:** Nginx menyajikan halaman web dari folder `frontend`.
- **Reverse Proxy:** Nginx meneruskan request `/api/` ke backend menggunakan DNS service `backend`.
- **Backend:** Python menangani operasi GET, POST, dan DELETE pada data voting.
- **Custom Bridge Network:** Menghubungkan frontend dan backend melalui jaringan `app-net`.
- **Named Volume:** Menyimpan file `votes.json` agar data tidak hilang ketika container dibuat ulang.
- **Bind Mount:** Menghubungkan folder frontend dan konfigurasi Nginx dari host ke container sehingga perubahan file dapat langsung diterapkan tanpa rebuild image.

## 4. Struktur Folder

```text
campus-polling-system/
├── backend/
│   ├── app.py
│   └── Dockerfile
├── frontend/
│   └── index.html
├── nginx/
│   └── default.conf
├── docker-compose.yml
├── Jenkinsfile
├── README.md
└── .gitignore
```

## 5. REST API

Backend menyediakan tiga endpoint utama:

| Method | Endpoint Backend | Fungsi |
|---|---|---|
| GET | `/votes` | Mengambil jumlah suara, hasil voting, dan daftar suara |
| POST | `/votes` | Menambahkan suara baru |
| DELETE | `/votes?index=X` | Menghapus catatan suara berdasarkan index |

### Contoh Request

**POST – Menambahkan suara**

```json
{
  "option": "A"
}
```

Pilihan yang valid adalah `A` atau `B`.

**GET – Mengambil hasil voting**

Mengembalikan total suara, jumlah suara masing-masing pilihan, dan daftar catatan voting.

**DELETE – Menghapus suara**

Contoh:

```text
DELETE /votes?index=0
```

### Validasi API

- HTTP 400: Request atau parameter tidak valid.
- HTTP 404: Data yang diminta tidak ditemukan.

Frontend mengakses API melalui Nginx dengan prefix `/api/`, misalnya `/api/votes`. Nginx meneruskan request tersebut ke endpoint backend `/votes`.

## 6. Instalasi dan Menjalankan Aplikasi

### Persyaratan

- Docker Desktop atau Docker Engine.
- Docker Compose.
- Git.

### Langkah Instalasi

Clone repository:

```bash
git clone https://github.com/renizar/campus-polling-system.git
```

Masuk ke folder project:

```bash
cd campus-polling-system
```

Build dan jalankan aplikasi:

```bash
docker compose up -d --build
```

Periksa status container:

```bash
docker compose ps
```

Buka aplikasi melalui browser:

```text
http://localhost:8083
```

### Menghentikan Aplikasi

```bash
docker compose down
```

Perintah tersebut menghentikan dan menghapus container serta network project, tetapi tidak menghapus Named Volume.

## 7. Pengujian Persistensi Data

Pengujian dilakukan untuk memastikan data voting tetap tersimpan meskipun container backend dihapus dan dibuat kembali.

Hapus container backend:

```bash
docker rm -f polling-backend
```

Jalankan kembali backend:

```bash
docker compose up -d backend
```

Periksa kembali aplikasi melalui browser.

**Hasil pengujian:** Data voting tetap tersedia setelah container backend dibuat ulang karena data disimpan menggunakan Docker Named Volume `polling-db-data`.

## 8. Implementasi CI/CD dengan Jenkins

Project menggunakan Jenkins Declarative Pipeline yang terintegrasi dengan GitHub Webhook.

Alur otomatisasi:

```text
Developer
    |
    v
Git Push ke GitHub (Branch main)
    |
    v
GitHub Webhook
    |
    v
Jenkins Pipeline
    |
    v
Checkout
    |
    v
Syntax Check
    |
    v
Build Image
    |
    v
Automated Deployment
    |
    v
Health Check
```

### Tahapan Jenkins Pipeline

| Stage | Fungsi |
|---|---|
| Checkout | Mengambil source code dari repository GitHub |
| Syntax Check | Memeriksa sintaks Python dan konfigurasi Docker Compose |
| Build Image | Membangun Docker image aplikasi |
| Automated Deployment | Menjalankan atau memperbarui container menggunakan Docker Compose |
| Health Check | Memastikan backend dapat diakses melalui jaringan Docker |

### Konfigurasi Jenkins

1. Jalankan Jenkins dan pastikan plugin Git, GitHub, dan Pipeline tersedia.
2. Buat job baru dengan tipe Pipeline.
3. Pilih **Pipeline script from SCM**.
4. Pilih Git sebagai SCM.
5. Masukkan repository URL:

   `https://github.com/renizar/campus-polling-system.git`

6. Gunakan branch `*/main`.
7. Isi Script Path dengan `Jenkinsfile`.
8. Aktifkan **GitHub hook trigger for GITScm polling**.
9. Konfigurasikan GitHub Webhook dengan URL Jenkins yang dapat diakses dari internet dan path `/github-webhook/`.
10. Gunakan Content type `application/json` dan pilih event Push.

### Hasil Pengujian CI/CD

Pengujian dilakukan dengan melakukan push perubahan ke branch `main`.

Jenkins berhasil menjalankan seluruh lima tahapan pipeline:

- Checkout: Success
- Syntax Check: Success
- Build Image: Success
- Automated Deployment: Success
- Health Check: Success

Dengan demikian, perubahan pada repository GitHub dapat memicu proses build, deployment, dan pemeriksaan kesehatan aplikasi secara otomatis.

## 9. Video Demo

Tautan video demonstrasi aplikasi dan implementasi CI/CD:

**[Tambahkan tautan video demo 5–8 menit di sini]**

## 10. Kesimpulan

Campus Polling System berhasil diimplementasikan sebagai aplikasi web berbasis container dengan arsitektur frontend dan backend yang terpisah.

Docker Compose digunakan untuk mengelola layanan, Custom Bridge Network untuk komunikasi antar-container, Named Volume untuk menjaga persistensi data, serta Bind Mount untuk memudahkan perubahan file frontend.

Integrasi GitHub Webhook dengan Jenkins juga berhasil menjalankan lima tahapan CI/CD secara otomatis hingga Health Check berhasil.