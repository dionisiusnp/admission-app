# Dummy Test — Development Plan
## Vibe Coding & Venture SEVIMA
### Tema: Pengembangan Lanjutan Aplikasi PMB
### Waktu: 30 Menit | Output: File Markdown (.md)

---

## Tujuan Test Ini

Dalam vibe coding, kualitas output AI sangat bergantung pada **seberapa jelas kamu mendefinisikan apa yang ingin dibangun** sebelum mulai prompt. Test ini melatih kemampuan itu.

Kamu tidak diminta menulis kode. Kamu diminta membuat **Development Plan** — dokumen perencanaan teknis yang nantinya bisa langsung dijadikan prompt ke AI untuk menghasilkan fitur baru.

**Prinsipnya sederhana:** Semakin baik plan-mu, semakin sedikit iterasi yang kamu butuhkan saat vibe coding.

---

## Konteks: Sistem yang Sudah Ada

Aplikasi PMB (Penerimaan Mahasiswa Baru) **sudah berjalan** dengan stack **React 18 + Tailwind CSS** (frontend) dan **Laravel 12 + SQLite** (backend). Sistem saat ini sudah mencakup:

| Fitur yang Sudah Ada | Status |
|----------------------|--------|
| Formulir pendaftaran online | ✅ Selesai |
| Generate nomor pendaftaran otomatis | ✅ Selesai |
| Cek status pendaftaran (by nomor pendaftar) | ✅ Selesai |
| Dashboard admin dengan statistik per prodi & jalur | ✅ Selesai |
| Update status pendaftar (Menunggu / Lolos Seleksi / Tidak Lolos) | ✅ Selesai |
| Login admin dengan Sanctum token | ✅ Selesai |
| Export data pendaftar ke CSV | ✅ Selesai |
| Tombol heregistrasi untuk pendaftar yang lolos | ✅ Selesai |

Database sudah memiliki tabel `pendaftars` dan `users` yang aktif.

---

## Brief Pengembangan Selanjutnya

Kampus menyampaikan kebutuhan baru:

> *"Sistem pendaftaran sudah berjalan bagus. Sekarang kami butuh modul untuk mengelola jadwal tes seleksi dan wawancara PMB. Saat ini semua masih manual — panitia kirim jadwal lewat WhatsApp, peserta sering tidak tahu jadwal mereka, dan banyak yang tidak hadir karena informasi tidak sampai. Kami mau digitalisasi proses ini dan integrasikan dengan sistem yang sudah ada."*

Tugasmu: buat Development Plan untuk **menambahkan modul penjadwalan tes** ke sistem yang sudah ada, **lalu ubah plan tersebut menjadi prompt siap pakai untuk AI**.

---

## Instruksi Pengerjaan

1. Buat file baru bernama `devplan-[namamu].md`
2. Isi setiap bagian di bawah secara berurutan
3. Boleh menggunakan AI untuk membantu mengisi bagian tertentu — tapi prompt yang kamu buat **harus menggunakan formula 5 komponen**
4. Rancang fitur sebagai **tambahan** di atas sistem yang sudah ada — bukan dari nol
5. Submit file `.md` ke GitHub repository, kumpulkan linknya ke fasilitator

---

## Bagian yang Harus Diisi

---

### BAGIAN 1 — Analisa Teknis (20 poin)

Jawab pertanyaan-pertanyaan berikut secara singkat namun spesifik:

**1.1 Identifikasi Pengguna**
Siapa saja yang akan menggunakan **modul penjadwalan tes** ini? Sebutkan minimal 3 tipe pengguna dan jelaskan peran spesifik mereka dalam konteks penjadwalan.

> **Kisi-kisi:** Gunakan format tabel dengan kolom Pengguna dan Peran. Perhatikan bahwa beberapa pengguna (seperti admin dan calon mahasiswa) sudah ada di sistem — jelaskan peran *baru* mereka dalam modul ini. Nilai tinggi jika kamu mengidentifikasi pengguna yang spesifik untuk modul penjadwalan yang mungkin berbeda dari pengguna di modul pendaftaran.

**1.2 Fitur Utama per Pengguna**
Untuk setiap tipe pengguna di atas, list 3–5 fitur baru yang mereka butuhkan dalam modul penjadwalan ini.

