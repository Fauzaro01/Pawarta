# Pawarta

**Pawarta: pembawa kabar untuk setiap tenggatmu.**

Pawarta adalah aplikasi web pengingat jadwal kuliah dan deadline tugas berbasis **Python (Flask)**. Pengguna login dengan akun Google, tugas yang dibuat lewat web maupun lewat **Gemini / Google Tasks** disinkronkan otomatis, lalu kabar pengingat dikirim melalui **notifikasi web (Web Push)** dan **WhatsApp**. Tersedia juga fitur **AI** untuk mengubah kalimat bebas menjadi tugas terstruktur.

---

## 1. Makna Nama

**Pawarta** berasal dari kata dalam bahasa Jawa/Kawi yang berarti *pembawa kabar* atau *pemberi kabar*. Filosofinya sejalan dengan fungsi aplikasi: Pawarta bertugas membawa kabar tentang tugas dan tenggat kepada mahasiswa, tepat sebelum waktunya terlewat. Seperti pembawa pesan pada zaman dahulu, Pawarta hadir agar tidak ada kabar penting yang terlambat sampai.

- **Tagline:** *"Pembawa kabar untuk setiap tenggatmu."*
- **Tagline alternatif:** *"Deadline aman, kuliah tenang."*
- **Ide logo:** siluet burung pembawa pesan atau gulungan surat dengan ikon lonceng/notifikasi.

## 2. Latar Belakang

Mahasiswa sering lupa deadline karena tugas tersebar di banyak tempat: grup WhatsApp, pesan dosen, catatan pribadi, dan platform kampus. Pengingat bawaan HP bersifat manual dan tidak terhubung dengan jadwal kuliah. Pawarta mengumpulkan semuanya di satu tempat dan mengingatkan secara otomatis lewat kanal yang pasti dibuka mahasiswa: browser dan WhatsApp.

## 3. Tujuan

1. Menerapkan materi dasar pemrograman Python (variabel, perulangan, list/tuple, pengkondisian, aritmatika) dalam aplikasi nyata.
2. Mengembangkan materi dasar menjadi aplikasi berbasis web dengan library tambahan.
3. Membantu mahasiswa mengelola deadline dengan pengingat otomatis.
4. Mengeksplorasi integrasi dengan layanan Google, WhatsApp, dan AI generatif.

## 4. Fitur

### Fitur Utama
| Fitur | Keterangan |
|-------|------------|
| Login Google | Pengguna masuk memakai email Google (OAuth 2.0) |
| Manajemen tugas | Tambah, ubah, hapus, tandai selesai |
| Jadwal kuliah | Daftar mata kuliah per hari dan jam |
| Hitung sisa waktu | Sisa hari dan jam otomatis menuju deadline |
| Status prioritas | Aman, Mendekati, Mendesak, Terlambat |
| Sinkronisasi Google Tasks / Calendar | Tugas yang dibuat lewat Gemini atau aplikasi Google ikut masuk ke Pawarta |
| Notifikasi web | Pengingat muncul di browser/HP lewat Web Push |
| Notifikasi WhatsApp | Pengingat dikirim ke nomor WhatsApp pengguna |
| Input bahasa natural dengan AI | Contoh: "kumpul tugas PKN Jumat jam 5 sore" menjadi data tugas |

### Fitur Tambahan (Opsional)
| Fitur | Keterangan |
|-------|------------|
| Perintah via Gemini + tombol power | Pengguna menyebut perintah ke Gemini (menahan tombol power) agar tugas dibuat di Google Tasks, lalu otomatis tersinkron ke Pawarta. Ini bukan fitur inti, melainkan bonus karena memanfaatkan jalur resmi Gemini ke Google Tasks |
| Ringkasan mingguan AI | AI merangkum tugas minggu ini dan menyarankan urutan pengerjaan |
| Grafik tugas | Statistik tugas selesai dan terlambat dengan Matplotlib/Chart.js |

## 5. Pemetaan Materi Dasar Python

Bagian ini memudahkan penilaian bahwa semua materi dasar dipakai.

