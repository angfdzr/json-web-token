# 🔐 REST API Autentikasi dengan JSON Web Token (JWT)

REST API autentikasi berbasis **Node.js**, **Express**, dan **Sequelize (MySQL)** yang memakai **JSON Web Token (JWT)** untuk mengamankan akses pengguna. Kata sandi disimpan dalam bentuk *hash* dengan **bcrypt**, dan token yang diterbitkan dicatat di database sehingga dapat dicabut (*revoke*).

---

## ✨ Fitur

- **Autentikasi pengguna** dengan username dan kata sandi.
- **Penerbitan JWT** (algoritma HS256) yang berisi ID pengguna dan masa berlaku token.
- **Penyimpanan kata sandi aman** menggunakan *hashing* bcrypt.
- **Pencatatan token di database** beserta status `revoked`, sehingga token dapat dinonaktifkan (misalnya saat *logout*).
- **Relasi antartabel**: setiap token terhubung ke pengguna melalui `UserId`.
- **Konfigurasi lewat environment variable** menggunakan `dotenv`.

## 🔄 Alur Autentikasi

```
Klien                         API                              MySQL
  │  1. kirim username+password │                                 │
  │ ───────────────────────────►│  2. cari user, cocokkan hash    │
  │                             │ ───────────────────────────────►│
  │                             │  3. buat JWT (payload: id)      │
  │                             │  4. simpan token (revoked = 0)  │
  │                             │ ───────────────────────────────►│
  │  5. terima JWT              │                                 │
  │ ◄───────────────────────────│                                 │
  │  6. akses data dengan JWT   │  7. verifikasi JWT & status     │
  │ ───────────────────────────►│     revoked pada tabel tokens   │
```

Contoh payload JWT yang diterbitkan (hasil dekode token pada data contoh):

```json
{
  "id": 1,
  "iat": 1733738819,
  "exp": 1733742419
}
```

Token berlaku selama **1 jam** (selisih `exp` dan `iat` = 3600 detik).

## 🗄️ Skema Database

Database: `jwt_db` (file dump: `jwt_db.sql`)

**Tabel `users`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | INT, PK, auto increment | ID pengguna |
| `username` | VARCHAR(255), unik | Nama pengguna |
| `password` | VARCHAR(255) | Kata sandi ter-*hash* (bcrypt) |
| `createdAt` | DATETIME | Waktu dibuat |
| `updatedAt` | DATETIME | Waktu diperbarui |

**Tabel `tokens`**

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | INT, PK, auto increment | ID token |
| `token` | VARCHAR(255) | String JWT |
| `revoked` | TINYINT(1), default 0 | Status pencabutan token |
| `createdAt` | DATETIME | Waktu dibuat |
| `updatedAt` | DATETIME | Waktu diperbarui |
| `UserId` | INT, FK → `users.id` | Pemilik token (`ON DELETE SET NULL`, `ON UPDATE CASCADE`) |

## 📁 Struktur Proyek

```
JSON-Web-Token/
├── config/            # Konfigurasi aplikasi / koneksi database
├── models/            # Model Sequelize (User, Token)
├── routes/            # Definisi rute API
├── app.js             # Titik masuk aplikasi Express
├── jwt_db.sql         # Dump database MySQL (struktur + data contoh)
├── package.json
└── package-lock.json
```

## 🛠️ Teknologi yang Digunakan

| Paket | Fungsi |
|---|---|
| **express** | Framework web |
| **jsonwebtoken** | Membuat dan memverifikasi JWT |
| **bcryptjs** | *Hashing* kata sandi |
| **sequelize** | ORM |
| **mysql2** | Driver MySQL |
| **body-parser** | Membaca *body* request |
| **dotenv** | Membaca variabel dari file `.env` |

## 🚀 Cara Menjalankan

### 1. Clone repository

```bash
git clone https://github.com/angfdzr/JSON-Web-Token.git
cd JSON-Web-Token
```

### 2. Install dependensi

```bash
npm install
```

### 3. Siapkan database

1. Jalankan MySQL/MariaDB (mis. lewat XAMPP atau Laragon).
2. Buat database `jwt_db`.
3. Impor file `jwt_db.sql`:

```bash
mysql -u root -p jwt_db < jwt_db.sql
```

Atau impor lewat phpMyAdmin.

### 4. Buat file `.env`

Buat file `.env` di folder utama berisi konfigurasi yang dibaca aplikasi, seperti *secret key* JWT dan pengaturan koneksi database. Sesuaikan nama variabelnya dengan yang dipakai di folder `config/` dan `app.js`.

> ⚠️ Jangan meng-*commit* file `.env` ke GitHub. Tambahkan ke `.gitignore`.

### 5. Jalankan aplikasi

```bash
node app.js
```

## 🔒 Catatan Keamanan

- Simpan *secret key* JWT di `.env`, bukan di dalam kode.
- Selalu gunakan HTTPS saat aplikasi dipublikasikan.
- Kata sandi tidak pernah disimpan sebagai teks biasa, hanya sebagai *hash* bcrypt.
- Periksa kolom `revoked` pada tabel `tokens` saat memverifikasi token agar token yang sudah dicabut ditolak.
- Berkas `jwt_db.sql` berisi data contoh (pengguna dan token). Hapus data tersebut sebelum dipakai di lingkungan produksi.

## 👤 Penulis

**Angga Fadzar**

---

*Proyek ini dibuat untuk keperluan pembelajaran.*
