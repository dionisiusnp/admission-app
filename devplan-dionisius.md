# Development Plan — Modul Penjadwalan Tes Seleksi PMB
## Vibe Coding & Venture SEVIMA
**Nama:** Dionisius NP | **Tanggal:** 2026-06-13

---

## BAGIAN 1 — Analisa Teknis

### 1.1 Identifikasi Pengguna

| Pengguna | Peran dalam Modul Penjadwalan |
|----------|-------------------------------|
| **Admin PMB** | Membuat sesi tes, menentukan kapasitas & lokasi, assign pendaftar ke sesi, memantau kehadiran, reschedule jika perlu |
| **Panitia Tes** | Peran baru — melihat daftar peserta per sesi tes untuk keperluan teknis di hari H (absensi, persiapan ruang), tandai kehadiran |
| **Calon Mahasiswa** | Melihat jadwal tes mereka via nomor pendaftaran (publik, tanpa login), tahu lokasi & waktu, bisa request reschedule |

> Catatan: Admin sudah ada di sistem (Sanctum login). Calon mahasiswa sudah ada (CekStatus via nomor pendaftaran). Panitia Tes adalah persona **baru** — akses read-only ke daftar peserta sesi tanpa perlu full admin access.

---

### 1.2 Fitur Utama per Pengguna

**Admin PMB:**
- Buat sesi tes (nama, tanggal, waktu mulai–selesai, lokasi, kapasitas, tipe tes: Tertulis/Wawancara)
- Assign otomatis / manual pendaftar berstatus "Lolos Seleksi" ke sesi tes
- Lihat daftar peserta per sesi beserta status kehadiran
- Tandai kehadiran peserta (Hadir / Tidak Hadir)
- Reschedule peserta ke sesi lain (kapasitas tersedia)
- Lihat ringkasan kapasitas per sesi (terisi / total)

**Panitia Tes:**
- Lihat daftar peserta sesi tes hari ini (read-only, filter by tanggal)
- Tandai kehadiran peserta per sesi yang mereka pegang
- Cetak / export daftar peserta sesi ke PDF atau CSV

**Calon Mahasiswa (publik):**
- Cek jadwal tes via nomor pendaftaran (integrasi ke halaman CekStatus yang sudah ada)
- Lihat detail: nama sesi, tanggal, waktu, lokasi, tipe tes
- Lihat status kehadiran setelah hari tes
- Request reschedule (submit form — diproses admin secara manual)

---

### 1.3 Tech Stack Tambahan

| Komponen | Pilihan | Alasan |
|----------|---------|--------|
| Date/time input frontend | `<input type="datetime-local">` native HTML | Tanpa dependency tambahan, sudah cukup untuk prototype |
| Kalender view (opsional) | Tidak ada — gunakan tabel biasa | Sesuai scope prototype, kalender library (react-calendar) tambah kompleksitas tidak perlu |
| Notifikasi peserta | Tidak ada di Fase ini — jadwal cukup ditampilkan di CekStatus | WhatsApp/email butuh third-party service — di luar scope prototype |
| Export daftar peserta sesi | Laravel StreamedResponse (sama dengan exportCsv yang sudah ada) | Konsisten dengan pattern yang sudah ada di PendaftarController |
| Auth panitia | Sanctum token (tabel `users` sudah ada) | Gunakan role field di `users`, tidak perlu auth library baru |

---

### 1.4 Batasan & Asumsi

1. **Hanya pendaftar berstatus `Lolos Seleksi` yang bisa dijadwalkan** — modul baca dari tabel `pendaftars` where `status = 'Lolos Seleksi'`. Pendaftar `Menunggu` atau `Tidak Lolos` tidak tampil di pool assignment.

2. **Satu pendaftar maksimal satu jadwal aktif** — tidak ada multi-jadwal per pendaftar. Jika admin reschedule, jadwal lama di-update (bukan ditambah record baru).

3. **Tidak ada notifikasi otomatis** — modul ini tidak mengirim email/WhatsApp. Informasi jadwal tersedia di halaman CekStatus publik; peserta diasumsikan cek sendiri.

4. **Kapasitas sesi bersifat soft limit** — backend validasi kapasitas saat assign, tapi admin bisa override jika perlu (via flag `force`). Tidak diimplementasi di prototype.