| Materi | Penerapan di Pawarta |
|--------|----------------------|
| **Variabel** | Menyimpan judul, mata kuliah, deadline, status, nomor WhatsApp |
| **List / Tuple** | List untuk daftar tugas dan jadwal; tuple untuk data tetap seperti nama hari `("Senin", "Selasa", ...)` dan kategori status |
| **Perulangan** | Menampilkan semua tugas, memeriksa deadline satu per satu, mengirim pengingat ke tiap pengguna |
| **Pengkondisian** | Menentukan status (aman / mendekati / mendesak / terlambat), validasi input, memilih kanal notifikasi |
| **Aritmatika** | Menghitung sisa hari dan jam, persentase tugas selesai, rata-rata tugas per minggu |

Contoh potongan kode inti (logika dasar):

```python
from datetime import datetime

# TUPLE: kategori status yang tetap
STATUS = ("Aman", "Mendekati", "Mendesak", "Terlambat")

def hitung_status(deadline: datetime):
    sekarang = datetime.now()
    sisa_jam = (deadline - sekarang).total_seconds() / 3600   # ARITMATIKA

    # PENGKONDISIAN
    if sisa_jam < 0:
        return STATUS[3], sisa_jam
    elif sisa_jam <= 24:
        return STATUS[2], sisa_jam
    elif sisa_jam <= 72:
        return STATUS[1], sisa_jam
    else:
        return STATUS[0], sisa_jam

daftar_tugas = []   # LIST
# PERULANGAN
for tugas in daftar_tugas:
    status, sisa = hitung_status(tugas["deadline"])
    print(f'{tugas["judul"]} -> {status} ({sisa:.1f} jam lagi)')
```

## 6. Alur Kerja Sistem

```
Pengguna
   |-- (1) Login Google ------------------> Flask (OAuth 2.0)
   |-- (2) Tambah tugas di web ------------> Flask -> Database
   |-- (3) Ketik kalimat bebas ------------> Flask -> AI (parsing) -> Database
   |-- (4) Ucapkan ke Gemini --------------> Google Tasks / Calendar
                                                  |
                    (penjadwal berkala membaca perubahan lewat Google API)
                                                  v
                                      Flask -> Database (sinkron)
                                                  |
                           Penjadwal (APScheduler) memeriksa deadline
                                                  |
                          +-----------------------+-----------------------+
                          v                                               v
                  Web Push ke browser                         Pesan WhatsApp ke nomor pengguna
```

### Alur Sinkronisasi Google Tasks (Jalur C)
1. Pengguna login Google dan memberi izin akses Tasks (dan/atau Calendar).
2. Pengguna membuat tugas lewat Gemini, misalnya: "Ingatkan aku kumpul laporan Python Jumat jam 5 sore @Google Tasks".
3. Gemini menyimpannya sebagai tugas di Google Tasks.
4. Penjadwal di Pawarta secara berkala (misalnya tiap 5 menit) memanggil Google Tasks API dan mengambil tugas yang baru atau berubah.
5. Tugas baru dimasukkan ke database Pawarta dan ikut masuk ke siklus pengingat.

**Catatan penting:**
- Google Tasks API pada dasarnya menyimpan tenggat berupa tanggal; jam yang diucapkan ke Gemini mungkin tidak ikut. Jika jam penting, gunakan Google Calendar API atau biarkan pengguna melengkapi jam di web.
- Pembuatan tugas lewat suara Gemini dilaporkan kadang tidak stabil. Sediakan cara input alternatif (form web dan input bahasa natural) sehingga Pawarta tetap berfungsi penuh tanpa Gemini.

## 7. Teknologi yang Digunakan

| Bagian | Teknologi |
|--------|-----------|
| Bahasa | Python 3.10+ |
| Web framework | Flask |
| Database | SQLite (bisa diganti PostgreSQL) dengan SQLAlchemy |
| Login | Google OAuth 2.0 (`google-auth-oauthlib`) |
| Google API | `google-api-python-client` (Tasks / Calendar) |
| Penjadwal | APScheduler |
| Notifikasi web | Web Push API, service worker, `pywebpush` (PWA) |
| WhatsApp | WhatsApp Cloud API (Meta) atau gateway pihak ketiga |
| AI | Gemini API (`google-genai`) atau LLM lain |
| Frontend | HTML, CSS, JavaScript (Bootstrap/Tailwind opsional) |
| Konfigurasi | `python-dotenv` |

