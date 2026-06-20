# ASCII Wireframes — Portal Internal GBB
> Desain: Notion-style (sidebar kiri + konten kanan)

## Global Layout

```
┌──────────────────────────────────────────────────────────────────────┐
│ 🟢 GBB Portal Internal                              🔔  👤 Admin ▼ │
├────────────┬─────────────────────────────────────────────────────────┤
│            │                                                         │
│  SIDEBAR   │                    CONTENT AREA                         │
│            │                                                         │
│  📊 Dashboard   │                                                    │
│  🗓 Periode     │                                                    │
│  👥 Beswan      │                                                    │
│  📚 Kurikulum   │                                                    │
│  🎓 Mentor      │                                                    │
│  🎤 Event       │                                                    │
│  📝 Penugasan   │                                                    │
│  💰 Keuangan ▼  │                                                    │
│    Rekonsiliasi  │                                                    │
│    Overview      │                                                    │
│    Db Donatur    │                                                    │
│    Monitoring    │                                                    │
│  📄 Laporan     │                                                    │
│            │                                                         │
│  ─────────── │                                                       │
│  ⚙ Settings │                                                       │
│  Filter Periode:│                                                    │
│  BBB4 ▼         │                                                    │
└────────────┴─────────────────────────────────────────────────────────┘
```

---

## 1. Dashboard

> 4 tab: **Event** (data dari portal), **Beswan** (analitik pembinaan beswan — untuk PCM), **Trend Donatur** (data dari GSheets pendaftaran donatur), **Growth** (data dari GForm pendaftaran event)

### Tab: Event

```
┌─────────────────────────────────────────────────────────────────┐
│  Dashboard                                       [↻ Refresh]    │
│                                                                 │
│  [Tab: Event] [Tab: Beswan] [Tab: Trend Donatur] [Tab: Growth]  │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ 📊 12    │ │ ✅ 8     │ │ 👥 15    │ │ 💰 4.2jt │           │
│  │ Total    │ │ Selesai  │ │ Beswan   │ │ Donasi   │           │
│  │ Event    │ │ Event    │ │ Aktif    │ │ Bulan Ini│           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│                                                                 │
│  ┌─────────────────────────┐ ┌─────────────────────────┐       │
│  │  Event per Bulan        │ │  Kehadiran Beswan        │       │
│  │  ┌───┐                  │ │                          │       │
│  │  │▓▓▓│ ┌───┐            │ │  ████████████░░ 85%      │       │
│  │  │▓▓▓│ │▓▓▓│ ┌───┐      │ │  Rata-rata kehadiran     │       │
│  │  │▓▓▓│ │▓▓▓│ │▓▓▓│      │ │                          │       │
│  │  Agt   Sep   Okt        │ │  ┌─── Pie Chart ───┐     │       │
│  └─────────────────────────┘ │  │  Hadir  85%      │     │       │
│                              │  │  Absen  15%      │     │       │
│  ┌─────────────────────────┐ └─────────────────────────┘       │
│  │  Penugasan Overview     │                                    │
│  │  Submitted: 45          │                                    │
│  │  Graded: 38             │                                    │
│  │  Pending: 7             │                                    │
│  └─────────────────────────┘                                    │
└─────────────────────────────────────────────────────────────────┘
```

### Tab: Beswan

> Analitik pembinaan beswan untuk tim PCM — agregat (kumulatif) + bisa drill per beswan per periode.
> Sumber: `event_beswan` (kehadiran), `hasil_penugasan` (nilai), `refleksi` (completion), `beswan_ipk` (IPK), `prestasi`.
>
> ⚠ Tab ini **read-only (analitik)**. Untuk **mengelola** beswan — tambah/edit, upload CV, generate kredensial login, assign periode, kelola Rapor/Absensi/Tugas/Refleksi — gunakan menu **👥 Database Beswan** (§2).

