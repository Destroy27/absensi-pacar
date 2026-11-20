# 💗 Absensi Harian — Website Absen Sederhana

Aplikasi web absensi harian berbasis **Firebase Firestore** (tanpa server sendiri),
siap di-hosting gratis di **GitHub Pages**.

![Stack](https://img.shields.io/badge/Stack-HTML%20%2B%20CSS%20%2B%20JS-yellow) ![DB](https://img.shields.io/badge/DB-Firebase%20Firestore-orange) ![Host](https://img.shields.io/badge/Host-GitHub%20Pages-blue) ![License](https://img.shields.io/badge/License-MIT-green)

---

## ✨ Fitur

### 3 Halaman Utama

| Halaman | File | Fungsi |
|---|---|---|
| **Absen** | `index.html` | Form presensi harian |
| **Data** | `data.html` | Lihat, cari, filter, cetak & export data |
| **Admin** | `admin.html` | Kelola anggota, sesi & pengaturan |

### Halaman Absen
- Pilih **nama** dari daftar anggota
- **Sesi absen** punya jam buka & tutup sendiri
- **Countdown live** sisa waktu absen (atau waktu menuju sesi dibuka)
- 4 status: **Hadir**, **Sakit**, **Izin**, **Dispen** — alasan bisa diwajibkan lewat pengaturan
- Absen **sekali per sesi**; kirim ulang untuk mengubah jawaban
- Daftar "sudah absen" hari ini **real-time** (Firestore `onSnapshot`)

### Halaman Data
- Rekap seluruh absensi dalam satu tabel
- Filter berdasarkan **sesi**, **status**, dan **pencarian** nama/alasan
- Statistik: total, hadir, sakit, izin, dispen, dan **persentase kehadiran**
- **Export CSV** (siap dibuka di Excel) dan **cetak / simpan PDF**
- Hapus satu baris data bila salah input

### Halaman Admin
- Masuk dengan **PIN admin** (PIN awal `123456`, bisa diganti)
- **Kelola anggota**: tambah, edit, hapus (nama, panggilan, emoji, aktif/nonaktif)
- **Kelola sesi**: judul, tanggal, jam buka, jam tutup
- **Pengaturan**: judul aplikasi, judul halaman absen, teks sambutan, wajib alasan, ganti PIN

---

## 🧱 Teknologi

| Bagian | Dipakai |
|---|---|
| Tampilan | HTML + CSS + JavaScript (vanilla, tanpa build) |
| Database | Firebase Firestore (Spark / gratis) |
| SDK | Firebase Web SDK v10 (modular, via CDN) |
| Hosting | GitHub Pages |

Tanpa `npm install`, tanpa bundler — cukup buka filenya.

---

## 📁 Struktur Folder

```
absensi-pacar/
├── index.html                  # Halaman absen
├── data.html                   # Halaman rekap data
├── admin.html                  # Halaman admin
├── assets/
│   ├── css/
│   │   └── style.css           # Semua styling
│   └── js/
│       ├── firebase-config.js  # Konfigurasi Firebase
│       ├── common.js           # Helper bersama (db, format, toast)
│       ├── absen.js            # Logika halaman absen
│       ├── data.js             # Logika halaman data
│       └── admin.js            # Logika halaman admin
├── firestore.rules             # Aturan keamanan Firestore
└── README.md
```

---

## 🗄️ Skema Database

### `members` — daftar anggota
| Field | Tipe | Keterangan |
|---|---|---|
| `name` | string | Nama lengkap |
| `nickname` | string | Nama panggilan |
| `emoji` | string | Ikon avatar |
| `active` | boolean | `false` = disembunyikan dari form |
| `createdAt` | timestamp | Waktu dibuat |

### `sessions` — sesi absen
| Field | Tipe | Keterangan |
|---|---|---|
| `title` | string | Judul sesi |
| `date` | string | `YYYY-MM-DD` |
| `openAt` | string | Jam buka `HH:MM` |
| `closeAt` | string | Jam tutup `HH:MM` |
| `auto` | boolean | `true` bila dibuat otomatis |

> Jika hari ini belum ada sesi, halaman absen otomatis membuat sesi
> **"Absen Harian"** pukul `00:00–23:59` agar aplikasi selalu bisa dipakai.

### `attendance` — data kehadiran
| Field | Tipe | Keterangan |
|---|---|---|
| `sessionId` | string | ID sesi |
| `sessionDate` | string | `YYYY-MM-DD` |
| `sessionTitle` | string | Judul sesi (disalin) |
| `memberId` | string | ID anggota |
| `memberName` | string | Nama anggota (disalin) |
| `status` | string | `hadir` / `sakit` / `izin` / `dispen` |
| `reason` | string | Alasan / catatan |
| `createdAt` | timestamp | Pertama kali absen |
| `updatedAt` | timestamp | Terakhir diubah |

ID dokumen: `{sessionId}__{memberId}` → satu anggota hanya punya satu baris per sesi.

### `settings/app` — pengaturan
| Field | Tipe | Default |
|---|---|---|
| `appTitle` | string | `Absensi Harian` |
| `pageTitle` | string | `Absen Hari Ini` |
| `subtitle` | string | teks halaman absen |
| `requireReason` | boolean | `false` |
| `adminPin` | string | `123456` |

---

## 🚀 Cara Menjalankan

### 1. Clone repository
```bash
git clone https://github.com/Destroy27/absensi-pacar.git
cd absensi-pacar
```

### 2. Siapkan Firebase
1. Buka [console.firebase.google.com](https://console.firebase.google.com) → **Add project**
2. Masuk **Build → Firestore Database → Create database**
   - Pilih **Start in test mode**
   - Lokasi: `asia-southeast2 (Jakarta)`
3. Buka **Project settings → Your apps → Web (`</>`)** → daftarkan app
4. Salin `firebaseConfig` ke `assets/js/firebase-config.js`

### 3. Pasang aturan keamanan
Buka **Firestore → Rules**, tempel isi `firestore.rules`, lalu **Publish**.

### 4. Jalankan lokal
Karena memakai ES Module, buka lewat server kecil (bukan `file://`):

```bash
# Python
python -m http.server 8080

# atau Node
npx serve .
```

Lalu buka `http://localhost:8080`.

### 5. Deploy ke GitHub Pages
1. Push ke GitHub
2. **Settings → Pages → Source: Deploy from a branch**
3. Pilih branch `main`, folder `/ (root)`, lalu **Save**
4. Situs tersedia di `https://<username>.github.io/absensi-pacar/`

> Tambahkan domain GitHub Pages ke **Firebase → Authentication → Settings → Authorized domains**
> bila nanti memakai Firebase Auth. Untuk Firestore saja tidak perlu.

---

## 🔐 Keamanan

Aturan di `firestore.rules` membuka baca/tulis **hanya** pada empat koleksi aplikasi:

```js
match /members/{docId}    { allow read, write: if true; }
match /sessions/{docId}   { allow read, write: if true; }
match /attendance/{docId} { allow read, write: if true; }
match /settings/{docId}   { allow read, write: if true; }
```

Catatan penting:

- **PIN admin bukan autentikasi sungguhan.** PIN disimpan di Firestore dan hanya
  menyembunyikan tampilan admin. Siapa pun yang tahu struktur database masih bisa
  menulis lewat SDK.
- Aplikasi ini cocok untuk **pemakaian pribadi / kelompok kecil**.
- Untuk keamanan penuh, tambahkan **Firebase Authentication** dan ubah rules menjadi
  `allow write: if request.auth != null;`.
- Jangan pernah menaruh **service account key** di repo ini.

---

## 🧪 Alur Pemakaian

1. Buka `admin.html` → masuk PIN `123456`
2. Tambah anggota di tab **Anggota**
3. Buat sesi di tab **Sesi** (atau biarkan sesi harian otomatis)
4. Bagikan `index.html` ke pengguna → mereka memilih nama dan absen
5. Pantau rekap di `data.html`, export CSV bila perlu

---

## 🗺️ Rencana Pengembangan

- [ ] Login Firebase Authentication
- [ ] Upload bukti (foto/PDF) via Firebase Storage
- [ ] Rekap bulanan per anggota
- [ ] Notifikasi pengingat absen
- [ ] Mode gelap

---

## 📄 Lisensi

MIT — bebas dipakai dan dimodifikasi.

Dibuat dengan 💗 untuk absensi harian yang simpel.
