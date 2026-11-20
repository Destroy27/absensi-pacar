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
- **Export CSV** (ramah Excel) dan **cetak / simpan PDF**
- Hapus satu baris data bila salah input

### Halaman Admin
- Masuk dengan **PIN admin** (PIN awal `123456`, bisa diganti)
- **Kelola anggota**: tambah, edit, hapus, dan tandai **aktif / nonaktif**
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

Tanpa `npm install` dan tanpa bundler. Tetap butuh **satu server statis kecil**
(lihat langkah 4), karena ES Module tidak berjalan lewat `file://`.

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
├── LICENSE
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
| `active` | boolean | `false` = disembunyikan dari form absen |
| `createdAt` | timestamp | Waktu dibuat |

### `sessions` — sesi absen
| Field | Tipe | Keterangan |
|---|---|---|
| `title` | string | Judul sesi |
| `date` | string | `YYYY-MM-DD` |
| `openAt` | string | Jam buka `HH:MM` |
| `closeAt` | string | Jam tutup `HH:MM` (harus lebih akhir dari `openAt`) |
| `auto` | boolean | `true` bila dibuat otomatis |
| `createdAt` | timestamp | Waktu dibuat |

> **Sesi otomatis:** kalau hari ini **belum ada satu pun sesi**, halaman absen membuat
> sesi **"Absen Harian"** pukul `00:00–23:59`. Kalau sesi hari ini sudah ada tetapi
> sudah lewat jam tutup, absen tidak bisa dilakukan sampai kamu membuat sesi baru di Admin.

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
| `subtitle` | string | teks sambutan halaman absen |
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
4. Buka `assets/js/firebase-config.js`. Isinya masih **config proyek contoh** —
   **ganti seluruh objek `firebaseConfig`** dengan milik proyek Firebase kamu.
   Jangan pakai config contoh, karena datanya bukan milikmu.

> ⚠️ **Mode test hanya berlaku 30 hari.** Kalau langkah 3 dilewati, aplikasi akan
> berhenti bisa menyimpan data setelah sebulan. Selalu pasang rules di langkah berikut.

### 3. Pasang aturan keamanan
Buka **Firestore → Rules**, tempel isi `firestore.rules`, lalu **Publish**.

### 4. Jalankan lokal
ES Module butuh server, bukan `file://`:

```bash
# Python (port 8080)
python -m http.server 8080
# buka http://localhost:8080

# atau Node (port mengikuti yang dicetak, biasanya 3000)
npx serve .
```

### 5. Deploy ke GitHub Pages
1. Push ke GitHub (repo harus **public** untuk akun gratis)
2. **Settings → Pages → Source: Deploy from a branch**
3. Pilih branch `main`, folder `/ (root)`, lalu **Save**
4. Situs tersedia di `https://<username>.github.io/absensi-pacar/`

---

## 🧪 Alur Pemakaian

1. Buka `admin.html` → masuk PIN `123456`
2. Tambah anggota di tab **Anggota** (centang "Tampil di halaman absen")
3. Buat sesi di tab **Sesi** (atau biarkan sesi harian otomatis)
4. Bagikan **link** `https://<username>.github.io/absensi-pacar/` — bukan file HTML-nya
5. Pantau rekap di `data.html`, export CSV berkala sebagai backup

> Login admin disimpan di `sessionStorage`: tetap masuk sampai tab ditutup.
> Tombol **Keluar** menghapus sesi login tersebut.

---

## 🔐 Keamanan

Aturan di `firestore.rules` membuka baca/tulis **hanya** pada empat koleksi aplikasi,
lalu menolak sisanya:

```js
match /members/{docId}    { allow read, write: if true; }
match /sessions/{docId}   { allow read, write: if true; }
match /attendance/{docId} { allow read, write: if true; }
match /settings/{docId}   { allow read, write: if true; }

// koleksi lain ditolak
match /{document=**}      { allow read, write: if false; }
```

Yang perlu kamu tahu:

- **PIN admin bukan autentikasi sungguhan.** PIN disimpan **teks biasa** di
  `settings/app` dan bisa dibaca siapa pun yang membuka aplikasi. Jangan pakai PIN
  yang sama dengan ATM, email, atau akun lain.
- Siapa pun yang punya config Firebase bisa **membaca, mengubah, bahkan menghapus**
  data lewat SDK. `firebase-config.js` memang aman dipublikasikan, tetapi rules-nya
  yang menentukan siapa boleh apa.
- Aplikasi ini cocok untuk **pemakaian pribadi / kelompok kecil**, bukan data sensitif.
- Untuk keamanan penuh: tambahkan **Firebase Authentication**, lalu ubah rules menjadi
  `allow write: if request.auth != null;`.
- Jangan pernah menaruh **service account key** di repo ini.
- Rutin **export CSV** sebagai backup.

---

## 🩺 Troubleshooting

| Gejala | Penyebab | Solusi |
|---|---|---|
| `Missing or insufficient permissions` | Rules belum di-Publish | Firestore → Rules → **Publish** |
| Halaman kosong / tidak jalan | Dibuka lewat `file://` | Jalankan server statis (langkah 4) |
| Data tidak muncul, padahal sudah isi | `firebase-config.js` masih config contoh | Ganti dengan config proyekmu |
| Absen otomatis gagal / error saat buka | Rules menolak tulis `sessions` | Pastikan rules `sessions` terpasang |
| Absen tiba-tiba berhenti setelah ~30 hari | Mode test kedaluwarsa | Pasang `firestore.rules` |
| `404` di GitHub Pages | Pages belum aktif / branch salah | Settings → Pages → `main` + `/ (root)` |
| Tombol kirim mati | Di luar jam buka/tutup sesi | Buat sesi baru di Admin |

---

## 🗺️ Rencana Pengembangan

- [ ] Login Firebase Authentication
- [ ] Upload bukti (foto/PDF) via Firebase Storage
- [ ] Rekap bulanan per anggota
- [ ] Notifikasi pengingat absen
- [ ] Mode gelap

---

## 📄 Lisensi

MIT — lihat [LICENSE](LICENSE).

Dibuat dengan 💗 untuk absensi harian yang simpel.