5. **Integrasi non-breaking** — semua route, model, dan komponen yang ada tidak diubah strukturnya. Modul baru hanya menambah tabel, routes, controller, dan komponen baru.

6. **SQLite dev, PostgreSQL prod** — ikuti keputusan yang sudah ada; tidak ada perubahan strategi database.

---

## BAGIAN 2 — Bisnis Proses & Flow

### 2.1 Flow Utama: Penjadwalan Tes Seleksi

```
[Admin] → Buat sesi tes via form dashboard → [POST /api/sesi-tes dikirim]
        ↓
[SISTEM] → Validasi input (nama, tanggal, waktu, kapasitas > 0) → [StoreSesiTesRequest]
        ↓ jika valid
[SISTEM] → Simpan ke tabel `sesi_tes` → [Record sesi baru tersimpan]
        ↓
[Admin] → Buka panel assign (GET /api/sesi-tes/{id}/kandidat) → [List pendaftar status='Lolos Seleksi' belum terjadwal]
        ↓
[Admin] → Pilih pendaftar & submit assign (POST /api/sesi-tes/{id}/assign, body: pendaftar_ids[]) → [Request diterima backend]
        ↓
[SISTEM] → Cek kapasitas: COUNT(jadwal_pendaftar WHERE sesi_tes_id) < kapasitas → [Hasil cek]
        ↓ jika kapasitas cukup
[SISTEM] → INSERT INTO jadwal_pendaftar (status_kehadiran='Terjadwal') → [Jadwal tersimpan per pendaftar]
        ↓ jika kapasitas penuh
[SISTEM] → Tolak request → [HTTP 422: "Kapasitas sesi sudah penuh"]
        ↓
[Calon Mahasiswa] → Input nomor pendaftaran di halaman CekStatus → [GET /api/jadwal/{nomorPendaftaran}]
        ↓
[SISTEM] → JOIN jadwal_pendaftar + sesi_tes + pendaftars WHERE nomor_pendaftaran = ? → [Data jadwal atau null]
        ↓ jika jadwal ditemukan
[Calon Mahasiswa] → Lihat kartu jadwal (nama sesi, tanggal, waktu, lokasi, tipe tes) → [Info lengkap tampil]
        ↓ jika belum dijadwalkan
[Calon Mahasiswa] → Lihat pesan "Jadwal belum ditentukan" → [Pantau berkala]
        ↓
[Admin/Panitia] → Buka daftar peserta sesi hari H (GET /api/sesi-tes/{id}/peserta) → [List peserta per sesi]
        ↓
[Admin/Panitia] → Tandai kehadiran (PATCH /api/jadwal/{jadwalId}/kehadiran, body: status_kehadiran) → [Update dikirim]
        ↓
[SISTEM] → UPDATE jadwal_pendaftar SET status_kehadiran = 'Hadir'|'Tidak Hadir' → [Status tersimpan, tampil di CekStatus peserta]
```

---

### 2.2 Flow Alternatif: Peserta Minta Reschedule

```
[Calon Mahasiswa] → Di halaman CekStatus, klik "Minta Ubah Jadwal"
                  → Form muncul: nomor pendaftaran (auto-filled) + alasan + pilihan sesi tersedia
                  → Submit form (POST /api/jadwal/reschedule-request)
                    → [SISTEM] Simpan request (tidak langsung ubah jadwal)
                    → Response: "Permintaan diterima, admin akan menghubungi kamu"
       ↓
[Admin] → Lihat daftar reschedule request di dashboard
        → Review: pilih sesi baru untuk pendaftar tersebut
        → PATCH /api/jadwal/{jadwalId} (update sesi_tes_id ke sesi baru)
        → [SISTEM] Validasi kapasitas sesi baru
                 ↓ kapasitas tersedia
        → Update `jadwal_pendaftar.sesi_tes_id` + reset `status_kehadiran` ke 'Terjadwal'
        → Response: jadwal ter-update
```

> Catatan prototype: tabel `reschedule_requests` tidak diimplementasi di scope ini — request disimpan sebagai catatan di `jadwal_pendaftar.catatan` dan admin reschedule manual via dashboard.

---

### 2.3 Happy Path vs Error Path

**Happy path:**
1. Admin buat sesi tes → sesi tersimpan, tampil di daftar sesi
2. Admin assign 10 pendaftar ke sesi kapasitas 30 → semua ter-assign, status `Terjadwal`
3. Peserta buka CekStatus → jadwal tampil lengkap (nama sesi, tanggal, lokasi)
4. Hari H panitia tandai hadir → status `Hadir` tersimpan, tampil di CekStatus peserta