```
┌─────────────────────────────────────────────────────────────────┐
│  Dashboard › Beswan                              [↻ Refresh]    │
│  Filter: [BBB4 (Jan–Jun 2026) ▼]   Lihat: (•) Agregat ( ) Per Beswan │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ 👥 15    │ │ 📊 83%   │ │ 🎓 3.45  │ │ 📝 78%   │           │
│  │ Beswan   │ │ Avg      │ │ Avg IPK  │ │ Refleksi │           │
│  │ Aktif    │ │ Kehadiran│ │          │ │ On-time  │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│                                                                 │
│  ┌─────────────────────────┐ ┌─────────────────────────────────┐│
│  │ Tren Kehadiran /bln     │ │ Refleksi Completion /bln        ││
│  │ (line, kumulatif)       │ │ (stacked bar: sudah/belum)      ││
│  │  ╱▔▔▔╲___╱              │ │  ▓▓░ ▓▓▓ ▓░░ ▓▓░               ││
│  │  Jan Feb Mar Apr        │ │  Jan Feb Mar Apr                ││
│  └─────────────────────────┘ └─────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────┐ ┌─────────────────────────────────┐│
│  │ Distribusi IPK          │ │ Rata-rata Nilai Tugas /batch    ││
│  │ (bar: <3.0,3.0-3.5,>3.5)│ │ (bar per batch)                 ││
│  └─────────────────────────┘ └─────────────────────────────────┘│
│                                                                 │
│  ═══ Tabel Progress per Beswan (klik → detail) ═══════════════  │
│  ┌────────────────┬──────┬──────┬────────┬──────┬─────────────┐ │
│  │ Nama           │ Hadir│ Tugas│Refleksi│ IPK  │ Prestasi    │ │
│  ├────────────────┼──────┼──────┼────────┼──────┼─────────────┤ │
│  │ Ahmad Fauzi    │ 83%  │ 85   │ 4/5    │ 3.45 │ 2           │ │
│  │ Siti Nurhaliza │ 92%  │ 88   │ 5/5    │ 3.78 │ 3           │ │
│  └────────────────┴──────┴──────┴────────┴──────┴─────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

> **Mode "Per Beswan"**: pilih 1 beswan → tampil progress chart kumulatif lintas periode (kehadiran, nilai tugas, IPK per semester, refleksi completion). Sama dengan grafik di tab Rapor (Detail Beswan §2a), dipakai PCM untuk mengolah/mengevaluasi tiap beswan.

---

### Tab: Trend Donatur

> Sumber data: Google Sheets — Database/Pendaftaran Donatur

```
┌─────────────────────────────────────────────────────────────────┐
│  Dasbor Trend Donatur                            [↻ Refresh]    │
│                                                                 │
│  ┌──────────────┐ ┌──────────────┐ ┌───────────────────────────┐│
│  │ 💰 Rp 11.5jt │ │ 👥 98        │ │ Tren Pendaftaran          ││
│  │ Total Est.   │ │ Total Calon  │ │ (line chart per bulan     ││
│  │ Komitmen     │ │ Donatur      │ │  Mar 24 → Jun 26)         ││
│  └──────────────┘ └──────────────┘ └───────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────┐ ┌─────────────────────────────────┐│
│  │ Jenis Beasiswa          │ │ Kriteria Penerima               ││
│  │ (pie chart)             │ │ (horizontal bar)                ││
│  │ - Belum/Lainnya  74%    │ │ - Belum/Lainnya                 ││
│  │ - Beasiswa Uang  15%   │ │ - Kriteria Butuh Dukung. Ekon.  ││
│  │ - Beasiswa Biaya  7%   │ │ - Butuh duk. ekonomi + kemauan  ││
│  │ - dll                   │ │ - dll                           ││
│  └─────────────────────────┘ └─────────────────────────────────┘│
│                                                                 │
│  ┌───────────────┐ ┌───────────────┐ ┌─────────────────────────┐│
│  │ Pekerjaan     │ │ Saluran Info  │ │ Alumni                  ││
│  │ (bar chart)   │ │ (bar chart)   │ │ (bar per angkatan)      ││
│  │ Karyawan  73  │ │ BelumLainnya  │ │ 2020: 51                ││
│  │ PNS  4       │ │ Teman  14     │ │ 2019: 26                ││
│  │ dll           │ │ IG  5        │ │ dll                      ││
│  └───────────────┘ └───────────────┘ └─────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────┐ ┌─────────────────────────────────┐│
│  │ Skema Donasi            │ │ Faktor Ragu                     ││
│  │ (pie chart)             │ │ (horizontal bar)                ││
│  │ - Sebulan sekali  62%   │ │ - Belum/Lainnya                 ││
│  │ - 1Sem sekali  17%     │ │ - Belum punya penghasilan       ││
│  │ - Sebulan sekali dona   │ │ - Transparansi                  ││
│  │ - dll                   │ │ - dll                           ││
│  └─────────────────────────┘ └─────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Tab: Growth