> **Kisi-kisi:** Tulis fitur per pengguna secara terpisah. Fokus pada fitur yang **belum ada** di sistem saat ini — jangan list ulang fitur yang sudah berjalan (pendaftaran, cek status, dll.). Fitur yang baik adalah fitur yang menjawab masalah spesifik dari brief: jadwal tidak tersampaikan, peserta tidak hadir, informasi tidak terpusat.

**1.3 Tech Stack yang Dipilih**
Stack yang digunakan mengikuti sistem yang sudah ada: **React 18 + Tailwind CSS** (frontend) dan **Laravel 12** (backend). Jelaskan komponen tambahan apa yang akan kamu gunakan untuk modul ini dan alasannya.

> **Kisi-kisi:** Gunakan format tabel. Kamu tidak perlu mengganti stack utama — fokus pada keputusan teknis *tambahan* yang spesifik untuk modul ini. Contoh keputusan yang perlu dipikirkan: library kalender/datepicker, cara notifikasi, strategi pengiriman email/reminder, atau pustaka pendukung lainnya. Nilai tinggi jika pilihan tambahan diberikan alasan yang relevan dengan kebutuhan modul.

**1.4 Batasan & Asumsi**
Sebutkan minimal 3 batasan teknis atau asumsi yang kamu buat, khususnya terkait integrasi dengan sistem yang sudah berjalan.

> **Kisi-kisi:** Karena ini adalah pengembangan di atas MVP yang ada, batasan yang relevan berbeda dari membangun dari nol. Pikirkan: bagaimana modul baru ini berinteraksi dengan data `pendaftars` yang sudah ada? Apa yang diasumsikan tentang data pendaftar yang layak mendapat jadwal? Batasan apa yang sengaja dibuat agar tidak merusak fungsionalitas yang sudah berjalan?

---

### BAGIAN 2 — Bisnis Proses & Flow (25 poin)

Gambarkan alur proses bisnis menggunakan teks terstruktur (bukan gambar — tapi harus jelas dan bisa dibaca seperti flowchart).

**2.1 Flow Utama: Penjadwalan Tes Seleksi**
Gambarkan langkah-langkah dari admin membuat jadwal tes sampai pendaftar yang sudah **Lolos Seleksi** mendapatkan jadwalnya.

```
Gunakan format ini:
[AKTOR] → Aksi → [HASIL/KONDISI]
       ↓ jika [kondisi tertentu]
[AKTOR] → Aksi → [HASIL]
```

> **Kisi-kisi:** Perhatikan bahwa flow ini terhubung dengan data yang sudah ada — pendaftar berstatus "Lolos Seleksi" sudah tersimpan di database. Flow yang dinilai tinggi menunjukkan *titik integrasi* yang jelas: di mana modul baru ini membaca atau menulis data dari sistem lama? Sertakan peran sistem otomatis (assignment jadwal, notifikasi) sebagai aktor tersendiri.

**2.2 Flow Alternatif: Peserta Minta Reschedule**
Gambarkan alur jika peserta ingin meminta perubahan jadwal.

**2.3 Happy Path vs Error Path**
Untuk flow utama (2.1), sebutkan:
- Happy path: semua berjalan normal
- Error path: minimal 2 kondisi error dan bagaimana sistem meresponsnya

---

### BAGIAN 3 — Alur Data (20 poin)

Gambarkan bagaimana data bergerak di dalam sistem.

**3.1 Alur Data: Proses Penjadwalan**
Dari sumber data masuk sampai ditampilkan ke pengguna akhir.

```
Gunakan format ini:
[Sumber Data] → [Proses] → [Penyimpanan] → [Proses] → [Output ke Pengguna]
```

> **Kisi-kisi:** Alur data di sini harus mencerminkan bahwa ini adalah sistem yang sudah berjalan — bukan greenfield. Tunjukkan bagaimana data dari tabel `pendaftars` yang ada *dibaca* oleh modul baru ini. Nilai tinggi jika alur memperlihatkan titik baca/tulis lintas modul dengan jelas dan menyertakan nama layer teknis sesuai stack (React component, API endpoint, Laravel controller, tabel database).

**3.2 Alur Data: Peserta Cek Jadwal**
Dari request peserta sampai data jadwal tampil di layar.