**Error path:**

| Kondisi Error | Respons Sistem |
|--------------|----------------|
| Assign ke sesi yang sudah penuh (kapasitas tercapai) | HTTP 422 — `"Kapasitas sesi sudah penuh. Sisa: 0 slot"` |
| GET jadwal untuk nomor pendaftaran yang belum dijadwalkan | HTTP 200 dengan `data: null` — frontend tampil pesan "Jadwal tes belum ditentukan, pantau terus halaman ini" |
| Admin buat sesi dengan `waktu_selesai` < `waktu_mulai` | HTTP 422 dari FormRequest validation — `"Waktu selesai harus setelah waktu mulai"` |
| Pendaftar dengan status bukan `Lolos Seleksi` dicoba di-assign | HTTP 422 — `"Pendaftar ini tidak memenuhi syarat penjadwalan"` |

---

## BAGIAN 3 — Alur Data

### 3.1 Alur Data: Proses Penjadwalan

**Proses Buat Sesi:**
```
[Admin: form di ManajemenSesi.jsx] → [jadwalApi.createSesi() — apiFetch POST /api/sesi-tes] → [SesiTesController@store: StoreSesiTesRequest → INSERT sesi_tes] → [Response { success, data: sesi }] → [React: state sesiList update, kartu sesi tampil di dashboard]
```

**Proses Assign Pendaftar ke Sesi:**
```
[Admin: checklist kandidat di ManajemenSesi.jsx] → [jadwalApi.assignPendaftar(sesiId, ids[]) — apiFetch POST /api/sesi-tes/{id}/assign] → [SesiTesController@assign: SELECT pendaftars WHERE status='Lolos Seleksi' → validasi kapasitas → INSERT jadwal_pendaftar] → [Response { success, data: jadwal[] }] → [React: refresh daftar peserta sesi, badge kapasitas update]
```

---

### 3.2 Alur Data: Peserta Cek Jadwal

```
[Calon Mahasiswa: input nomor pendaftaran di CekStatus.jsx]
    → [React: CekStatus.jsx — state: nomorPendaftaran]
    → [GET /api/jadwal/{nomorPendaftaran} — api.js: jadwalApi.getByNomor()]
    → [Laravel: JadwalController@showByNomor]
    → [Query:
        SELECT jp.*, st.nama_sesi, st.tanggal, st.waktu_mulai, st.waktu_selesai, st.lokasi, st.tipe_tes
        FROM jadwal_pendaftar jp
        JOIN sesi_tes st ON jp.sesi_tes_id = st.id
        JOIN pendaftars p ON jp.pendaftar_id = p.id
        WHERE p.nomor_pendaftaran = ?
       ]
    → [Response JSON: { success, data: jadwal | null }]
    → [React: CekStatus.jsx tampil kartu jadwal atau pesan "Belum terjadwal"]
```

---

### 3.3 Data Sensitif

| Field / Data | Perlakuan Khusus | Alasan |
|--------------|-----------------|--------|
| `nomor_hp` pendaftar | Jangan tampilkan di endpoint publik `/api/jadwal/{nomor}` | Data pribadi — endpoint ini tidak butuh autentikasi |
| `email` pendaftar | Sama — jangan masuk ke response publik | Privasi, potensi spam |
| `lokasi` sesi tes | Boleh publik — info ini memang harus diketahui peserta | Bukan data sensitif |
| Token Sanctum admin/panitia | Tetap di `sessionStorage` browser | Sesuai pattern yang sudah ada, hilang saat tab tutup |
| Data kehadiran per peserta | Hanya admin/panitia yang bisa lihat list lengkap | Peserta hanya lihat status kehadiran milik sendiri |

---

## BAGIAN 4 — ERD / Desain Database

### 4.1 Daftar Tabel

| Nama Tabel | Deskripsi |
|------------|-----------|
| `sesi_tes` | **(Baru)** Satu sesi tes seleksi — menyimpan jadwal, lokasi, kapasitas, dan tipe tes |
| `jadwal_pendaftar` | **(Baru)** Tabel pivot — menghubungkan pendaftar ke sesi tes, menyimpan status kehadiran |
| `pendaftars` | **(Sudah ada)** Data calon mahasiswa — dibaca via FK `pendaftar_id` |
| `users` | **(Sudah ada)** Akun admin/panitia — dibaca via Sanctum, field `role` ditambah |