## 8. Struktur Folder (Rencana)

```
pawarta/
|-- app.py                  # titik masuk Flask
|-- config.py               # membaca konfigurasi dari .env
|-- models.py               # tabel database
|-- requirements.txt
|-- .env.example
|-- core/
|   |-- status.py           # logika dasar: status, sisa waktu (materi dasar Python)
|   |-- scheduler.py        # pemeriksa deadline berkala
|-- services/
|   |-- google_sync.py      # sinkron Google Tasks / Calendar
|   |-- webpush.py          # kirim notifikasi web
|   |-- whatsapp.py         # kirim pesan WhatsApp
|   |-- ai_parser.py        # ubah kalimat bebas menjadi tugas
|-- templates/              # halaman HTML
|-- static/
|   |-- sw.js               # service worker untuk Web Push
|   |-- manifest.json       # PWA (nama aplikasi: Pawarta)
|   |-- css/ js/
|-- README.md
```

## 9. Rancangan Database

**users**: id, email, nama, google_token (terenkripsi), no_whatsapp, push_subscription, dibuat_pada

**courses (jadwal kuliah)**: id, user_id, mata_kuliah, hari, jam_mulai, jam_selesai, ruang

**tasks**: id, user_id, judul, mata_kuliah, deadline, status, sumber (`web` / `ai` / `google_tasks`), google_task_id, selesai, dibuat_pada

**reminders_log**: id, task_id, kanal (`web` / `whatsapp`), waktu_kirim, hasil (agar pengingat tidak terkirim dobel)

## 10. Aturan Pengingat

| Kondisi | Tindakan |
|---------|----------|
| Sisa waktu <= 3 hari | Notifikasi web ringan |
| Sisa waktu <= 24 jam | Notifikasi web + WhatsApp |
| Sisa waktu <= 3 jam | Notifikasi web + WhatsApp (mendesak) |
| Terlambat | Tandai merah, kirim satu pengingat |

Setiap pengingat dicatat di `reminders_log` agar tidak dikirim berulang.

Contoh gaya pesan agar sesuai tema "pembawa kabar":
> **Kabar dari Pawarta:** Tugas "Laporan Python" tenggat besok pukul 17.00 (sisa 20 jam). Semangat!

## 11. Rincian Komponen

### 11.1 Login Google
Pengguna masuk memakai OAuth 2.0 dan memberi izin akses ke Tasks (dan Calendar bila dipakai). Token disimpan terenkripsi dan hanya scope yang diperlukan yang diminta.

### 11.2 Notifikasi Web (Web Push)
- Situs dijadikan PWA (manifest + service worker).
- Browser meminta izin notifikasi, lalu langganan (subscription) disimpan ke database.
- Server mengirim notifikasi memakai kunci VAPID lewat `pywebpush`.
- Wajib HTTPS (kecuali `localhost` saat pengembangan).

### 11.3 Notifikasi WhatsApp
Pilih salah satu:
- **WhatsApp Cloud API (resmi Meta):** pesan di luar jendela layanan 24 jam harus memakai template yang sudah disetujui, hanya untuk pengguna yang sudah opt-in, dan ada biaya per pesan template. Cek tarif terbaru Meta untuk Indonesia.
- **Gateway pihak ketiga:** pengaturan lebih cepat, tetapi tidak resmi dan ada risiko nomor diblokir. Gunakan nomor khusus untuk project, bukan nomor pribadi.

Pengguna wajib menyetujui (opt-in) penerimaan pesan saat mengisi nomor WhatsApp.

### 11.4 AI Parsing
Pengguna mengetik kalimat bebas. AI diminta mengembalikan JSON terstruktur:

```json
{
  "judul": "Kumpul laporan Python",
  "mata_kuliah": "Pemrograman Dasar",
  "deadline": "2026-12-04T17:00:00+07:00"
}
```

Hasil divalidasi dengan kode Python (cek format tanggal, deadline tidak di masa lalu) sebelum disimpan. Tampilkan pratinjau ke pengguna untuk dikonfirmasi.