> Sumber data: Google Form — Pendaftar Event GBB (peserta eksternal)

```
┌─────────────────────────────────────────────────────────────────┐
│  Dashboard Growth                                [↻ Refresh]    │
│  Data peserta eksternal event GROWTH — analitik minat            │
│  kontribusi & demografi                                         │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ 👥 70    │ │ 🤝 28    │ │ 🎓 0     │ │ ❤ 0      │           │
│  │ Total    │ │ Berminat │ │ Calon    │ │ Calon    │           │
│  │ Peserta  │ │ Kontrib. │ │ Mentor   │ │ Donatur  │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│                                                                 │
│  ┌─────────────────────────┐ ┌─────────────────────────────────┐│
│  │ Distribusi Profesi      │ │ Minat Kontribusi ke GBB         ││
│  │ (pie chart)             │ │ (horizontal bar per tahun)      ││
│  │ - Lainnya  81%          │ │ 2020: 9 | 2018: 6 | 2019: 4    ││
│  │ - Mahasiswa  4%         │ │ 2017: 3 | dll                  ││
│  │ - Karyawan Swasta       │ │                                 ││
│  └─────────────────────────┘ └─────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────┐ ┌─────────────────────────────────┐│
│  │ Tren Tema Diminati      │ │ Bidang Keahlian Ditawarkan      ││
│  │ (horizontal bar)        │ │ (horizontal bar)                ││
│  │ - Karir & Dunia Kerja   │ │ - Pendidikan & Riset            ││
│  │ - CV  5                 │ │ - Karir & Pengembangan Prof.    ││
│  │ - Personal Branding  4  │ │                                 ││
│  │ - Interview  3          │ │                                 ││
│  │ - dll                   │ │                                 ││
│  └─────────────────────────┘ └─────────────────────────────────┘│
│                                                                 │
│  ┌───────────────┐ ┌───────────────┐ ┌─────────────────────────┐│
│  │ Univ. Asal    │ │ Saluran Info  │ │ Pengenalan GBB          ││
│  │ (pie chart)   │ │ (pie chart)   │ │ (donut chart)           ││
│  │ - Ya  64%     │ │ - WhatsApp 45%│ │ - per fakultas/prodi    ││
│  │ - Tidak  27%  │ │ - IG  23%     │ │ Biologi, T.Kimia,      ││
│  │ - Lainnya  9% │ │ - Rekan  32%  │ │ Statistika, T.Industri ││
│  └───────────────┘ └───────────────┘ └─────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 1b. Konfigurasi Periode

> Setup batch/periode (Step 1.1). Akses: Super Admin ✏️, PCM 👁. Bisa **>1 periode aktif**.

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigurasi Periode                              [+ Tambah]    │
│                                                                 │
│  ┌────┬──────────────────────┬──────────┬──────────┬────────┬───┐│
│  │ ID │ Nama (display)       │ Mulai    │ Selesai  │ Status │Aks││
│  ├────┼──────────────────────┼──────────┼──────────┼────────┼───┤│
│  │ 4  │ BBB #4 (Jan–Jun 2026)│ 01/01/26 │ 30/06/26 │ Aktif  │✏🗑││
│  │ 3  │ BBB #3 (Jul–Des 2025)│ 01/07/25 │ 31/12/25 │ Aktif  │✏🗑││
│  │ 2  │ BBB #2 (Jan–Jun 2025)│ 01/01/25 │ 30/06/25 │ Selesai│✏🗑││
│  └────┴──────────────────────┴──────────┴──────────┴────────┴───┘│
│                                                                 │
│  ── Tambah Periode (auto-suggest dari periode terakhir) ─────── │
│  Nama batch: [BBB5]   Semester: (•) Jan–Jun  ( ) Jul–Des       │
│  Mulai: [01/07/2026]  Selesai: [31/12/2026]                    │
│  Status: [Aktif ▼]                              [Batal][Simpan] │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Database Beswan

> **Avg IPK** = rata-rata IPK kumulatif terbaru beswan aktif, dari `beswan_ipk` (beswan update tiap semester di Portal Beswan → Profile).

```
┌─────────────────────────────────────────────────────────────────┐
│  Database Beswan                                  [+ Tambah]    │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ 👥 15    │ │ ✅ 12    │ │ 🎓 3     │ │ 📊 3.45  │           │
│  │ Total    │ │ Aktif    │ │ Alumni   │ │ Avg IPK  │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│                                                                 │
│  🔍 Cari nama/NIM...  [Semua Batch ▼] [Semua Status ▼] 🔄      │
│                                                                 │
│  ┌───┬────┬────────────────┬──────────┬───────┬────────┬──────┐ │
│  │ # │Foto│ Nama           │ NIM      │ Batch │ Status │ Aksi │ │
│  ├───┼────┼────────────────┼──────────┼───────┼────────┼──────┤ │
│  │ 1 │ 🟢│ Ahmad Fauzi     │ 21050111 │ BBB3  │ Aktif  │ 👁 ✏│ │
│  │ 2 │ 🟢│ Siti Nurhaliza  │ 21050112 │ BBB4  │ Aktif  │ 👁 ✏│ │
│  │ 3 │ ⚫│ Budi Santoso    │ 20050109 │ BBB2  │ Alumni │ 👁 ✏│ │
│  └───┴────┴────────────────┴──────────┴───────┴────────┴──────┘ │
│                                              [< 1 2 3 ... >]   │
└─────────────────────────────────────────────────────────────────┘
```

### 2a. Detail Beswan (Slide-over / Modal)

```
┌─────────────────────────────────────────────────────────────────┐
│  ← Kembali                Ahmad Fauzi                    [✏ Edit]│
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 📷 Foto   Nama: Ahmad Fauzi                               │ │
│  │           NIM: 21050111                                    │ │
│  │           Email: ahmad@mail.com | HP: 081234567890         │ │
│  │           Batch: BBB3, BBB4    | Status: Aktif             │ │
│  │           CV: 📎 Download                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  [Tab: Rapor] [Tab: Absensi] [Tab: Tugas] [Tab: Refleksi]      │
│                                                                 │
│  ── Rapor ──────────────────────────────────────────────────── │
│  Periode: [BBB4 ▼]   (multi-batch → pilih periode)              │
│  Kehadiran: 10/12 event (83%)  ████████░░                      │
│  Tugas: 5/6 submitted, avg nilai 85                            │
│  Refleksi: 4/5 bulan submitted                                 │
│                                                                 │
│  ── Progress Chart (per beswan × periode terpilih) ─────────── │
│  ┌─────────────────────────┐ ┌─────────────────────────────────┐│
│  │ Kehadiran & Nilai /bln  │ │ IPK per Semester (kumulatif)    ││
│  │ (line ganda)            │ │ (line lintas periode)           ││
│  │  ╱▔╲╱▔                  │ │  3.2→3.4→3.45                   ││
│  └─────────────────────────┘ └─────────────────────────────────┘│
│                                                                 │
│  ── Absensi per Event ──────────────────────────────────────── │
│  │ Event                    │ Tanggal    │ Status   │          │
│  │ Growth: CV Writing       │ 15 Sep 25  │ ✅ Hadir │          │
│  │ Talkshow: Public Speaking│ 22 Sep 25  │ ❌ Absen │          │
│  │ Growth: Interview Prep   │ 10 Okt 25  │ ✅ Hadir │          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Kurikulum & Library