---

### 4.2 Struktur Tiap Tabel

**Tabel: `sesi_tes`**

| Nama Kolom | Tipe Data | Constraint | Keterangan |
|------------|-----------|------------|------------|
| `id` | BIGINT UNSIGNED | PK, AUTO_INCREMENT | Primary key |
| `nama_sesi` | VARCHAR(100) | NOT NULL | Contoh: "Tes Tertulis Gelombang 1" |
| `tipe_tes` | VARCHAR(20) | NOT NULL | Enum: 'Tertulis', 'Wawancara' |
| `tanggal` | DATE | NOT NULL | Tanggal pelaksanaan tes |
| `waktu_mulai` | TIME | NOT NULL | Jam mulai sesi |
| `waktu_selesai` | TIME | NOT NULL | Jam selesai; harus > waktu_mulai |
| `lokasi` | VARCHAR(200) | NOT NULL | Gedung/ruang/alamat pelaksanaan |
| `kapasitas` | SMALLINT UNSIGNED | NOT NULL, DEFAULT 30 | Maks peserta per sesi |
| `created_at` | TIMESTAMP | NULLABLE | Auto-managed Laravel |
| `updated_at` | TIMESTAMP | NULLABLE | Auto-managed Laravel |

**Tabel: `jadwal_pendaftar`**

| Nama Kolom | Tipe Data | Constraint | Keterangan |
|------------|-----------|------------|------------|
| `id` | BIGINT UNSIGNED | PK, AUTO_INCREMENT | Primary key |
| `pendaftar_id` | BIGINT UNSIGNED | NOT NULL, FK → pendaftars.id | Pendaftar yang dijadwalkan |
| `sesi_tes_id` | BIGINT UNSIGNED | NOT NULL, FK → sesi_tes.id | Sesi yang dituju |
| `status_kehadiran` | VARCHAR(20) | NOT NULL, DEFAULT 'Terjadwal' | Enum: 'Terjadwal', 'Hadir', 'Tidak Hadir' |
| `catatan` | TEXT | NULLABLE | Catatan admin (misal alasan reschedule) |
| `created_at` | TIMESTAMP | NULLABLE | Auto-managed Laravel |
| `updated_at` | TIMESTAMP | NULLABLE | Auto-managed Laravel |

**Perubahan tabel `users` (existing — tambah kolom):**

| Nama Kolom | Tipe Data | Constraint | Keterangan |
|------------|-----------|------------|------------|
| `role` | VARCHAR(20) | NOT NULL, DEFAULT 'admin' | 'admin' atau 'panitia' |

---

### 4.3 Relasi Antar Tabel

```
[pendaftars] ---(1:0..1)--- [jadwal_pendaftar]
Keterangan: Satu pendaftar maksimal punya satu jadwal aktif.
            FK: jadwal_pendaftar.pendaftar_id → pendaftars.id
            Relasi 1:0..1 karena tidak semua pendaftar lolos & dijadwalkan.

[sesi_tes] ---(1:N)--- [jadwal_pendaftar]
Keterangan: Satu sesi bisa punya banyak peserta (dibatasi oleh kolom kapasitas).
            FK: jadwal_pendaftar.sesi_tes_id → sesi_tes.id
            Sisi N di jadwal_pendaftar.

[users] ---(referensi logis, bukan FK)--- [sesi_tes]
Keterangan: Tidak ada FK eksplisit — admin yang buat sesi tidak dicatat as creator
            di prototype ini. Dapat ditambah kolom created_by_user_id di iterasi berikutnya.
```

**Diagram ringkas:**
```
pendaftars (existing)
    │ id (PK)
    │ nomor_pendaftaran
    │ status
    └──┐
       │ FK: pendaftar_id
jadwal_pendaftar (new)
    │ id (PK)
    │ pendaftar_id  ──────→ pendaftars.id
    │ sesi_tes_id   ──────→ sesi_tes.id
    │ status_kehadiran
    └──┐
       │ FK: sesi_tes_id
sesi_tes (new)
    id (PK)
    nama_sesi
    tanggal
    kapasitas
    ...
```

---

### 4.4 Indexing