**3.3 Data Apa yang Sensitif?**
Sebutkan field/data apa saja yang perlu perlakuan khusus (tidak boleh ditampilkan sembarangan, perlu enkripsi, dll.) dan alasannya.

---

### BAGIAN 4 — ERD / Desain Database (25 poin)

Rancang struktur database untuk sistem ini.

**4.1 Daftar Tabel**
List tabel baru yang perlu ditambahkan untuk mendukung modul penjadwalan.

> **Kisi-kisi:** Gunakan format tabel dengan kolom Nama Tabel dan Deskripsi. Ingat bahwa tabel `pendaftars` dan `users` **sudah ada** — jangan rancang ulang tabel yang sudah berjalan, cukup referensikan sebagai tabel yang sudah ada. Fokus pada tabel baru yang dibutuhkan. Nilai tinggi jika jumlah tabel proporsional dengan fitur yang direncanakan di Bagian 1.2 tanpa over-engineering.

**4.2 Struktur Tiap Tabel**
Untuk setiap tabel, sebutkan kolom, tipe data, dan constraint-nya.

> **Kisi-kisi:** Untuk setiap tabel di 4.1, buat tabel dengan kolom: Nama Kolom, Tipe Data, Constraint, dan Keterangan. Nilai tinggi diberikan jika: tipe data dipilih sesuai dengan jenis datanya (bukan semua VARCHAR), constraint mencerminkan aturan bisnis yang nyata (NOT NULL, UNIQUE, FK), dan ada konsistensi antar tabel (FK merujuk ke PK yang valid).

**4.3 Relasi Antar Tabel**
Gambarkan relasi antara tabel baru dengan tabel yang sudah ada.

```
Gunakan format ini:
[tabel_a] ---(tipe relasi)--- [tabel_b]
Keterangan: [jelaskan mengapa relasi ini ada dan bagaimana cara kerjanya]
```

> **Kisi-kisi:** Pastikan relasi ke tabel yang sudah ada (`pendaftars`, `users`) juga digambarkan — ini adalah titik integrasi penting. Nilai tinggi jika: tipe relasi dipilih dengan tepat, keterangan menjelaskan *mengapa* relasi itu ada dalam konteks bisnis proses, dan relasi konsisten dengan FK yang sudah didefinisikan di 4.2.

**4.4 Indexing**
Sebutkan kolom mana saja yang perlu diberi index dan alasannya.

> **Kisi-kisi:** Fokus pada kolom yang sering digunakan sebagai filter atau kondisi pencarian (bukan semua kolom). Berikan alasan teknis yang konkret untuk setiap index yang kamu usulkan. Nilai lebih jika mempertimbangkan kolom FK dan kolom yang digunakan di klausa WHERE atau JOIN yang sering terjadi.

---

### BAGIAN 5 — Prompt Siap Pakai untuk AI (10 poin)

Ini bagian paling penting — ubah semua plan di atas menjadi **satu prompt terstruktur** yang bisa langsung kamu kirim ke Claude atau AI lain untuk mulai generate kode fitur baru ini.

Prompt ini harus menggunakan **formula 5 komponen**, merangkum semua keputusan dari Bagian 1–4, dan **menyertakan konteks sistem yang sudah ada** agar AI tidak membangun dari nol.

Struktur wajib menggunakan 5 komponen berikut: `[KONTEKS]`, `[TUJUAN]`, `[FITUR]`, `[CONSTRAINT]`, `[TAMPILAN]`.

> **Kisi-kisi:** Prompt yang baik adalah prompt yang *self-contained* dan *context-aware* — AI harus langsung tahu bahwa ini adalah pengembangan lanjutan, bukan project baru. Nilai tinggi diberikan jika:
> - `[KONTEKS]` menyebutkan stack dan fitur yang sudah ada secara ringkas sehingga AI tidak mereset ulang arsitektur
> - `[TUJUAN]` mendefinisikan scope modul baru secara spesifik dan terukur
> - `[FITUR]` berisi daftar fitur baru yang actionable (bukan pengulangan fitur lama)
> - `[CONSTRAINT]` menyebutkan struktur tabel baru, batasan integrasi dengan data lama, dan konvensi kode yang sudah dipakai
> - `[TAMPILAN]` memberikan arahan visual yang konsisten dengan UI yang sudah ada
>
> Uji mandiri: *"Jika prompt ini dikirim ke AI sekarang, apakah hasilnya akan menyambung dengan sistem yang sudah berjalan tanpa merusak fitur lama?"*