```
┌─────────────────────────────────────────────────────────────────┐
│  Kurikulum & Library                                            │
│                                                                 │
│  [Tab: 📋 Kurikulum] [Tab: 📚 Library]                          │
│                                                                 │
│  ═══ KURIKULUM ═══════════════════════════════════════════════  │
│                                                                 │
│  Periode: BBB4 (Jan-Jun 2026)    Goal: "Membangun softskill..." │
│                                                    [+ Topik]    │
│                                                                 │
│  ┌───┬────────────────────┬──────────┬─────────┬──────┬───────┐ │
│  │ # │ Topik              │ Detail   │ TOR     │Status│ Media │ │
│  ├───┼────────────────────┼──────────┼─────────┼──────┼───────┤ │
│  │ 1 │ CV Writing         │ Cara...  │ 📎 PDF  │ ✅   │ ▶ 📄 │ │
│  │ 2 │ Public Speaking     │ Tips...  │ 📎 PDF  │ ✅   │ ▶ 📄 │ │
│  │ 3 │ Interview Prep      │ Mock...  │ 📎 PDF  │ 🟡   │ — —  │ │
│  │ 4 │ Personal Branding   │ Build... │ 📎 PDF  │ ⬜   │ — —  │ │
│  └───┴────────────────────┴──────────┴─────────┴──────┴───────┘ │
│  Legend: ✅ done  🟡 ongoing  ⬜ planned  ▶=YouTube  📄=Slide   │
│                                                                 │
│  ═══ LIBRARY ═════════════════════════════════════════════════  │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                        │
│  │ 📚 24    │ │ 🎤 18    │ │ 📤 6     │                        │
│  │ Total    │ │ Dari     │ │ Upload   │                        │
│  │ Materi   │ │ Event    │ │ Manual   │                        │
│  └──────────┘ └──────────┘ └──────────┘                        │
│                                                                 │
│  🔍 Cari materi...  [Semua Tag ▼] [Semua Tipe ▼]  [+ Upload]   │
│                                                                 │
│  ┌────────────────────┐ ┌────────────────────┐                  │
│  │ 📄 CV Writing 101  │ │ 📄 Public Speaking │                  │
│  │ Tags: #career #cv  │ │ Tags: #softskill   │                  │
│  │ AI: "Materi ini..."│ │ AI: "Tips untuk..." │                  │
│  │ 📎 Download  ▶ YT  │ │ 📎 Download  ▶ YT  │                  │
│  └────────────────────┘ └────────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Database Mentor

```
┌─────────────────────────────────────────────────────────────────┐
│  Database Mentor                                  [+ Tambah]    │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                        │
│  │ 🎓 12    │ │ 🏠 5     │ │ 🌐 7     │                        │
│  │ Total    │ │ UNDIP    │ │ non-UNDIP│                        │
│  └──────────┘ └──────────┘ └──────────┘                        │
│                                                                 │
│  🔍 Cari nama...  [Semua Bidang ▼] [UNDIP/non-UNDIP ▼] 🔄      │
│                                                                 │
│  ┌───┬────────────────┬────────────────┬──────────┬─────┬──────┐│
│  │ # │ Nama           │ Bidang         │ Event    │ Avg │ Aksi ││
│  ├───┼────────────────┼────────────────┼──────────┼─────┼──────┤│
│  │ 1 │ Dr. Rini       │ Public Speaking│ 3 event  │ 4.8 │ 👁 ✏││
│  │ 2 │ Farhat (🏠)    │ Project Mgmt   │ 2 event  │ 4.5 │ 👁 ✏││
│  └───┴────────────────┴────────────────┴──────────┴─────┴──────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Event Talkshow