| Tabel | Kolom | Tipe Index | Alasan |
|-------|-------|-----------|--------|
| `jadwal_pendaftar` | `pendaftar_id` | INDEX | Query paling sering: `WHERE pendaftar_id = ?` saat peserta cek jadwal by nomor_pendaftaran |
| `jadwal_pendaftar` | `sesi_tes_id` | INDEX | Join & GROUP BY saat hitung kapasitas terisi per sesi |
| `jadwal_pendaftar` | `(pendaftar_id)` | UNIQUE | Satu pendaftar hanya boleh satu jadwal aktif — enforce di DB level |
| `sesi_tes` | `tanggal` | INDEX | Filter sesi by tanggal (view harian panitia) |
| `sesi_tes` | `tipe_tes` | INDEX | Filter by tipe tes di dashboard admin |

---

## BAGIAN 5 — Prompt Siap Pakai untuk AI

Prompt ini dirancang agar bisa dijalankan langsung di **Claude Code CLI** (`claude`) maupun **Claude.ai browser**. Tidak ada asumsi konteks sesi sebelumnya — semua info yang dibutuhkan AI tersedia dalam prompt ini.

---

```
[KONTEKS]
Kamu sedang bekerja di project "Sistem PMB (Penerimaan Mahasiswa Baru)" yang sudah berjalan dengan stack:
- Frontend: React 18 + Vite + Tailwind CSS (di folder pmb-frontend/)
- Backend: Laravel 12 + SQLite (di folder pmb-backend/)
- Auth: Laravel Sanctum (Bearer token di sessionStorage dengan key 'pmb_admin_token')
- Fetch wrapper: src/utils/api.js menggunakan apiFetch() dengan format response standar { success, data, message }

Fitur yang SUDAH ADA dan tidak boleh diubah:
- POST/GET /api/pendaftar — form pendaftaran & cek status by nomor pendaftaran
- PATCH /api/pendaftar/{id}/status — update status pendaftar (Menunggu/Lolos Seleksi/Tidak Lolos)
- POST/POST /api/auth/login|logout — Sanctum auth
- GET /api/statistik — statistik per prodi & jalur
- GET /api/pendaftar/export/csv — export CSV (protected)
- POST /api/pendaftar/{nomor}/heregistrasi — heregistrasi mahasiswa lolos

Tabel database yang SUDAH ADA:
- pendaftars: id, nomor_pendaftaran, nama, nomor_hp, email, asal_sekolah, prodi, jalur, status, heregistrasi_at, timestamps
- users: id, name, username, password, remember_token, timestamps (+ tambah kolom 'role' VARCHAR(20) default 'admin')
- personal_access_tokens (Sanctum)

Konvensi kode yang HARUS diikuti:
- React: functional component, camelCase hooks, PascalCase komponen, Tailwind only (no inline style)
- Laravel: snake_case kolom, PHPDoc di setiap method, try-catch di controller, FormRequest untuk validasi
- Response API: selalu { "success": bool, "data": ..., "message": string }
- Konstanta PHP di Model: const STATUS_X = 'X'
- Error di React: tampilkan inline, bukan alert()
- Warna Tailwind: blue-600 (utama), amber-500 (aksen), slate-800 (teks), slate-200 (border)

[TUJUAN]
Tambahkan modul penjadwalan tes seleksi ke sistem yang sudah ada.
Scope: admin bisa buat sesi tes dan assign pendaftar yang sudah "Lolos Seleksi" ke sesi tersebut. Peserta bisa cek jadwal mereka via nomor pendaftaran di halaman publik CekStatus. Admin/panitia bisa tandai kehadiran hari H.
Tidak perlu: notifikasi email/WhatsApp, kalender visual, autentikasi panitia terpisah.

[FITUR]
Backend — buat file-file berikut:
1. Migration: create_sesi_tes_table
   Kolom: id, nama_sesi VARCHAR(100), tipe_tes VARCHAR(20), tanggal DATE, waktu_mulai TIME, waktu_selesai TIME, lokasi VARCHAR(200), kapasitas SMALLINT default 30, timestamps
   
2. Migration: create_jadwal_pendaftar_table
   Kolom: id, pendaftar_id FK→pendaftars.id, sesi_tes_id FK→sesi_tes.id, status_kehadiran VARCHAR(20) default 'Terjadwal', catatan TEXT nullable, timestamps
   Index: UNIQUE(pendaftar_id), INDEX(sesi_tes_id), INDEX(sesi_tes.tanggal)

3. Migration: add_role_to_users_table
   Tambah kolom: role VARCHAR(20) NOT NULL DEFAULT 'admin'

4. Model SesiTes.php — fillable semua kolom, konstanta TIPE_TERTULIS='Tertulis', TIPE_WAWANCARA='Wawancara', hasMany JadwalPendaftar

5. Model JadwalPendaftar.php — fillable semua kolom, konstanta STATUS_TERJADWAL/HADIR/TIDAK_HADIR, belongsTo Pendaftar & SesiTes

6. Controller SesiTesController.php (App\Http\Controllers\Api) — method:
   - index(): list semua sesi + hitung peserta_terdaftar per sesi
   - store(): buat sesi baru (validasi waktu_selesai > waktu_mulai, kapasitas > 0)
   - show($id): detail sesi + list peserta (JOIN jadwal_pendaftar + pendaftars, tanpa nomor_hp & email)
   - kandidat($id): list pendaftar status='Lolos Seleksi' yang belum punya jadwal
   - assign($id): batch insert ke jadwal_pendaftar, validasi kapasitas, tolak jika pendaftar bukan 'Lolos Seleksi'
   - destroy($id): soft check — tolak jika sesi sudah ada peserta terdaftar

7. Controller JadwalController.php — method:
   - showByNomor($nomorPendaftaran): GET jadwal by nomor pendaftaran (publik, JANGAN include nomor_hp/email di response)
   - updateKehadiran($id): PATCH status_kehadiran (Hadir/Tidak Hadir), protected Sanctum

8. Routes baru di routes/api.php (tambah ke grup yang sudah ada):
   Publik: GET /jadwal/{nomorPendaftaran}
   Protected (auth:sanctum): GET/POST /sesi-tes, GET /sesi-tes/{id}, DELETE /sesi-tes/{id}
   Protected: GET /sesi-tes/{id}/kandidat, POST /sesi-tes/{id}/assign
   Protected: PATCH /jadwal/{id}/kehadiran

Frontend — buat/modifikasi file berikut:
9. src/utils/api.js — tambah export jadwalApi:
   - createSesi(data), getSesiList(), getSesiDetail(id), getSesiKandidat(id)
   - assignPendaftar(sesiId, pendaftarIds[]), deleteKehadiran(jadwalId, status)
   - getJadwalByNomor(nomorPendaftaran)

10. src/components/pmb/JadwalTes.jsx — komponen baru untuk CekStatus:
    Tampilkan jadwal jika ada (kartu dengan ikon kalender, nama sesi, tanggal formatted, waktu, lokasi, tipe tes, badge status kehadiran)
    Tampilkan pesan "Jadwal belum ditentukan" jika null

11. src/components/pmb/ManajemenSesi.jsx — komponen admin:
    - Form buat sesi baru (semua field sesi_tes)
    - Tabel daftar sesi (nama, tanggal, waktu, lokasi, peserta/kapasitas, aksi)
    - Modal detail sesi: list peserta + kapasitas tersisa
    - Di modal: tombol "Assign Pendaftar" buka sub-panel daftar kandidat (checklist) + tombol simpan
    - Tandai kehadiran per peserta (dropdown atau toggle Hadir/Tidak Hadir)

12. Modifikasi src/components/pmb/CekStatus.jsx:
    - Setelah data pendaftar berhasil di-fetch dan status='Lolos Seleksi', fetch juga GET /api/jadwal/{nomor}
    - Render <JadwalTes jadwal={jadwal} /> di bawah info status pendaftar
    - Jika status bukan 'Lolos Seleksi', jangan tampilkan section jadwal

13. Modifikasi src/pages/Admin.jsx:
    - Tambah tab navigasi: "Pendaftar" (existing) | "Penjadwalan Tes" (new)
    - Tab "Penjadwalan Tes" render <ManajemenSesi />
    - State activeTab, default ke 'pendaftar' agar tidak merusak tampilan existing

[CONSTRAINT]
- Jangan ubah atau hapus kode yang sudah ada — hanya tambah
- Endpoint publik /api/jadwal/{nomor} TIDAK boleh return nomor_hp, email, atau id numerik pendaftar
- Validasi backend: pendaftar yang di-assign harus status='Lolos Seleksi', sesi harus belum penuh
- UNIQUE constraint di jadwal_pendaftar.pendaftar_id — handle di controller dengan cek exists() sebelum insert
- Ikuti format response { success, data, message } di semua endpoint baru
- Semua konstanta status jadwal (TERJADWAL, HADIR, TIDAK_HADIR) didefinisikan di Model JadwalPendaftar.php
- Tambah jadwalApi ke src/utils/api.js — jangan buat file api baru
- Komponen React baru menggunakan pola yang sama dengan Admin.jsx: useState, useEffect, try/catch, loading & error state
- Styling: Tailwind only, gunakan warna yang konsisten (blue-600 utama, slate-200 border, rounded-xl card)
- Jalankan migration dengan: php artisan migrate (jangan --fresh — data pendaftar existing harus tetap ada)

[TAMPILAN]
Konsistensi visual dengan UI yang sudah ada (putih bersih, border slate-200, rounded-xl):

Tab navigasi di Admin.jsx:
- Dua tab di bawah header: "Data Pendaftar" | "Penjadwalan Tes"
- Tab aktif: border-b-2 border-blue-600 text-blue-600
- Tab inactive: text-slate-500 hover:text-slate-700

Kartu sesi tes di ManajemenSesi.jsx:
- Grid 1-col mobile, 2-col sm, 3-col lg
- Setiap kartu: bg-white border border-slate-200 rounded-xl p-4
- Badge kapasitas: "12/30 peserta" — jika > 80% penuh ganti teks ke text-amber-600
- Tombol buat sesi: bg-blue-600 text-white rounded-lg px-4 py-2

Badge status kehadiran (konsisten dengan StatusBadge.jsx yang sudah ada):
- Terjadwal: bg-yellow-100 text-yellow-800
- Hadir: bg-green-100 text-green-800
- Tidak Hadir: bg-red-100 text-red-800

Kartu jadwal di CekStatus.jsx (komponen JadwalTes.jsx):
- Tampil di bawah info status, hanya jika status='Lolos Seleksi'
- bg-blue-50 border border-blue-200 rounded-xl p-4
- Header: ikon kalender (SVG) + "Jadwal Tes Seleksi Kamu"
- Baris info: Tanggal, Waktu, Lokasi, Tipe Tes, Status Kehadiran (badge)
- Jika belum dijadwalkan: bg-slate-50, teks "Jadwal belum ditentukan oleh panitia. Pantau terus halaman ini."
```