### 11.5 Fitur Tambahan: Gemini + Tombol Power
Tidak ada pengaturan khusus di sisi aplikasi. Pengguna cukup memanggil Gemini (menahan tombol power), mengucapkan perintah pengingat, dan memastikan tugas tersimpan di Google Tasks. Pawarta lalu mengambilnya lewat sinkronisasi (bagian 6). Integrasi langsung memanggil aplikasi lewat Gemini (AppFunctions / Connected Apps Platform) masih terbatas pada program preview, sehingga dicatat sebagai rencana pengembangan.

## 12. Cara Instalasi (Rencana)

```bash
# 1. Clone dan masuk folder
git clone <url-repo>
cd pawarta

# 2. Buat virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Pasang dependensi
pip install -r requirements.txt

# 4. Salin konfigurasi
cp .env.example .env            # lalu isi nilainya

# 5. Jalankan
flask run
```

### Contoh `.env.example`
```
SECRET_KEY=ganti-dengan-kunci-acak
DATABASE_URL=sqlite:///pawarta.db

GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=http://localhost:5000/oauth/callback

VAPID_PUBLIC_KEY=
VAPID_PRIVATE_KEY=
VAPID_CLAIM_EMAIL=mailto:email@contoh.com

WHATSAPP_PROVIDER=cloud_api   # atau gateway
WHATSAPP_TOKEN=
WHATSAPP_PHONE_ID=

GEMINI_API_KEY=
```

Jangan pernah mengunggah file `.env` ke GitHub. Tambahkan ke `.gitignore`.

## 13. Skenario Demo (Untuk Presentasi)

1. **Pembuka dengan makna nama:** "Pawarta berarti pembawa kabar. Aplikasi ini membawa kabar tenggat kepada mahasiswa sebelum terlambat."
2. **Masalah:** tunjukkan betapa mudahnya lupa deadline.
3. **Login Google** di website.
4. **Tambah tugas lewat form** dan tunjukkan status serta sisa waktu terhitung otomatis (materi dasar).
5. **Tambah tugas dengan kalimat bebas** dan tunjukkan AI mengubahnya menjadi data tugas.
6. **Panggil Gemini** (tahan tombol power), ucapkan perintah pengingat, lalu tunjukkan tugas muncul di Pawarta setelah sinkronisasi.
7. **Picu pengingat:** notifikasi muncul di browser dan pesan masuk ke WhatsApp.
8. **Tunjukkan potongan kode** perhitungan status sebagai bukti penerapan materi dasar.
9. **Tutup dengan rencana pengembangan.**

Siapkan cadangan demo (rekaman layar) bila internet atau layanan Gemini bermasalah.

## 14. Keamanan dan Privasi

- Minta izin Google seminimal mungkin (scope terbatas).
- Simpan token dan kunci API di `.env`, jangan di kode.
- Enkripsi token pengguna di database.
- Minta persetujuan (opt-in) sebelum mengirim WhatsApp.
- Sediakan tombol putus akun dan hapus data.

## 15. Batasan yang Diketahui

- Google Tasks API hanya mendukung tanggal tenggat; jam bisa hilang.
- Pembuatan tugas lewat suara Gemini dapat gagal sesekali.
- WhatsApp resmi memerlukan template dan biaya per pesan; gateway tidak resmi berisiko diblokir.
- Web Push memerlukan HTTPS dan izin notifikasi dari pengguna.

## 16. Rencana Pengembangan

- Integrasi langsung dengan Gemini lewat AppFunctions / Connected Apps Platform bila akses sudah terbuka.
- Aplikasi Android (PWA terpasang atau Flutter).
- Impor jadwal kuliah dari file atau kalender kampus.
- Mode kolaborasi tugas kelompok.
- Statistik produktivitas dan pencapaian.

## 17. Pengembang

- **Nama:** Muhamad Fauzaan
- **NIM:** 15260750
- **Program Studi:** Informatika
- **Universitas:** Universitas Bina Sarana Informatika (BSI), Kampus Cikampek
- **Kelas:** 15.1A.15