```
┌─────────────────────────────────────────────────────────────────┐
│  Event Talkshow                                   [+ Buat Event]│
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ 🎤 12    │ │ ✅ 8     │ │ 📅 3     │ │ ⚠ 2     │           │
│  │ Total    │ │ Done     │ │ Upcoming │ │ Belum    │           │
│  │ Event    │ │          │ │          │ │ Rekaman  │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│                                                                 │
│  ⚠ Alert: 2 event selesai belum ada link rekaman/materi!        │
│  [EVT-BBB4-03 Growth: Interview] [EVT-BBB4-04 Talkshow: AI]    │
│                                                                 │
│  🔍 Cari event...  [Semua Tipe ▼] [Semua Status ▼] 🔄          │
│                                                                 │
│  ┌──────┬──────────────────┬──────────┬────────┬──────┬────────┐│
│  │ Kode │ Nama Event       │ Tanggal  │ Mentor │Status│ Media  ││
│  ├──────┼──────────────────┼──────────┼────────┼──────┼────────┤│
│  │EVT-01│ CV Writing       │ 15/09/25 │ Rini   │ ✅   │ ▶ 📄  ││
│  │EVT-02│ Public Speaking  │ 22/09/25 │ Ahmad  │ ✅   │ ▶ 📄  ││
│  │EVT-03│ Interview Prep   │ 10/10/25 │ Sari   │ ⚠   │ — —   ││
│  │EVT-04│ AI for Students  │ 25/11/25 │ TBD    │ 📅   │ — —   ││
│  └──────┴──────────────────┴──────────┴────────┴──────┴────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Penugasan (Master-Detail)

```
┌─────────────────────────────────────────────────────────────────┐
│  Penugasan                                   [+ Buat Penugasan] │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                        │
│  │ 📝 8     │ │ ✅ 45    │ │ ⏳ 12    │                        │
│  │ Total    │ │ Submitted│ │ Belum    │                        │
│  │ Tugas    │ │          │ │ Kumpul   │                        │
│  └──────────┘ └──────────┘ └──────────┘                        │
│                                                                 │
│  🔍 Cari...  [Semua Batch ▼] [Semua Event ▼]                   │
│                                                                 │
│  ┌─── MASTER (Daftar Tugas) ─────────────────────────────────┐  │
│  │ Kode     │ Judul              │ Event    │Deadline│Kumpul  │  │
│  │ TGS-01   │ Refleksi CV ✅     │ EVT-01   │ 20/09  │ 14/15  │  │
│  │▶TGS-02   │ Essay Speaking 🔵  │ EVT-02   │ 30/09  │ 10/15  │  │
│  │ TGS-03   │ Portfolio ⏳       │ non-event│ 15/10  │ 3/15   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌─── DETAIL (Hasil Beswan — TGS-02) ───────────────────────┐  │
│  │ Tugas: Essay Speaking | Deadline: 30/09 | Event: EVT-02   │  │
│  │                                                            │  │
│  │ Nama            │ Status      │ File  │ Nilai │ Feedback   │  │
│  │ Ahmad Fauzi     │ ✅ graded   │ 📎    │ 85    │ "Bagus..." │  │
│  │ Siti Nurhaliza  │ 📤 submitted│ 📎    │ —     │ [Nilai]    │  │
│  │ Budi Santoso    │ ⏳ belum    │ —     │ —     │ —          │  │
│  │ Dewi Kartika    │ 📤 submitted│ 📎    │ —     │ [Nilai]    │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7-8. Keuangan (Rekonsiliasi, Database Donatur)

