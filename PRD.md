# PRD — Portal Web Beasiswa GBB (v2)

> **Version**: 2.1 — MVP | **Date**: 19 Jun 2026 | **Status**: 🟢 Approved

### Cara membaca dokumen ini

PRD ini mendefinisikan **apa & mengapa** (produk, flow, business rules). Detail teknis **bagaimana** ada di dokumen pendamping:

| Dokumen | Isi |
|---------|-----|
| `PRD.md` (ini) | Produk, user flow, business rules, NFR, scalability, quality harness |
| `implementation_plan.md` | Tech stack, arsitektur, fase build, struktur folder, cron, file limits, verifikasi |
| `docs/erd.dbml` | Skema database (31 tabel, 10 group) — buka di [dbdiagram.io](https://dbdiagram.io) |
| `docs/wireframes-*.md` | Wireframe per portal (Internal/Beswan/Donatur) |
| `docs/colorpalette.md` | Design system & token warna (light theme, selaras baikberdampak.org) |
| `docs/seeds/` | Data awal (seed) — mis. 17 pertanyaan refleksi |

**Untuk engineer**: baca §1–8 untuk konteks produk, lalu §12 (NFR), §13 (Scalability), §14 (Quality Harness) sebagai acuan teknis & Definition of Done sebelum mulai tiap fase.

### Sumber Data Eksternal — Google Sheets / Form

Sebagian dashboard di-feed dari Google Sheets/Form, di-**mirror read-only** ke PostgreSQL via Google Sheets API v4 + Service Account, sync tiap 15 menit. Engineer butuh akses **Viewer** ke 3 sumber ini di awal:

| # | Sumber | Tipe | Target tabel | Mode sync | Link |
|---|--------|------|--------------|-----------|------|
| 1 | Database / Pendaftaran Donatur | Google Sheet | `donatur` | Mirror (profil s/d kolom *Akun Media Sosial*) | _(isi link)_ |
| 2 | Pendaftar Event GBB (Growth) | Google Form → Responses | `pendaftar_event` | Mirror (semua kolom) | _(isi link)_ |
| 3 | Feedback Talkshow | Google Sheet | `feedback_event` | Mirror (semua kolom) | _(isi link)_ |

> Service Account harus di-share sebagai **Viewer** ke ketiga sumber. Detail strategi mirroring & migrasi data: `implementation_plan.md` → Google Sheets Mirroring Strategy & Data Migration Plan.

---

## 1. Latar Belakang

Yayasan Gerakan Baik Berdampak (AHU-0010854.AH.01.04.Tahun 2024) menjalankan program **Beasiswa Baik Berdampak (BBB)** untuk mahasiswa aktif Undip. Didirikan oleh 10 alumni muda Undip — motto **#10AlumniBantuSatu** — alumni menyisihkan mulai Rp 100.000/bulan.

Beswan juga mendapat **talkshow, penugasan, dan program pengembangan diri**.

---

## 2. Visi Produk

Portal web terpusat di `portal.baikberdampak.org` yang:
1. Memudahkan pekerjaan Tim Finance & AnC
2. Memusatkan seluruh data, brief, materi & kegiatan
3. Memberikan transparansi keuangan & progress kepada donatur
4. Menjadi single platform bagi beswan berinteraksi dengan program GBB

---

## 3. User Roles

| Role | `users.role` | Deskripsi | Portal | Auth Method |
|------|--------------|-----------|--------|-------------|
| **Super Admin** | `admin` | Tim inti GBB — full access semua modul | Internal | Email + Password |
| **Admin Program (PCM)** | `pcm` | Tim Program & Kurikulum | Internal | Email + Password |
| **Admin Finance** | `finance` | Tim Keuangan | Internal | Email + Password |
| **Admin AnC** | `anc` | Alumni & Collaboration (donatur & mentor) | Internal | Email + Password |
| **Viewer** | `viewer` | Lihat dashboard internal, tanpa akses edit | Internal | Email + Password |
| **Beswan** | — | Penerima beasiswa aktif (tabel `beswan`) | Beswan | Email + Password (generated) |
| **Donatur** | — | Alumni donatur | Donatur | Gmail OAuth |
| **Mentor** | — | Mentor event & 1-on-1 | - | Tidak login, data dikelola internal |

> Nilai `users.role` di atas adalah identifier baku di DB & RBAC server. Label panjang ("Super Admin", dll) hanya untuk display.

### Matriks Akses per Modul (Portal Internal)

`✏️ Full` = lihat + ubah · `👁 View` = read-only · `—` = tidak ada akses

| Modul | Super Admin | Admin Program (PCM) | Admin Finance | Admin AnC | Viewer |
|-------|:-----------:|:-------------------:|:-------------:|:---------:|:------:|
| Dashboard (3 tab) | ✏️ | 👁 | 👁 | 👁 | 👁 |
| Konfigurasi Periode | ✏️ | 👁 | — | — | — |
| Beswan (data/absensi/rapor) | ✏️ | ✏️ | — | 👁 | 👁 |
| Kurikulum & Library | ✏️ | ✏️ | — | 👁 | 👁 |
| Mentor | ✏️ | ✏️ | — | ✏️ | 👁 |
| Event | ✏️ | ✏️ | — | 👁 | 👁 |
| Penugasan & Penilaian | ✏️ | ✏️ | — | — | 👁 |
| Keuangan — Rekonsiliasi & Cashflow | ✏️ | — | ✏️ | 👁 | — |
| Keuangan — Overview Keuangan | ✏️ | — | ✏️ | 👁 | 👁 |
| Database Donatur | ✏️ | — | 👁 | ✏️ | 👁 |
| Monitoring Donatur | ✏️ | — | 👁 | ✏️ | 👁 |
| Laporan & Booklet | ✏️ | upload | upload | upload | 👁 |
| Settings — Users & role | ✏️ | — | — | — | — |
| Settings — Template pesan WA | ✏️ | — | — | ✏️ | — |
| Settings — Master kategori cashflow | ✏️ | — | ✏️ | — | — |
| Settings — Konfigurasi AI | ✏️ | — | — | — | — |

> **Default proposal — mohon dikoreksi bila perlu.** Prinsipnya: PCM pegang sisi program (beswan/kurikulum/event/penugasan), Finance pegang keuangan, AnC pegang donatur & mentor, Viewer read-only. RBAC ditegakkan di **server**, bukan sekadar disembunyikan di UI (lihat §12).

---

## 4. Arsitektur Portal

```
portal.baikberdampak.org
├── /internal   → Portal Internal (Tim GBB)
├── /beswan     → Portal Beswan (Penerima)
└── /donatur    → Portal Donatur (Alumni)

Single login → diarahkan ke portal sesuai role
```

---

## 5. FLOW PORTAL — Actor 1: Tim Internal

### Dashboard Internal (3 tab)

#### Tab: Event
- Metric cards: Total Event, Selesai, Beswan Aktif, Donasi Bulan Ini
- Chart: Event per Bulan (bar), Kehadiran Beswan (pie + progress), Penugasan Overview
- Sumber data: tabel `event`, `event_beswan`, `penugasan`, `cashflow`

#### Tab: Trend Donatur
- Metric cards: Total Estimasi Komitmen, Total Calon Donatur
- Chart: Tren Pendaftaran (line), Jenis Beasiswa (pie), Kriteria Penerima (bar), Pekerjaan (bar), Saluran Info (bar), Alumni per Angkatan (bar), Skema Donasi (pie), Faktor Ragu (bar)
- Sumber data: Google Sheets — Database/Pendaftaran Donatur (di-mirror ke tabel `donatur`)

#### Tab: Growth
- Metric cards: Total Peserta Teregistrasi, Berminat Berkontribusi, Calon Mentor, Calon Donatur
- Chart: Distribusi Profesi (pie), Minat Kontribusi per Angkatan (bar), Tren Tema Diminati (bar), Bidang Keahlian Ditawarkan (bar), Universitas Asal (pie), Saluran Informasi (pie), Pengenalan GBB per Prodi (donut)
- Sumber data: Google Form — Pendaftar Event GBB (di-mirror ke tabel `pendaftar_event`)

### FASE 1 — Setup Periode (sekali per batch)

#### Step 1.1 — Konfigurasi Periode
- Klik "+ Tambah" → sistem auto-suggest nama batch, semester, tanggal mulai/selesai berdasarkan periode terakhir (e.g. jika terakhir BBB4 Jan–Jun 2026, suggest BBB5 Jul–Des 2026)
- Semua field bisa diedit sebelum save
- Input: nama batch (e.g. BBB5), tanggal mulai, tanggal selesai
- Output: periode aktif tersimpan → acuan semua modul
- Tabel view: ID (auto-increment), Nama (display: `BBB #4 (Jan–Jun 2026)`), Mulai, Selesai, Status (Aktif/Selesai), Aksi (edit/hapus)
- Rule: bisa **>1 periode aktif bersamaan** (jika pembinaan molor)
- Rule: periode hanya 2 opsi: **Jan–Jun** (semester 1) atau **Jul–Des** (semester 2)
- Rule: `periode.nama` yang tersimpan di DB adalah versi pendek (e.g. `BBB5`), display format panjang di-generate dari nama + semester + tahun

#### Step 1.2 — Input Data Beswan
- Input per beswan: nama lengkap, NIM, email, HP, foto, CV
- Assign ke periode aktif → status aktif
- Sistem generate kredensial login (email + password default)
- Rule: 1 beswan bisa >1 periode (multi-batch)

#### Step 1.3 — Susun Kurikulum
- Input per topik: urutan, goal, topik, detail topik, TOR (file/link)
- Per periode, status awal: **planned**
- Topik menjadi pilihan saat membuat event

### FASE 2 — Operasional Event

#### Step 2.0 — Input/Update Database Mentor
- Input: nama, email, HP, LinkedIn, CV, bidang keahlian
- Flag `is_internal = true` jika mentor adalah anggota GBB
- Data tampil di Portal Beswan (tanpa HP/CV)

#### Step 2.1 — Buat Event
- Pilih periode → pilih topik kurikulum (opsional, bisa non-kurikulum)
- Input: nama, tipe (talkshow/growth/other), format (online/offline/hybrid), lokasi/link, tanggal, jam, kapasitas, deskripsi
- Assign mentor (bisa >1, peran: speaker/moderator/fasilitator)
- Status awal: **upcoming**

#### Step 2.2 — Event Berlangsung
- Absensi dicatat oleh tim internal (hadir/tidak hadir)
- Peserta hadir → row terbuat di `event_beswan`

#### Step 2.3 — Pasca Event
- Upload link rekaman YouTube + slide materi
- Status event → **done**
- Status kurikulum terkait → **done** (youtube & slide otomatis muncul)
- Alert jika event done tapi rekaman/slide belum diisi

#### Step 2.4 — Materi Masuk Library
- Slide/materi dari event otomatis masuk Library
- AI summary + auto-tagging topik — **provider AI dikonfigurasi di Settings → Konfigurasi AI** (multi-provider via API key; lihat `implementation_plan.md`). Kalau API gagal/timeout, materi **tetap tersimpan tanpa summary** (tidak blocking) + bisa retry manual
- Tim bisa edit tag manual
- Library juga bisa diisi upload manual (materi luar event)

#### Step 2.5 — Buat Penugasan
- Pilih batch → pilih event sumber (opsional, bisa non-event)
- Input: judul, deskripsi/soal, deadline
- Muncul di Portal Beswan untuk semua beswan periode tersebut
- Status awal per beswan: **belum_kumpul**

#### Step 2.6 — Penilaian Penugasan
- Beswan submit → status **submitted**
- PCM buka Hasil Penugasan → master-detail view
- Input per beswan: nilai (angka) + feedback (teks)
- Status → **graded**, notifikasi ke beswan

#### Step 2.7 — Rapor & Absensi
- Generate rapor per beswan per periode:
  - Absensi: hadir/tidak per event
  - Tugas: nilai & status per penugasan
  - Refleksi: sudah/belum submit per bulan
- Referensi contoh: Binar Academy

### FASE 3 — Keuangan (Finance)

#### Step 3.1 — Upload Bank Statement BSI
- Upload file .xlsx mutasi BSI (multi-sheet: Sept, Agt, dll). **Semua sheet diproses sekaligus**; pengelompokan bulan murni dari **tanggal transaksi**, bukan nama sheet (jadi transaksi tetap benar walau sheet & isinya tidak rapi)
- **Struktur file BSI**: baris 1–10 = metadata rekening (Account, Date Period, Opening/Closing Balance, total debit/kredit, Branch). **Baris 12 = header kolom**, **data mulai baris 13**. Ada blok ringkasan manual di kolom kanan (mulai kolom K, "SUMMARY …") yang **harus diabaikan** parser
- **Pemetaan kolom** (A–I): `Date` · `FT Number` · `Description` · `Currency` · `Amount` · `DB` · `CR` · `Balance` · `Category`
- **Tipe In/Out dari file (bukan diedit)**: kolom **CR** terisi → `cash_in`, kolom **DB** terisi → `cash_out`. Tipe **dikunci** mengikuti file — tidak bisa diubah manual (keterangan masuk/keluar sudah pasti dari bank)
- Bulan transaksi diturunkan dari **tanggal transaksi** (kolom Date), bukan dari nama sheet
- **Deduplication: FT Number + Nominal** (kombinasi keduanya), pengecekan **antar-upload** (file baru vs data tersimpan) — tidak ada duplikat dalam 1 file. Baris yang sudah ada **tetap ditampilkan & dihitung di kartu DUPLIKAT, tapi TIDAK disimpan**. Tanpa override manual (kombinasi FT Number + Nominal sama = pasti transaksi sama). Baris yang FT Number-nya bukan format FT (mis. Biaya Administrasi pakai `7280955285.11.202509`) → fallback dedup: tanggal + nominal + deskripsi
- **Parser tahan format & multi-bank (target ke depan)**: MVP minimal BSI; arsitektur disiapkan agar bank lain bisa ditambah (profil pemetaan kolom per bank), plus penanganan error kalau file bukan format yang dikenali / sheet kosong (lihat "Parsing Engine" di `implementation_plan.md`)

#### Step 3.2 — Klasifikasi Cashflow
- **Auto klasifikasi kategori (semua baris)**: kolom **`Category` di file sudah terisi** untuk tiap transaksi (`Donasi`, `Bagi Hasil`, `Biaya Administrasi Perbankan`, `Program`, `Biaya Transaksi`, `Pajak`, dll) → sistem memetakannya ke kategori internal (master `cashflow_kategori`) secara otomatis. **Tetap bisa dikoreksi manual**
- **Jenis cash in**: uang masuk bisa **Donasi**, **Pengembalian**, atau **Bagi Hasil** (bunga bank) — ditentukan dari kolom Category, bisa dikoreksi. **Hanya baris berkategori `Donasi` yang butuh nama donatur**; Pengembalian / Bagi Hasil cukup kategori
- **Auto-match nama donatur (khusus cash in = Donasi)**: deskripsi BSI umumnya **cukup lengkap, kadang nama terpotong sedikit**. Sistem mencocokkan deskripsi ("Trf Dari - …", "QR … BAIK BERDAMPAK") dengan nama donatur di DB — termasuk **nama terpotong** (prefix/sebagian, normalisasi spasi & kapital). Aturan:
  - **Tepat 1 donatur cocok** → `donatur_id` terisi otomatis, `match_source = auto`
  - **Lebih dari 1 cocok / tidak ada** → biarkan `unknown` (tim isi manual)
  - Hasil auto-match **tetap bisa dikoreksi manual** (`match_source = manual`)
  - Donasi via QR/QRIS umumnya tanpa nama → tim pilih donatur atau **"Anonymous"**
  - Kandidat match = **semua donatur di database**, termasuk yang non-aktif / berskema "belum bersedia" (tidak dikecualikan — kalau memang dia transfer, biar tetap ke-match)
- **Klasifikasi manual**:
  - Cash In = Donasi → ketik/pilih nama donatur, **atau pilih "Anonymous"** untuk donasi tanpa identitas (flag `is_anonymous`, **tidak pernah auto-set** — selalu manual)
  - Cash In = Pengembalian / Bagi Hasil, & semua Cash Out → pilih/koreksi kategori (& sub-kategori bila ada) dari master
  - **Catatan (opsional)**: tim finance bisa menambah notes internal di baris mana pun (mis. konteks pengembalian, penjelasan transaksi), tidak wajib & tidak memengaruhi status
- **Status klasifikasi (hanya 2):**

  | Status | Kondisi |
  |--------|---------|
  | `inputted` | Kategori sudah terisi **dan** (khusus cash in = Donasi) donatur sudah dipilih atau ditandai Anonymous |
  | `unknown` | Belum diklasifikasi (kategori belum ter-set, atau Donasi tapi donatur belum dipilih) |

  > Tidak ada status "pending". Sebuah baris hanya: sudah (`inputted`) atau belum (`unknown`). Karena kolom Category file hampir selalu terisi, sebagian besar baris langsung `inputted`; yang tersisa biasanya baris **Donasi** yang perlu nama donatur.
- **Simpan**: **wajib 100% baris `inputted`**. Setiap transaksi pasti bisa diklasifikasi (cash in = donasi/pengembalian/bagi hasil; cash out = kategori dari file), jadi tidak ada baris yang "tidak bisa diklasifikasi"
- Simpan → masuk database Cashflow → otomatis muncul di Dashboard & Monitoring Donatur

#### Step 3.3 — Edit, Koreksi & Hapus Cashflow (setelah rekon)
- Transaksi yang sudah disimpan **tetap bisa diedit**, tapi **hanya untuk koreksi klasifikasi** (kesalahan tim ada di klasifikasi, bukan di data bank)
- **Bisa diedit**: nama donatur, status Anonymous, kategori, sub-kategori, **catatan** (notes internal tim finance — opsional, terpisah dari deskripsi bawaan bank)
- **Dikunci (mengikuti file BSI)**: **tipe (in/out)**, FT Number, nominal, tanggal — data mentah dari bank, tidak boleh diubah. Tipe in/out tidak bisa dibalik karena keterangan CR/DB sudah pasti dari file. Kalau data bank-nya sendiri keliru → perbaiki via upload ulang
- **Konsistensi saat ganti kategori**: kalau kategori cash in diubah dari `Donasi` ke non-Donasi (mis. Pengembalian), field donatur/Anonymous otomatis dikosongkan (donatur hanya relevan untuk Donasi)
- **Hapus baris**: bisa (untuk koreksi kesalahan yang jarang terjadi), **dengan konfirmasi pop-up** ("Yakin mau hapus transaksi ini?") dan **bisa di-undo** setelah dihapus
- Setiap edit mencatat **siapa & kapan** (`updated_by`, `updated_at`) — cukup penulis terakhir, **tanpa riwayat perubahan penuh**. Perubahan donatur otomatis ter-refleksi di Monitoring Donatur & Dashboard

#### Step 3.4 — Master Kategori Cashflow (bisa diedit)
- Daftar kategori & sub-kategori dikelola lewat tombol **Kategori** (bukan hardcode) — bisa **tambah/edit/hapus/nonaktifkan**
- Tiap kategori bertipe `cash_in` / `cash_out` / `both`. Seed awal (dari file + tampilan):
  - **Cash in**: Donasi, Pengembalian, Bagi Hasil
  - **Cash out**: Beasiswa, Program, Pajak, Biaya Transaksi, Biaya Administrasi Perbankan, Biaya Operasional (sub: Administrasi, Pembinaan, dst.)
- Master ini juga jadi acuan pemetaan dari kolom `Category` file BSI → kategori internal (auto-klasifikasi) & pengelompokan di laporan/Dashboard

**Hak akses fitur Keuangan** (detail dari matriks §3):

| Role | Upload | Klasifikasi & Edit | Hapus baris | Kelola master kategori |
|------|--------|--------------------|-------------|------------------------|
| **Super Admin** (`admin`) | ✅ | ✅ | ✅ | ✅ |
| **Admin Finance** (`finance`) | ✅ | ✅ | ✅ (dengan konfirmasi) | ✅ |
| **Admin AnC** (`anc`) | ❌ | ❌ (👁 view, untuk lihat donasi masuk) | ❌ | ❌ |
| **PCM / Viewer** | ❌ | ❌ | ❌ | ❌ |

> **Catatan istilah**: "Rekonsiliasi" di sini = proses **klasifikasi** transaksi bank (bukan pencocokan saldo buku vs bank). Tidak ada validasi Opening/Closing Balance — kolom `Balance` di file tidak disimpan. Cukup mengelompokkan tiap transaksi.

#### Step 3.5 — Overview Keuangan (internal)

> Halaman ringkasan keuangan untuk tim Finance — sub-menu di bawah Keuangan. Sumber data: `cashflow` yang sudah `inputted`. **Murni laporan/agregasi, read-only** (bukan input).

**Filter**: rentang waktu (periode / rentang bulan / tahun).

**Metric Cards (4)**:
- Total Masuk (sum cash in pada rentang)
- Total Keluar (sum cash out pada rentang)
- **Net** (Masuk − Keluar) untuk rentang tsb
- Jumlah Transaksi (count)

> Tidak menampilkan "saldo kas" absolut — Opening Balance tidak disimpan (lihat catatan rekonsiliasi). Yang disajikan = arus & komposisi, bukan saldo bank.

**Chart & Tabel**:
- **Tren Masuk vs Keluar per bulan** (bar/line ganda)
- **Komposisi Pemasukan** per jenis: Donasi / Pengembalian / Bagi Hasil (pie) + porsi Anonim vs teridentifikasi
- **Komposisi Pengeluaran** per kategori (pie/bar, dari master `cashflow_kategori`)
- **Tabel ringkasan per kategori**: nominal + % terhadap total, bisa di-drilldown ke daftar transaksinya

**Akses**: Super Admin & Admin Finance (✏️/👁), AnC & Viewer (👁) — lihat matriks §3.

### FASE 4 — Alumni & Collaboration (AnC)

#### Step 4.1 — Monitoring Donatur

> Matriks keikutsertaan patungan + link pesan WhatsApp siap kirim

**Metric Cards (4)**:
- Total Donatur (semua donatur di database)
- Aktif Periode Ini (donatur yang sudah dikonfirmasi untuk periode aktif)
- Belum Periode Ini (donatur yang belum dikonfirmasi/belum aktif di periode ini)
- Periode Aktif (nama periode yang sedang berjalan, e.g. BBB #4 Jan–Jun 2026)

**Filter Bar**:
- Search: cari nama atau kode donatur
- Dropdown Periode: Semua Periode | BBB #1 (Jul–Des 2024) | BBB #2 (Jan–Jun 2025) | dst
- Dropdown Status: Semua Status | Aktif Periode Ini | Belum Periode Ini
- Dropdown Tag: Semua Tag | Sudah Konfirmasi | Donatur Setia | Baru Bergabung | Perlu Follow-Up | Sulit Dihubungi | Sementara Berhenti | Sudah Ditandai

**Tabel Matriks**:

| Kolom | Detail |
|-------|--------|
| **Tandai** | Checkbox per row + tombol Reset. Centang tersimpan di DB (persisten), muncul di semua layar/user, tidak hilang kecuali di-uncheck manual. Digunakan admin untuk menandai donatur saat proses reminder |
| **Nama & Tag** | Nama lengkap, kode donatur (e.g. DLW12025), skema donasi (e.g. "Sebulan sekali Patungan"), badge tag jika ada (e.g. "Donatur Setia") |
| **Kirim** | Satu tombol "Kirim" per donatur → buka picker template pesan → pilih template (reminder, follow-up, dll) → buka `wa.me` link dengan pesan pre-filled (placeholder otomatis terisi: nama, kode, bulan). Menggantikan 2 kolom lama (Tgl 7 & Tgl 25) |
| **Periode Terakhir** | Periode terakhir di mana donatur aktif (e.g. "BBB #2 Jan–Jun 2025"), "—" jika belum pernah |
| **Kolom Bulan** | Kolom dinamis per bulan dalam periode yang dipilih (e.g. Agt 25, Sep 25, ..., Mei 26). Isi: nominal donasi dari cashflow (e.g. "100k", "200k") atau "—" jika belum ada |
| **Total** | Total donasi dalam periode filter (e.g. "Rp 700.000") |
| **Catatan** | Icon catatan per donatur — klik untuk tambah/edit catatan internal |

> **Baris "Anonim"**: donasi cash in yang ditandai Anonymous **tetap masuk matriks** sebagai satu baris agregat khusus ("Anonim"), dipecah per kolom bulan dari tanggal transaksi — supaya tim tahu berapa donasi anonim per bulan. Baris ini tanpa kode donatur, tanpa tombol Kirim, dan tidak bisa di-tag/di-tandai.

> **Sumber data kolom bulan**: dari tabel `cashflow` (cash in yang sudah `inputted`). Bulan ditentukan dari **tanggal transaksi**. `donatur_periode` dipakai admin untuk menandai donatur ini termasuk periode mana (klasifikasi manual) — bukan untuk menentukan masuk bulan apa.

**Info Banner**: Panduan penggunaan link WA — "Klik tombol Kirim, pilih template pesan (reminder, follow-up, dll), lalu WhatsApp terbuka dengan pesan siap kirim"

**WhatsApp Link (wa.me)** — implementasi MVP:
- Bukan WA API — hanya `wa.me/{nomorHP}?text={pesan}` yang membuka WhatsApp di device user
- Pesan pre-filled otomatis berisi: salam, nama donatur, bulan, instruksi transfer (lewat placeholder)

**Template Pesan (multi-template, editable)**:
- Admin bisa **buat, simpan, edit, hapus, dan re-order banyak template** — tidak lagi terbatas 2 template tetap
- Setiap template: nama (e.g. "Reminder Awal Bulan", "Follow-up Belum Transfer") + isi pesan dengan placeholder `{nama}`, `{kode}`, `{bulan}`, `{bulan_berikutnya}`, `{nominal}`
- Dikelola di Settings/Konfigurasi; bisa set 1 template sebagai default
- Seed awal = 2 template (reminder & follow-up), migrasi dari desain lama
- Tombol **Kirim** (1 kolom): klik → pilih template → `wa.me` terbuka dengan pesan terisi
- **Emoji wajib didukung & terbaca**: kolom template UTF-8 (`text`), editor menerima emoji, dan saat membangun link `wa.me/{hp}?text=...` pesan di-encode dengan `encodeURIComponent` agar emoji, baris baru, dan karakter khusus tidak rusak di WhatsApp

> **Next step (future — kalau memungkinkan)**: integrasi WhatsApp langsung (bukan sekadar `wa.me` link). AnC login WhatsApp **sekali** di portal, lalu kirim pesan ter-template langsung dari portal (mis. via WhatsApp Web session / WA Business API). Di luar scope MVP — dieksekusi belakangan.

**Tag System**:

| Tag | Deskripsi |
|-----|-----------|
| ✅ Sudah Konfirmasi | Donatur sudah konfirmasi ikut periode ini |
| ⭐ Donatur Setia | Donatur konsisten >3 periode berturut-turut |
| 🆕 Baru Bergabung | Donatur baru join di periode ini |
| 🔴 Perlu Follow-Up | Belum ada kabar, perlu di-follow up |
| ⚠ Sulit Dihubungi | Sudah di-follow up tapi tidak merespon |
| ⏸ Sementara Berhenti | Donatur konfirmasi pause sementara |
| ☑ Sudah Ditandai | Generic flag dari checkbox bulk marking |

- Tag di-assign manual oleh AnC (klik atau bulk via checkbox)
- 1 donatur bisa punya >1 tag
- Tag tersimpan per donatur (bukan per periode)

**Pagination**:
- Footer: "Menampilkan X dari Y donatur" + "Halaman N dari M" + navigasi `< >`
- Default 25 row per halaman

**Skema Donasi** (values di kolom Nama & Tag):
- Sebulan sekali Patungan
- 1 Semester sekali Patungan
- Sebulan sekali donasi
- 1 Semester sekali donasi
- Belum bersedia

#### Step 4.2 — Upload Laporan & Booklet
- Upload PDF laporan/booklet GBB
- Flag `is_public` (tampil di Portal Donatur)
- Attach ke periode terkait

---

## 6. FLOW PORTAL — Actor 2: Beswan

#### Step B1 — Login
- Email + password dari tim GBB
- Pertama kali → lengkapi profil (HP wajib, CV opsional, **IPK semester berjalan wajib** — lihat B9)

#### Step B2 — Beranda
- Greeting otomatis tiap hari (if-else 5 variasi, tanpa AI)
- My Events (aktif & history)
- My Tasks (pending & history)
- Notifikasi: tugas baru, tugas dinilai, pengingat refleksi, **pengingat update IPK semester**

#### Step B3 — Ikut Event
- Event berdasarkan periode aktif
- Setelah selesai: tombol slide & rekaman (disabled jika belum ada)
- Absensi dicatat tim internal

#### Step B4 — Upload Penugasan
- List: judul, event sumber, deadline, status
- Upload file jawaban (PDF/dokumen)
- Submit → **submitted** (tidak bisa edit ulang)
- Jika dinilai → tampil nilai & feedback

#### Step B5 — Library Materi
- Search & filter by topik/tag
- Fitur "Usulan Topik Materi" → masuk database untuk review PCM

#### Step B6 — Database Mentor
- Tampil: nama, bidang keahlian, history event
- Tidak tampil: HP, CV → "hubungi tim Program GBB"
- Request 1-on-1:
  - Sudah pilih mentor → request langsung
  - Belum tahu → form curhat/clue → tim GBB matching

#### Step B7 — Refleksi Bulanan (wajib)
- Cek berdasarkan start_date–end_date periode aktif
- Alert jika bulan ini belum diisi
- Form: 17 pertanyaan default dari PCM & AnC (Identitas, Aktivitas Bulanan, Refleksi & Evaluasi, Rencana & Masukan, Dokumentasi) — seed, editable admin. Detail di `implementation_plan.md` → Refleksi Bulanan Beswan — Form
- Termasuk pertanyaan bersyarat (mis. nama lomba muncul jika ikut lomba) & skala kepuasan 1–10
- Upload **dokumentasi kegiatan bulan ini** (wajib — 1 field multi-file, foto/PDF). Transkrip nilai **tidak lagi di sini** → pindah ke update IPK (B9)
- Dropdown bulan untuk lihat/edit sebelumnya

#### Step B8 — Input Prestasi
- Input: judul, kategori (studi/organisasi/luar kampus), tanggal, deskripsi
- Upload sertifikat/foto (wajib, multi-file)
- Alert tiap akhir kuartal jika belum update
- Data tampil di Portal Donatur

#### Step B9 — Update IPK (per semester, wajib)
- Di halaman **Profile**: input **IP semester berjalan** + **IPK kumulatif** + upload **transkrip** (bukti)
- **Wajib diperbarui tiap semester** (1 entri per periode aktif). Alert di Beranda + email reminder jika belum update di semester berjalan
- IPK terbaru jadi sumber metric **"Avg IPK"** (Database Beswan internal) & angka IPK di Portal Donatur — menggantikan data yang sebelumnya tak punya sumber
- Tabel: `beswan_ipk` (1 baris per beswan × periode)

---

## 7. FLOW PORTAL — Actor 3: Donatur

#### Step D1 — Login
- Gmail OAuth
- Jika email Gmail **tidak cocok** dengan email di database donatur (dari Google Sheets / form bit.ly/AlumniMauBantu), login sukses tapi **akun belum terhubung** — donatur tidak melihat data. Muncul pesan: "Akun belum terhubung. Hubungi Tim AnC."
- **Fallback admin-link**: Admin AnC bisa **menghubungkan akun Gmail secara manual** di Database Donatur (field `user_id` di tabel `donatur` di-set ke user yang login). Ini menangani kasus email berbeda tanpa harus ubah data di Google Sheets

#### Step D2 — Beranda
- Greeting otomatis tiap hari
- **Info banner** (dismissible, tampil saat pertama login): "Pastikan email Gmail yang kamu pakai login sama dengan email saat mengisi form bit.ly/AlumniMauBantu. Jika berbeda, hubungi Tim AnC agar akun dihubungkan."
- Total donasi, history konsistensi per bulan/batch, batch yang diikuti
- **Akses data beswan = sesuai periode yang di-assign oleh AnC** melalui `donatur_periode`. Donatur hanya melihat beswan dari periode di mana ia aktif

#### Step D3 — Dashboard GBB
- Financial:
  - **Alokasi dana** = otomatis dari `cashflow` (komposisi pengeluaran per kategori, mirip Overview Keuangan internal tapi versi ringkas & publik) — tidak menampilkan nominal saldo absolut
  - **Narasi keuangan** = teks ringkas yang **diisi manual** tim (mis. highlight penyaluran periode ini); opsional
- Beswan: jumlah aktif, ringkasan progress
- Event: jumlah terlaksana, topik yang dibahas

#### Step D4 — Data & Kegiatan Beswan
- Hanya lihat beswan dari periode di mana donatur aktif
- Per beswan: nama, update kegiatan, prestasi, refleksi (summary)
- Filter per periode/batch

#### Step D5 — Daftar Mentor (CTA)
- Info program mentoring, metric jumlah mentor
- Form pendaftaran: nama, bidang, upload CV, LinkedIn
- Submit → masuk database mentor, status **pending review**

#### Step D6 — Laporan GBB
- Daftar laporan & booklet (is_public = true)
- Download PDF langsung

---

## 8. Business Rules

| Rule | Detail |
|------|--------|
| **Periode = 6 bulan fixed** | Hanya 2 opsi: **Jan–Jun** atau **Jul–Des** |
| **Multi-periode aktif** | Bisa >1 periode aktif bersamaan (jika pembinaan molor) |
| **Multi-batch beswan** | 1 beswan bisa di >1 periode. Punya >1 rapor, >1 set absensi, >1 set tugas |
| **Beswan = 6 bulan** | Durasi beswan per periode = 6 bulan |
| **Donatur daftar per periode** | Daftar 1x per periode. Berikutnya ditanya ulang, AnC centang di portal jika lanjut |
| **Donatur kolom periode auto** | Kolom periode baru otomatis muncul saat periode baru dikonfigurasi |
| **Kode donatur** | Auto-generated: `[Inisial][Semester][Tahun]` — collision: extend inisial nama terakhir 2 huruf |
| **Absensi** | Dicatat tim internal, bukan beswan |
| **Penugasan submit** | Tidak bisa edit setelah submit |
| **Refleksi alert** | Wajib bulanan, terkoneksi periode batch |
| **Prestasi alert** | Wajib update per kuartal |
| **IPK update** | Beswan wajib update IP/IPK + transkrip tiap semester di Profile (1 entri per periode, tabel `beswan_ipk`). Alert + email reminder jika belum. Sumber "Avg IPK" internal & IPK di Portal Donatur |
| **Donatur visibility** | Hanya lihat beswan dari periode di mana ia aktif (scope via `donatur_periode`) |
| **Donatur klasifikasi** | AnC wajib meng-assign donatur ke periode (`donatur_periode`). Donatur tanpa periode = belum diklasifikasi → **alert di Monitoring & Database Donatur** agar segera di-assign |
| **Donatur update musiman** | Tiap awal semester (Juli-Agustus, Desember-Januari), AnC harus **update manual** keikutsertaan donatur: siapa yang lanjut, siapa berhenti, siapa baru. Reminder otomatis tampil di dashboard |
| **Donatur email mismatch** | Jika Gmail login ≠ email di DB, akun belum terhubung. Admin AnC bisa **hubungkan manual** (set `donatur.user_id` → user). Banner info di Beranda Donatur |
| **Mentor privacy** | Portal Beswan: tanpa HP & CV |
| **BSI parsing** | Header baris 12, data baris 13+; CR/DB → tipe in/out (dikunci); **dedup FT Number + Nominal** (antar-upload) |
| **Cashflow jenis & klasifikasi** | Cash in = Donasi/Pengembalian/Bagi Hasil (hanya Donasi butuh donatur); kategori auto dari kolom Category file (master `cashflow_kategori`, editable); status `inputted`/`unknown` (wajib 100% inputted sebelum simpan) |
| **Edit cashflow** | Hanya koreksi klasifikasi + catatan; tipe/FT/nominal/tanggal dikunci; hapus baris pakai konfirmasi + undo |
| **Login beswan** | Email + password (generated) |
| **Login donatur** | Gmail OAuth |

---

## 9. Design Artifacts (Status: ✅ Done)

Semua artefak desain sudah final — tidak ada open question tersisa.

| Artefak | Lokasi | Status |
|---------|--------|--------|
| ERD (31 tabel, 10 group) | `docs/erd.dbml` | ✅ |
| Wireframe Internal (12 halaman) | `docs/wireframes-internal.md` | ✅ |
| Wireframe Beswan (5 halaman) | `docs/wireframes-beswan.md` | ✅ |
| Wireframe Donatur (6 halaman) | `docs/wireframes-donatur.md` | ✅ |
| Color Palette & Design System | `docs/colorpalette.md` | ✅ |
| Font | Plus Jakarta Sans (single family) | ✅ |
| Seed data refleksi | `docs/seeds/refleksi_pertanyaan.seed.json` | ✅ |

---

## 10. Tutorial Onboarding (Guided Tour)

> Tutorial interaktif langsung di layar — tooltip + spotlight sambil praktek. Bahasa Indonesia.

### Mekanisme

| Item | Detail |
|------|--------|
| **Library** | React Joyride (tooltip-based guided tour, React-native) |
| **Bahasa** | Full Bahasa Indonesia |
| **Trigger pertama** | Otomatis muncul saat user login pertama kali |
| **Ulangi** | Tombol "Ulangi Tutorial" di sidebar bawah / Settings. Bisa dijalankan ulang kapan saja |
| **Skip** | Tombol "Lewati" di setiap langkah + "Lewati Semua" untuk skip tutorial sepenuhnya |
| **Progress** | Indikator step "Langkah 3 dari 12" di tooltip |
| **Spotlight** | Elemen yang dibahas di-highlight, background di-dim |
| **Navigasi** | Tombol "Sebelumnya", "Selanjutnya", "Lewati" di setiap tooltip |
| **Persistence** | Status tutorial (selesai/belum, step terakhir) disimpan di `localStorage` + DB user preferences |
| **Per-role** | Tutorial berbeda per portal (Internal, Beswan, Donatur) |
| **Kontekstual** | Jika user pindah halaman, tutorial lanjut dari step yang relevan di halaman tersebut |

### Tutorial Flow — Portal Internal

> Internal punya banyak tour karena flow-nya kompleks. Tour 1 auto saat login pertama, tour lainnya trigger kontekstual (pertama kali buka halaman / klik aksi tertentu).

**Tour 1: Pengenalan Portal** (auto saat pertama login)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Sidebar | "Ini sidebar navigasi. Semua menu portal bisa diakses dari sini." |
| 2 | Periode selector (sidebar bawah) | "Pilih periode aktif di sini. Semua data di portal akan terfilter sesuai periode yang dipilih." |
| 3 | Dashboard tab | "Dashboard punya 3 tab: Event untuk operasional, Trend Donatur untuk data pendaftaran, dan Growth untuk analitik peserta event." |
| 4 | Notifikasi bell | "Klik bell untuk melihat notifikasi terbaru. Badge merah = ada yang belum dibaca." |
| 5 | Menu Beswan | "Kelola data beswan di sini — tambah beswan, lihat detail profil, absensi, tugas, dan generate rapor." |
| 6 | Menu Kurikulum | "Susun topik pembinaan per periode. Topik yang kamu buat akan jadi pilihan saat membuat event." |
| 7 | Menu Mentor | "Database mentor — tambah mentor baru, lihat history event dan rata-rata feedback." |
| 8 | Menu Event | "Buat dan kelola event talkshow/growth — mulai dari buat event sampai upload rekaman pasca event." |
| 9 | Menu Penugasan | "Buat tugas untuk beswan, lalu nilai dan beri feedback satu per satu." |
| 10 | Menu Keuangan | "4 sub-menu: Rekonsiliasi (upload & klasifikasi bank statement BSI), Overview Keuangan (ringkasan arus kas), Database Donatur, dan Monitoring Donasi." |
| 11 | Menu Laporan | "Upload laporan dan booklet GBB. Yang ditandai publik akan tampil di Portal Donatur." |
| 12 | Tombol Ulangi Tutorial | "Kalau bingung, kamu bisa ulangi tutorial ini kapan saja dari sini. Selamat bekerja!" |

**Tour 2: Setup Periode Baru** (trigger: pertama kali buka Konfigurasi Periode)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Tabel periode | "Ini daftar semua periode/batch GBB. Kamu bisa punya lebih dari 1 periode aktif bersamaan." |
| 2 | Tombol + Tambah | "Klik untuk membuat periode baru. Sistem akan auto-suggest nama batch berikutnya." |
| 3 | Form nama batch | "Nama batch otomatis di-suggest (misal BBB5). Kamu bisa edit jika perlu." |
| 4 | Field tanggal mulai/selesai | "Pilih tanggal. Periode hanya bisa Jan–Jun (semester 1) atau Jul–Des (semester 2)." |
| 5 | Kolom Status | "Status 'Aktif' artinya periode ini sedang berjalan. Ubah ke 'Selesai' kalau pembinaan sudah selesai." |
| 6 | Selesai | "Setelah periode dibuat, langkah selanjutnya: input data beswan dan susun kurikulum." |

**Tour 3: Input Data Beswan** (trigger: pertama kali buka Database Beswan)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Metric cards | "Ringkasan beswan: total, aktif, alumni, dan rata-rata IPK." |
| 2 | Tombol + Tambah | "Klik untuk input beswan baru. Isi profil → assign ke periode → sistem otomatis generate login." |
| 3 | Form profil | "Isi nama lengkap, NIM, email, HP, dan upload foto + CV." |
| 4 | Pilih periode | "Assign beswan ke periode aktif. Satu beswan bisa ikut lebih dari satu periode (multi-batch)." |
| 5 | Generate login | "Sistem akan generate password otomatis dan mengirim welcome email ke beswan." |
| 6 | Klik row beswan | "Klik nama beswan untuk lihat detail — ada 4 tab: Rapor, Absensi, Tugas, dan Refleksi." |
| 7 | Tab Rapor | "Ringkasan performa beswan: persentase kehadiran, rata-rata nilai tugas, dan status refleksi." |
| 8 | Selesai | "Setelah beswan terdaftar, mereka bisa login ke Portal Beswan dengan kredensial yang di-generate." |

**Tour 4: Susun Kurikulum** (trigger: pertama kali buka Kurikulum & Library)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Tab Kurikulum | "Susun topik pembinaan untuk periode ini. Setiap topik nanti bisa dihubungkan ke event." |
| 2 | Goal periode | "Tulis goal besar kurikulum periode ini — jadi panduan tim dalam menyusun topik." |
| 3 | Tombol + Topik | "Tambah topik baru. Isi urutan, judul, detail, dan upload TOR (Terms of Reference)." |
| 4 | Kolom Status | "Status topik: Planned → Ongoing (saat event dimulai) → Done (saat event selesai + rekaman diupload)." |
| 5 | Kolom Media | "Setelah event selesai, link YouTube dan slide otomatis muncul di sini dari data event." |
| 6 | Tab Library | "Library berisi semua materi — baik dari event maupun upload manual. AI akan otomatis buat summary dan tag." |
| 7 | Selesai | "Kurikulum siap! Langkah selanjutnya: buat event berdasarkan topik yang sudah disusun." |

**Tour 5: Buat & Kelola Event** (trigger: pertama kali buka Event Talkshow)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Metric cards | "Ringkasan event: total, selesai, upcoming, dan yang belum punya rekaman." |
| 2 | Alert rekaman | "Perhatikan alert ini — event yang sudah selesai tapi belum diupload rekaman/slide akan muncul di sini." |
| 3 | Tombol + Buat Event | "Klik untuk buat event baru." |
| 4 | Pilih periode | "Pilih periode. Ini menentukan event untuk batch mana." |
| 5 | Pilih topik kurikulum | "Pilih topik dari kurikulum, atau kosongkan jika ini event non-kurikulum. Kode event otomatis di-generate." |
| 6 | Detail event | "Isi nama event, tipe (Talkshow/Growth/Other), format (Online/Offline/Hybrid), tanggal, jam, lokasi, dan deskripsi." |
| 7 | Assign mentor | "Tambah satu atau lebih mentor. Pilih peran masing-masing: Speaker, Moderator, atau Fasilitator." |
| 8 | Selesai | "Event siap! Status awal 'Upcoming'. Setelah event berlangsung, lanjut ke tutorial berikutnya." |

**Tour 6: Pasca Event — Absensi & Upload** (trigger: admin klik event berstatus 'Upcoming' atau 'Done' pertama kali)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Detail event | "Ini halaman detail event. Dari sini kamu bisa kelola absensi dan upload materi." |
| 2 | Tabel absensi | "Catat kehadiran beswan di sini. Centang 'Hadir' untuk setiap beswan yang datang." |
| 3 | Tombol simpan absensi | "Klik simpan setelah semua absensi dicatat. Data ini akan masuk ke rapor beswan." |
| 4 | Field YouTube URL | "Setelah event selesai, paste link rekaman YouTube di sini." |
| 5 | Field Slide URL | "Upload atau paste link slide materi. Ini akan otomatis muncul di Kurikulum dan Library." |
| 6 | Tombol update status | "Ubah status ke 'Done'. Jika rekaman atau slide belum diisi, sistem akan menampilkan alert pengingat." |
| 7 | Auto-library | "Materi yang kamu upload otomatis masuk ke Library dengan AI summary dan auto-tag. Kamu bisa edit tag manual." |
| 8 | Selesai | "Pasca event selesai! Langkah selanjutnya: buat penugasan berdasarkan event ini." |

**Tour 7: Buat Penugasan** (trigger: pertama kali buka halaman Penugasan)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Metric cards | "Ringkasan penugasan: total tugas, jumlah submitted, dan yang belum dikumpulkan." |
| 2 | Master list (daftar tugas) | "Ini daftar semua penugasan. Klik salah satu untuk lihat detail hasil beswan di panel bawah." |
| 3 | Tombol + Buat Penugasan | "Klik untuk buat tugas baru." |
| 4 | Pilih periode/batch | "Pilih batch. Tugas ini akan muncul di Portal Beswan untuk semua beswan di batch tersebut." |
| 5 | Pilih event sumber | "Hubungkan ke event jika tugas ini berkaitan dengan event tertentu. Bisa juga dikosongkan untuk tugas mandiri." |
| 6 | Form tugas | "Isi judul, deskripsi/soal, dan deadline. Kode tugas otomatis di-generate (misal TGS-BBB4-01)." |
| 7 | Selesai | "Tugas terpublish! Beswan akan menerima notifikasi dan bisa submit jawaban di Portal Beswan." |

**Tour 8: Menilai Penugasan & Feedback** (trigger: pertama kali klik penugasan yang ada submisi)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Detail panel | "Ini panel detail — daftar semua beswan untuk tugas ini beserta status mereka." |
| 2 | Kolom Status | "Status per beswan: 'Belum kumpul' → 'Submitted' (sudah upload) → 'Graded' (sudah dinilai). 'Late' jika lewat deadline." |
| 3 | Kolom File | "Klik ikon file untuk download/preview jawaban beswan." |
| 4 | Kolom Nilai | "Masukkan nilai angka untuk setiap beswan yang sudah submit." |
| 5 | Kolom Feedback | "Tulis feedback teks — ini akan tampil di Portal Beswan sebagai catatan dari tim program." |
| 6 | Tombol simpan nilai | "Klik simpan. Status berubah ke 'Graded' dan beswan menerima notifikasi email bahwa tugasnya sudah dinilai." |
| 7 | Beswan belum kumpul | "Beswan yang belum submit ditandai 'Belum kumpul'. Mereka akan dapat reminder otomatis mendekati deadline." |
| 8 | Selesai | "Penilaian selesai! Nilai dan feedback akan masuk ke rapor beswan secara otomatis." |

**Tour 9: Upload Bank Statement & Klasifikasi** (trigger: pertama kali buka Keuangan → Rekonsiliasi)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Tombol Upload | "Upload file .xlsx mutasi bank BSI. Bisa multi-sheet (misal: sheet September, Agustus, dll)." |
| 2 | Hasil parsing | "Sistem otomatis membaca dari baris ke-13, mendeteksi kolom CR (masuk) dan DB (keluar). Transaksi yang sudah pernah masuk (FT Number + Nominal sama) ditandai DUPLIKAT dan tidak disimpan ulang." |
| 3 | Tabel transaksi | "Ini daftar transaksi yang berhasil di-parse. Setiap baris perlu diklasifikasikan." |
| 4 | Cash In → Donatur | "Uang masuk dikelompokkan jadi Donasi, Pengembalian, atau Bagi Hasil — kategori sudah otomatis dari file, bisa kamu koreksi. Khusus Donasi: sistem coba cocokkan nama donatur dari deskripsi (termasuk kalau namanya kepotong). Kalau cocok lebih dari satu / tidak ketemu, ketik/pilih nama donatur atau pilih 'Anonymous'. Nominal otomatis masuk ke monitoring donasi." |
| 5 | Cash Out → Kategori | "Untuk uang keluar: kategori ditebak otomatis (Beasiswa, Program, Biaya Operasional, dll) dan tetap bisa kamu koreksi. Kelola daftar kategori lewat tombol Kategori." |
| 6 | Kolom Status | "Status klasifikasi cuma dua: 'Inputted' (sudah diklasifikasi) dan 'Unknown' (belum). Semua baris wajib jadi Inputted sebelum bisa Simpan — baris yang bukan transaksi relevan bisa dihapus." |
| 7 | Edit & koreksi | "Salah klasifikasi? Klik baris untuk perbaiki kapan saja — nama donatur & kategori bisa dikoreksi. FT Number, nominal, dan tanggal dikunci karena ikut data bank. Bisa juga hapus baris (ada konfirmasi & bisa di-undo). Setiap perubahan tercatat." |
| 8 | Selesai | "Setelah semua baris diklasifikasi, data akan otomatis muncul di Dashboard dan Monitoring Donatur." |

**Tour 10: Monitoring Donatur** (trigger: pertama kali buka halaman Monitoring)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Metric cards | "Ringkasan cepat: total donatur, yang aktif periode ini, dan yang belum dikonfirmasi." |
| 2 | Info banner WA | "Portal ini terintegrasi dengan WhatsApp. Kamu bisa kirim reminder langsung dari tabel." |
| 3 | Filter bar | "Filter berdasarkan periode, status keaktifan, atau tag untuk mempersempit daftar." |
| 4 | Kolom Nama & Tag | "Setiap donatur menampilkan: nama, kode (misal DLW12025), skema donasi, dan badge tag jika ada." |
| 5 | Tombol Kirim | "Klik 'Kirim' lalu pilih template pesan (reminder awal bulan, follow-up, dll). WhatsApp terbuka dengan pesan sudah terisi otomatis — nama donatur, bulan, dll. Template bisa kamu kelola di Settings." |
| 6 | Checkbox tandai | "Centang donatur yang sudah kamu follow-up. Tanda ini permanen & tampil di semua user sampai di-uncheck." |
| 7 | Kolom bulan | "Kolom per bulan menampilkan nominal donasi dari data cashflow. '—' artinya belum ada data." |
| 8 | Total & Catatan | "Total donasi per periode di kolom terakhir. Klik ikon catatan untuk tambah note internal per donatur." |
| 9 | Dropdown Tag | "Assign tag ke donatur: Donatur Setia, Perlu Follow-Up, Sulit Dihubungi, dll. Satu donatur bisa punya banyak tag." |
| 10 | Selesai | "Monitoring donatur siap! Gunakan tombol Kirim untuk reminder & follow-up donatur secara rutin." |

**Tour 11: Generate Rapor** (trigger: pertama kali klik "Generate Rapor" atau buka tab Rapor di detail beswan)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Pilih beswan & periode | "Pilih beswan dan periode untuk rapor yang mau di-generate." |
| 2 | Section Kehadiran | "Ringkasan kehadiran: daftar event dan status hadir/tidak, plus persentase total." |
| 3 | Section Penugasan | "Semua tugas beswan di periode ini: judul, nilai, status, dan rata-rata." |
| 4 | Section Refleksi | "Status refleksi bulanan: sudah/belum submit per bulan." |
| 5 | Section Prestasi | "Prestasi yang dicatat beswan selama periode ini." |
| 6 | Tombol Download PDF | "Klik untuk download rapor sebagai PDF. Rapor bisa di-generate ulang kapan saja — data selalu real-time." |
| 7 | Selesai | "Rapor bisa diakses juga oleh beswan di portal mereka. Data otomatis update." |

### Daftar Tour & Trigger Summary — Portal Internal

| # | Nama Tour | Trigger | Steps |
|---|-----------|---------|-------|
| 1 | Pengenalan Portal | Auto saat login pertama | 12 |
| 2 | Setup Periode Baru | Pertama buka Konfigurasi Periode | 6 |
| 3 | Input Data Beswan | Pertama buka Database Beswan | 8 |
| 4 | Susun Kurikulum | Pertama buka Kurikulum & Library | 7 |
| 5 | Buat & Kelola Event | Pertama buka Event Talkshow | 8 |
| 6 | Pasca Event | Pertama klik detail event | 8 |
| 7 | Buat Penugasan | Pertama buka Penugasan | 7 |
| 8 | Menilai & Feedback | Pertama klik tugas yang ada submisi | 8 |
| 9 | Upload BSI & Klasifikasi | Pertama buka Keuangan → Rekonsiliasi | 8 |
| 10 | Monitoring Donatur | Pertama buka Monitoring | 10 |
| 11 | Generate Rapor | Pertama klik Generate Rapor / tab Rapor | 7 |

> Total: **11 tour, 88 langkah** untuk Portal Internal. Setiap tour bisa diulangi dan di-skip secara independen.

### Tutorial Flow — Portal Beswan

**Tour: Pengenalan Portal Beswan** (auto saat pertama login)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Greeting | "Selamat datang di Portal Beswan GBB! Di sini kamu bisa akses semua yang berkaitan dengan program beasiswa." |
| 2 | Notifikasi | "Cek notifikasi di sini — tugas baru, nilai, dan pengingat refleksi." |
| 3 | My Events | "Daftar event yang sudah dan akan kamu ikuti. Setelah event selesai, kamu bisa akses rekaman dan slide." |
| 4 | My Tasks | "Tugas dari tim GBB muncul di sini. Klik untuk lihat detail dan upload jawaban." |
| 5 | Menu Library | "Cari materi pembinaan di sini. Kamu juga bisa usulkan topik materi baru." |
| 6 | Menu Mentor | "Lihat daftar mentor. Butuh bimbingan? Request sesi 1-on-1 di sini." |
| 7 | Menu Refleksi | "Setiap bulan, kamu wajib mengisi refleksi dan upload dokumentasi kegiatan. Jangan sampai terlewat!" |
| 8 | Menu Profile | "Lengkapi profilmu di sini — HP, CV, dan **IPK semester berjalan** (wajib diperbarui tiap semester, lengkap dengan transkrip)." |
| 9 | Prestasi | "Catat prestasi kamu di sini — upload sertifikat atau foto. Wajib update tiap kuartal." |
| 10 | Selesai | "Itu dia! Kalau bingung, kamu bisa ulangi tutorial ini kapan saja. Semangat!" |

### Tutorial Flow — Portal Donatur

**Tour: Pengenalan Portal Donatur** (auto saat pertama login via Gmail)

| Step | Target Element | Tooltip |
|------|---------------|---------|
| 1 | Greeting | "Terima kasih sudah bergabung sebagai donatur GBB! Portal ini tempat kamu memantau dampak kontribusimu." |
| 1b | Info banner | "Pastikan email Gmail kamu sama dengan yang dipakai saat mengisi form bit.ly/AlumniMauBantu. Jika berbeda, hubungi Tim AnC agar akunmu dihubungkan." |
| 2 | Metric cards | "Ringkasan donasi kamu: total yang sudah diberikan, konsistensi per bulan, dan batch yang kamu ikuti." |
| 3 | History konsistensi | "Tabel ini menunjukkan konsistensi donasimu per bulan di setiap batch. Centang hijau = sudah tercatat." |
| 4 | Menu Dashboard | "Lihat ringkasan keuangan GBB, progress beswan, dan event yang sudah berjalan." |
| 5 | Menu Data Beswan | "Pantau perkembangan beswan yang kamu dukung — kegiatan, prestasi, dan refleksi mereka." |
| 6 | Menu Daftar Mentor | "Tertarik jadi mentor? Daftar di sini! Isi form dan tim GBB akan menghubungimu." |
| 7 | Menu Laporan | "Download laporan dan booklet GBB di sini." |
| 8 | Selesai | "Tutorial selesai! Kamu bisa ulangi kapan saja. Terima kasih sudah menjadi bagian GBB!" |

### Kustomisasi Tooltip

| Elemen | Teks |
|--------|------|
| Tombol Next | "Selanjutnya →" |
| Tombol Prev | "← Sebelumnya" |
| Tombol Skip | "Lewati" |
| Tombol Skip All | "Lewati Semua" |
| Tombol Finish | "Selesai!" |
| Counter | "Langkah {n} dari {total}" |
| Tombol Ulangi | "🔄 Ulangi Tutorial" (di sidebar) |

---

## 11. Out of Scope (MVP)

- RAG untuk Library
- Payment gateway
- Mobile app (responsive web + PWA dulu)
- Multi-bahasa
- Integrasi WhatsApp langsung / WA Business API (saat ini hanya `wa.me` link — lihat §13 untuk catatan future)

---

## 12. Non-Functional Requirements (NFR)

Acuan kualitas lintas-fitur. Setiap fase harus memenuhi NFR yang relevan sebelum dianggap selesai (lihat §14).

| Kategori | Requirement | Target / Acuan |
|----------|-------------|----------------|
| **Performance** | Initial load halaman utama (LCP) | < 2.5 dtk di koneksi 4G |
| | Response API (p95) | < 500 ms untuk operasi read |
| | List besar (donatur, beswan) | Server-side pagination + filter, **tidak** fetch semua row |
| **Scalability** | Skema mendukung multi-periode & multi-batch tanpa perubahan struktur | Lihat §13 |
| **Security** | Auth | Better Auth, password di-hash (bcrypt/argon2), session aman (httpOnly cookie) |
| | Otorisasi | Role-based access control (RBAC) di **server**, bukan hanya hide di UI |
| | Isolasi portal | Internal/Beswan/Donatur terisolasi — Donatur hanya lihat beswan periode di mana ia aktif |
| | File upload | Validasi tipe & ukuran (lihat limit di `implementation_plan.md`); scan ekstensi; URL MinIO tidak public-guessable |
| | Input | Validasi server-side (Zod) untuk semua form; proteksi SQL injection (Drizzle parameterized) & XSS |
| | Secret / API key | API key AI & Google Service Account disimpan **terenkripsi at-rest**, hanya didekripsi di server, **tidak pernah** dikirim ke client / masuk log |
| **Reliability** | Cron job (reminder, sync) | Idempotent + log hasil; gagal tidak boleh kirim email dobel |
| | PWA offline | Upload penugasan & refleksi masuk queue, auto-retry saat online (lihat `implementation_plan.md`) |
| | Backup | Backup DB harian + sebelum tiap migration; file MinIO di-backup |
| **Accessibility** | Kontras & keyboard | Ikuti token kontras `colorpalette.md`; form navigable via keyboard; label pada input |
| **Browser** | Support | 2 versi terakhir Chrome, Safari, Edge, Firefox + mobile Safari/Chrome |
| **Observability** | Logging & error | Structured log di server; error tracking (mis. Sentry) untuk produksi |
| **Privacy** | Data sensitif | HP/CV beswan & data donatur tidak bocor ke role yang tidak berhak; mentor tanpa HP/CV di Portal Beswan |

---

## 13. Scalability & Data Assumptions

Skala MVP kecil, tapi skema & arsitektur dirancang agar **tumbuh tanpa rewrite**.

### Estimasi volume (1–3 tahun)

| Entitas | Estimasi | Pertumbuhan |
|---------|----------|-------------|
| Periode/batch | 2 per tahun | Linear, kecil |
| Beswan | ~20–40 per batch | ~80/tahun |
| Donatur | ~100–300 | Lambat |
| Event | ~10–20 per batch | — |
| Refleksi | beswan × 6 bulan × jumlah jawaban | **Tumbuh tercepat** — perhatikan indexing |
| Cashflow | ratusan baris/bulan | Linear |
| File (CV, sertifikat, materi, laporan) | MB–GB | Pantau kuota MinIO |

### Prinsip scalability

| Aspek | Keputusan |
|-------|-----------|
| **Multi-periode/batch** | Semua entitas per-periode di-scope via `periode_id`/`beswan_periode` — tambah batch = insert data, bukan ubah skema |
| **Pagination** | Wajib server-side untuk semua list (default 25/halaman). Jangan `SELECT *` tanpa limit |
| **Indexing** | Index pada FK & kolom filter umum: `refleksi(beswan_id, periode_id, bulan)`, `cashflow(periode_id, tanggal)`, `event_beswan(event_id)`, `donatur_periode(periode_id)` |
| **Refleksi (normalized)** | Jawaban di tabel `refleksi_jawaban` (link via `pertanyaan_id`) — pertanyaan editable tanpa migrasi & data historis aman. Pertanyaan punya `kode` stabil untuk analitik lintas waktu |
| **File storage** | MinIO bucket terpisah per tipe; URL ber-expiry/presigned, bukan public listing |
| **Caching / PWA** | Data read-only (event, library, dashboard) di-cache di klien; mutasi via queue saat offline |
| **Google Sheets sync** | Mirror read-only tiap 15 menit (bukan source of truth untuk operasional portal) |
| **Template (refleksi & pesan WA)** | Disimpan sebagai data (`refleksi_pertanyaan`, `pesan_template`), bukan hardcode — admin ubah tanpa deploy |
| **Provider AI** | Lewat adapter (`anthropic` + `openai_compatible`) & tabel `ai_config` — ganti/tambah provider via Settings (API key), tanpa ubah kode. Konsumen pertama: auto-summary Library |
| **IPK per semester** | `beswan_ipk` di-scope per `periode_id` (1 baris/semester) — riwayat IPK tumbuh sebagai data, mendukung tren & multi-batch tanpa ubah skema |

### Future (kalau memungkinkan, di luar MVP)
- **WhatsApp langsung**: AnC login WA sekali di portal → kirim pesan ter-template langsung (WA Web session / WA Business API), menggantikan `wa.me` link manual. Skema `pesan_template` sudah siap menampung ini.
- RAG Library, payment gateway, mobile app native.

---

## 14. Quality Harness — Testing & Acceptance

Agar mutu terjamin dan konsisten lintas engineer.

### Definition of Done (per fitur/fase)
Sebuah fitur dianggap **selesai** hanya jika:
1. ✅ Memenuhi acceptance criteria fitur (lihat template di bawah)
2. ✅ NFR relevan terpenuhi (§12) — terutama RBAC server-side & validasi input
3. ✅ `npm run build` lulus tanpa error (client + server), TypeScript strict
4. ✅ Lint & format bersih
5. ✅ Migration + seed jalan dari nol tanpa error
6. ✅ Test relevan ditulis & hijau (lihat strategi di bawah)
7. ✅ Walkthrough manual + screenshot/recording (lihat Verification Plan di `implementation_plan.md`)
8. ✅ Reviewed via Pull Request (min. 1 reviewer)

### Template acceptance criteria (format Gherkin)
Tulis per user story, contoh untuk Refleksi Bulanan:
```
GIVEN beswan login di periode aktif
  AND belum mengisi refleksi bulan berjalan
WHEN beswan membuka menu Refleksi
THEN tampil form 17 pertanyaan dengan field wajib ditandai
  AND pertanyaan "nama lomba" hanya muncul jika "ikut lomba" = Ya
  AND submit ditolak jika field wajib / upload dokumentasi kosong
  AND setelah submit, status bulan tsb = "submitted" dan tidak bisa diedit selain via dropdown bulan
```

### Strategi testing

| Level | Cakupan | Tools (saran) |
|-------|---------|---------------|
| **Unit** | Logika murni: generate kode donatur, parsing BSI (baris 13, CR/DB, dedup), evaluasi `kondisi` form, format display periode | Vitest |
| **Integration** | API + DB: RBAC per role, pagination, mutasi refleksi/penugasan, dedup cashflow | Vitest + test DB |
| **E2E (kritikal)** | Login per role & routing, submit refleksi, upload penugasan, upload BSI → klasifikasi, kirim WA template | Playwright |
| **Migration/seed** | Migrate dari nol + load seed → skema valid | Drizzle test |
| **Manual** | Walkthrough per portal per fase, responsive, role-access | Checklist `implementation_plan.md` |

### Prioritas test (high-risk dulu)
1. **RBAC & isolasi portal** — kebocoran data antar-role = risiko tertinggi
2. **Parsing & dedup BSI** — salah hitung keuangan
3. **Refleksi conditional + wajib** — integritas data laporan
4. **Auth & generate kredensial**
5. **Cron idempotency** — hindari email dobel

### Quality gates di CI (saran)
`build` → `typecheck` → `lint` → `unit + integration` → (`e2e` di staging). Gagal di gate mana pun = blok merge.

### Environments
| Env | Tujuan |
|-----|--------|
| **dev** | Lokal engineer, DB & MinIO lokal/docker |
| **staging** | Mirror produksi di VPS, untuk UAT & e2e sebelum rilis |
| **prod** | `portal.baikberdampak.org` |