---

## BAGIAN 6 — Jalankan Prompt & Evaluasi Hasil *(Bonus)*

> Bagian ini diisi setelah prompt di Bagian 5 dieksekusi ke AI dan hasilnya dijalankan di browser.

### 6.1 Log Prompt & Iterasi

*(Diisi setelah eksekusi)*

**Prompt Utama:** (copy dari Bagian 5)

**Iterasi 1:** *(jika ada — tuliskan mengapa iterasi diperlukan)*

---

### 6.2 Tabel Evaluasi Kesesuaian

| Fitur / Item Plan | Status | Catatan |
|------------------|--------|---------|
| Migration sesi_tes | — | — |
| Migration jadwal_pendaftar + index | — | — |
| Model SesiTes + konstanta | — | — |
| Model JadwalPendaftar + konstanta | — | — |
| SesiTesController (index, store, show, kandidat, assign, destroy) | — | — |
| JadwalController (showByNomor, updateKehadiran) | — | — |
| Routes baru (publik + protected) | — | — |
| jadwalApi di api.js | — | — |
| Komponen JadwalTes.jsx | — | — |
| Komponen ManajemenSesi.jsx (form + tabel + modal + assign) | — | — |
| CekStatus.jsx — tampil jadwal jika Lolos Seleksi | — | — |
| Admin.jsx — tab "Penjadwalan Tes" | — | — |
| Fitur lama: Form Pendaftaran masih berjalan | — | — |
| Fitur lama: CekStatus (non-jadwal) masih berjalan | — | — |
| Fitur lama: Dashboard admin + statistik masih berjalan | — | — |
| Fitur lama: Export CSV masih berjalan | — | — |
| Fitur lama: Heregistrasi masih berjalan | — | — |

---

### 6.3 Push Hasil ke Repo

*(Diisi setelah fitur jalan di browser)*

Struktur repo target:
```
admission-app/
├── devplan-dionisius.md        ← file ini
└── app/
    ├── pmb-frontend/
    ├── pmb-backend/
    └── README-app.md
```