> ℹ Layout dasar (wizard 3 langkah, metric cards, tabel, filter) mengacu pada screenshot yang sudah ada.
> Tambahan/penegasan di bawah ini melengkapi elemen yang **belum tergambar di foto**.

**Wizard 3 langkah**: `1 Upload Mutasi` → `2 Klasifikasi` → `3 Simpan`

**Metric cards (Step Klasifikasi)**: Total · Cash In · Cash Out · **Duplikat** · **Belum Klasifikasi**
- *Duplikat* = baris yang FT Number + Nominal-nya sudah pernah disimpan (ditampilkan, tidak disimpan ulang)
- *Belum Klasifikasi* = baris status `unknown`. Tombol **Simpan** nonaktif selama angka ini > 0

**Tabel transaksi — per baris:**
```
┌────┬───────┬──────────┬───────┬──────────────┬─────────┬──────┬─────────────┬──────────────────────────┬────────────┐
│ #  │ Sheet │ Tanggal  │ Bulan │ Deskripsi    │ Nominal │ Tipe │ Kat. BSI    │ Klasifikasi              │ Status     │
├────┼───────┼──────────┼───────┼──────────────┼─────────┼──────┼─────────────┼──────────────────────────┼────────────┤
│ 65 │ Agt   │ 25/08 .. │2025-08│ Trf Dari - K │ 100.000 │ In   │ Donasi      │ [Nama donatur ▾] ✓auto   │ Inputted   │
│ 67 │ Agt   │ 25/08 .. │2025-08│ QR 250825 .. │ 100.000 │ In   │ Donasi      │ [Ketik nama… / Anonymous]│ ?  unknown │
│ 70 │ Agt   │ 26/08 .. │2025-08│ Biaya Adm    │   2.500 │ Out  │ Biaya       │ [Kategori ▾][Sub ▾] ✓auto│ Inputted   │
└────┴───────┴──────────┴───────┴──────────────┴─────────┴──────┴─────────────┴──────────────────────────┴────────────┘
```
- **Kolom Klasifikasi**:
  - Cash In → field cari/pilih **nama donatur** + opsi **"Anonymous"** di dropdown. Badge **✓auto** kalau diisi sistem (hasil auto-match nama, termasuk nama terpotong)
  - Cash Out → dropdown **Kategori** + **Sub-kategori** (dari master). Badge **✓auto** kalau ditebak sistem
- **Kolom Status**: hanya `Inputted` (hijau) / `unknown` (abu, ikon `?`). Tidak ada "Pending"
- **Baris Duplikat**: ditandai (mis. badge "DUPLIKAT" + baris redup), tidak ikut disimpan

**Tombol header**: `↻ Ulang` · **`Kategori`** (kelola master kategori cash out: tambah/edit/hapus/nonaktif) · `💾 Simpan`

**Edit setelah tersimpan** (modal/inline):
- Bisa diubah: nama donatur, Anonymous, kategori, sub-kategori, **Catatan** (notes internal finance, opsional — ikon 📝 per baris)
- **Terkunci (abu, tidak bisa diklik)**: Tipe (In/Out), FT Number, Nominal, Tanggal — ikut file bank
- **Hapus baris** → pop-up konfirmasi "Yakin hapus transaksi ini?" + opsi **Undo** setelah dihapus