---


### BAGIAN 6 — Jalankan Prompt & Evaluasi Hasil ⭐ Bonus (+20 poin)

> Bagian ini adalah bonus. Kerjakan setelah Bagian 1–5 selesai.
> Tujuannya adalah **menutup loop vibe coding secara utuh**: dari plan → prompt → aplikasi → refleksi.

---

**6.1 Jalankan Prompt dari Bagian 5**

Copy prompt yang sudah kamu buat di Bagian 5, kirim ke AI (Claude, ChatGPT, atau Cursor).
Jalankan hasilnya di browser **bersama dengan sistem yang sudah ada** — pastikan fitur baru tidak merusak fitur lama (pendaftaran, cek status, dashboard admin).
Jika ada error atau hasil tidak sesuai, lakukan iterasi dengan prompt lanjutan.

Catat semua prompt yang digunakan (termasuk iterasi) di file `devplan-[namamu].md`. Gunakan heading yang jelas untuk membedakan prompt utama dan setiap iterasi. Untuk setiap iterasi, tuliskan *mengapa* iterasi itu diperlukan (apa yang kurang atau tidak sesuai dari hasil sebelumnya).

> **Kisi-kisi:** Nilai bonus di bagian ini bukan dari banyaknya iterasi, melainkan dari *kualitas analisis* kamu — apakah kamu bisa mengidentifikasi dengan tepat mengapa hasil AI tidak sesuai dan bagaimana kamu memperbaiki prompt-nya? Poin tambahan jika kamu berhasil menunjukkan integrasi yang mulus antara modul baru dan fitur yang sudah ada.

---

**6.2 Evaluasi Kesesuaian dengan Development Plan**

Setelah modul baru berjalan di browser, bandingkan hasilnya dengan plan yang sudah kamu buat. Buat tabel evaluasi sendiri yang mencakup poin-poin dari Bagian 1.2 (fitur baru), Bagian 2 (flow), dan Bagian 4 (tabel baru). Tandai setiap poin dengan status kesesuaian (✅ / ⚠️ / ❌).

Tambahkan juga baris khusus untuk mengevaluasi **tidak adanya regresi** — apakah fitur lama (pendaftaran, cek status, dashboard) masih berjalan normal setelah penambahan modul baru.

> **Kisi-kisi:** Evaluasi yang baik adalah evaluasi yang *jujur dan reflektif* — bukan sekadar mencentang semua ✅. Nilai tinggi diberikan jika:
> - Tabel evaluasi mencakup semua fitur baru dari plan (bukan hanya yang berhasil)
> - Catatan untuk status ⚠️ dan ❌ menjelaskan *perbedaan spesifik* antara plan dan hasil
> - Kesimpulan menganalisis *mengapa* perbedaan itu terjadi (bukan hanya menyebutkan bahwa ada perbedaan)
> - Ada evaluasi eksplisit bahwa integrasi dengan sistem lama tidak menimbulkan masalah

---

**6.3 Push Hasil ke Repo yang Sama**

Upload hasil vibe coding ke **repository GitHub yang sama** dengan file devplan.
Struktur repo akhir yang diharapkan:

```
nama-repo/
├── devplan-[namamu].md        ← development plan + log prompt + evaluasi
└── app/
    ├── pmb-frontend/          ← fork/extend dari sistem yang sudah ada
    ├── pmb-backend/           ← fork/extend dari sistem yang sudah ada
    └── README-app.md          ← instruksi menjalankan + deskripsi fitur baru
```

`README-app.md` wajib berisi:
- Cara menjalankan aplikasi (langkah-langkah atau perintah)
- Fitur **baru** mana yang sudah berjalan dan mana yang belum
- Konfirmasi bahwa fitur lama tetap berjalan normal
- Screenshot atau deskripsi singkat tampilan modul penjadwalan


---

## Format File yang Dikumpulkan

```
devplan-[namamu].md
```

Isi file mengikuti struktur di atas, dengan semua bagian terisi (Bagian 1–5 wajib, Bagian 6 bonus).
Upload ke GitHub repository (public) — satu repo berisi devplan + folder `app/` (jika mengerjakan Bagian 6).
Kumpulkan link repository ke fasilitator.