**Hak akses**: Super Admin & Admin Finance = upload/klasifikasi/edit/hapus/kelola kategori; AnC = view; PCM/Viewer = tidak ada akses.

**Admin-link donatur account** (Database Donatur — kolom aksi per baris):
- Jika donatur belum ter-link (`user_id` kosong): tombol **[🔗 Hubungkan Akun]** → modal dropdown user Gmail yang belum ter-link → set `donatur.user_id`
- Jika sudah ter-link: ikon ✅ + tooltip "Terhubung ke {email Gmail}". Bisa di-unlink oleh admin

**Alert donatur belum diklasifikasi (Database Donatur view)**:
- Banner kuning di atas tabel donatur: "⚠️ {N} donatur belum masuk periode manapun — segera assign." Muncul jika ada donatur tanpa `donatur_periode` aktif.
- Filter cepat: tombol **[Belum Diklasifikasi]** untuk menampilkan hanya donatur tanpa periode.
- **Reminder musiman**: tiap masuk Juli-Agustus dan Desember-Januari (awal semester), dashboard Monitoring & Database Donatur menampilkan reminder "Saatnya update keikutsertaan donatur untuk periode baru" agar AnC mengecek siapa yang lanjut/berhenti.

---

## 8b. Overview Keuangan (read-only)

> Sub-menu Keuangan. Ringkasan arus kas dari `cashflow` yang sudah `inputted`. Filter: periode / rentang bulan / tahun.

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ Total Masuk  │ Total Keluar │ Net (M − K)  │ # Transaksi  │
│ Rp 5.645.545 │ Rp 315.609   │ Rp 5.329.936 │     36       │
└──────────────┴──────────────┴──────────────┴──────────────┘

┌─────────────────────────────┐  ┌─────────────────────────────┐
│ Tren Masuk vs Keluar /bln   │  │ Komposisi Pemasukan (pie)   │
│  ▆▆  ▆▆  ▆▆  (bar ganda)    │  │ Donasi / Pengembalian /     │
│                             │  │ Bagi Hasil + porsi Anonim   │
└─────────────────────────────┘  └─────────────────────────────┘

┌─────────────────────────────┐  ┌─────────────────────────────┐
│ Komposisi Pengeluaran (pie) │  │ Tabel ringkasan per kategori│
│ per kategori cashflow_kat.  │  │ Kategori | Nominal | %  →drill│
└─────────────────────────────┘  └─────────────────────────────┘
```
- **Tidak ada "saldo kas" absolut** (Opening Balance tidak disimpan) — yang disajikan arus & komposisi
- Klik baris tabel kategori → drilldown ke daftar transaksinya
- **Akses**: Super Admin & Finance; AnC & Viewer = view-only

---

## 9. Monitoring Donatur

```
┌─────────────────────────────────────────────────────────────────┐
│  Monitoring Donatur                              [↻ Refresh]    │
│  Matriks keikutsertaan patungan + link pesan WhatsApp siap kirim│
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐   │
│  │ 👥 98    │ │ ✅ 48    │ │ ⚠️ 12    │ │ 📅 Periode Aktif │   │
│  │ Total    │ │ Aktif    │ │ Belum    │ │ BBB #4, BBB #3   │   │
│  │ Donatur  │ │ Periode  │ │ Diklasif.│ │ (bisa >1)        │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘   │
│                                                                 │
│  ┌─ ⚠️ ───────────────────────────────────────────────────────┐ │
│  │ 12 donatur belum diklasifikasi ke periode manapun.         │ │
│  │ Segera assign di Database Donatur → kolom Periode.         │ │
│  │ [Lihat Daftar]                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌─ ℹ ────────────────────────────────────────────────────────┐ │
│  │ Cara pakai: Klik Kirim → pilih template pesan (reminder,  │ │
│  │ follow-up, dll) → WhatsApp terbuka dengan pesan siap kirim.│ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  🔍 Cari nama/kode  [Semua Periode ▼] [Semua Status ▼]         │
│                      [Semua Tag ▼]                               │
│                                                                 │
│  ┌────┬────────────────┬──────────┬─────────┬──────┬─────┐     │
│  │ ☐  │ Nama & Tag     │  Kirim   │Per.Akhir│ Bulan│Total│     │
│  │Rset│                │  (WA)    │         │ cols │     │     │
│  ├────┼────────────────┼──────────┼─────────┼──────┼─────┤     │
│  │ ☐  │ Dendy L.W.     │▶Kirim ▾  │ BBB #2  │ — —  │  —  │     │
│  │    │ DLW12025       │ pilih    │         │      │     │     │
│  │    │ 1Sem Patungan  │ template │         │      │     │     │
│  ├────┼────────────────┼──────────┼─────────┼──────┼─────┤     │
│  │ ☑  │ Ike A.H.P.     │▶Kirim ▾  │ BBB #4  │100k  │700k │     │
│  │    │ IAH22024       │ pilih    │         │100k  │     │     │
│  │    │ Sebulan Patung.│ template │         │200k  │     │     │
│  │    │ ⭐Donatur Setia│          │         │100k  │     │     │
│  └────┴────────────────┴──────────┴─────────┴──────┴─────┘     │
│                                                                 │
│  Tag: ✅Sudah Konfirmasi  ⭐Donatur Setia  🆕Baru Bergabung    │
│       🔴Perlu Follow-Up  ⚠Sulit Dihubungi ⏸Sementara Berhenti │
│       ☑Sudah Ditandai                                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Laporan

```
┌─────────────────────────────────────────────────────────────────┐
│  Laporan & Booklet                                [+ Upload]    │
│                                                                 │
│  🔍 Cari...  [Semua Tipe ▼] [Semua Periode ▼] [Public Only ☐]  │
│                                                                 │
│  ┌───┬──────────────────────┬──────────┬────────┬──────┬───────┐│
│  │ # │ Judul                │ Tipe     │Periode │Public│ Aksi  ││
│  ├───┼──────────────────────┼──────────┼────────┼──────┼───────┤│
│  │ 1 │ Booklet BBB4         │ Booklet  │ BBB4   │ ✅   │ 📎 ✏ ││
│  │ 2 │ Laporan Keuangan Q1  │ Keuangan │ BBB4   │ ✅   │ 📎 ✏ ││
│  │ 3 │ Internal Review BBB3 │ Internal │ BBB3   │ ❌   │ 📎 ✏ ││
│  └───┴──────────────────────┴──────────┴────────┴──────┴───────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Settings / Konfigurasi (admin)

> Akses: **Super Admin only**. Sub-menu: Users & Role · Template Pesan WA · Master Kategori Cashflow · **Konfigurasi AI**.

### Konfigurasi AI

```
┌─────────────────────────────────────────────────────────────────┐
│  Settings › Konfigurasi AI                                      │
│  Provider AI untuk auto-summary & auto-tag Library.             │
│                                                                 │
│  ┌───┬─────────────────┬───────────────────┬─────────┬───────┐  │
│  │ ● │ Label           │ Model             │ Provider│ Aksi  │  │
│  ├───┼─────────────────┼───────────────────┼─────────┼───────┤  │
│  │ ◉ │ Claude Opus 4.8 │ claude-opus-4-8   │anthropic│ ✏ 🗑 │  │
│  │ ○ │ GPT-4o          │ gpt-4o            │openai   │ ✏ 🗑 │  │
│  │ ○ │ Gemini 2.5 Pro  │ google/gemini-2.5 │openai*  │ ✏ 🗑 │  │
│  └───┴─────────────────┴───────────────────┴─────────┴───────┘  │
│  ◉ = aktif (dipakai)   * = via base_url OpenAI-compatible       │
│                                                  [+ Tambah]     │
│                                                                 │
│  ── Tambah / Edit Provider ──────────────────────────────────  │
│  Provider:  [anthropic ▼]  (anthropic / openai_compatible)      │
│  Label:     [Claude Opus 4.8            ]                       │
│  Model:     [claude-opus-4-8            ]                       │
│  Base URL:  [(kosong utk Anthropic)     ] ← isi utk compatible  │
│  API Key:   [••••••••••••••••] 🔒 disimpan terenkripsi          │
│                                                                 │
│  [🔌 Test Koneksi]                  [Jadikan Aktif] [Simpan]    │
└─────────────────────────────────────────────────────────────────┘
```
- API key **terenkripsi**, server-only, tak pernah tampil utuh setelah disimpan
- 1 provider **aktif** dipakai fitur AI; tambah provider = tambah baris (tanpa ubah kode)
- Kalau API gagal → materi tetap tersimpan tanpa summary (non-blocking, bisa retry)
