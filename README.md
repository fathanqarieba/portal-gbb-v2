# Portal Web GBB

Spesifikasi & rencana implementasi **Portal Web Beasiswa Baik Berdampak (GBB)** — portal terpusat di `portal.baikberdampak.org` dengan 3 sub-portal: **Internal** (tim GBB), **Beswan** (penerima beasiswa), dan **Donatur** (alumni).

> **Status**: 🟢 Spec final / approved. Repo ini berisi **dokumen perencanaan saja** — belum ada kode aplikasi. Build mengikuti fase di `implementation_plan.md`.

---

## Peta Dokumen

| Dokumen | Isi |
|---------|-----|
| [PRD.md](PRD.md) | **Mulai dari sini.** Produk, user roles & matriks akses, user flow per portal, business rules, NFR, scalability, quality harness, onboarding tour |
| [implementation_plan.md](implementation_plan.md) | Tech stack, arsitektur, fase build, struktur folder, parsing engine, cron, file limits, konfigurasi AI, verifikasi |
| [docs/erd.dbml](docs/erd.dbml) | Skema database (**31 tabel, 10 group**) — buka di [dbdiagram.io](https://dbdiagram.io) |
| [docs/wireframes-internal.md](docs/wireframes-internal.md) | Wireframe Portal Internal (13 halaman) |
| [docs/wireframes-beswan.md](docs/wireframes-beswan.md) | Wireframe Portal Beswan (5 halaman) |
| [docs/wireframes-donatur.md](docs/wireframes-donatur.md) | Wireframe Portal Donatur (6 halaman) |
| [docs/colorpalette.md](docs/colorpalette.md) | Design system & token warna (light theme, selaras baikberdampak.org) |
| [docs/email-templates.md](docs/email-templates.md) | Spesifikasi semua template email (base layout + 15 template, logo & brand) |
| [docs/seeds/](docs/seeds/) | Data awal (seed) — mis. 17 pertanyaan refleksi bulanan |

## Cara Membaca

- **Product / non-engineer** → cukup `PRD.md` (§1–8 untuk produk & flow, §3 untuk hak akses per role).
- **Engineer** → `PRD.md` §1–8 untuk konteks, lalu §12 (NFR) · §13 (Scalability) · §14 (Quality Harness) sebagai Definition of Done. Lanjut `implementation_plan.md` + `docs/erd.dbml` sebelum mulai tiap fase.

## Ringkas

| | |
|---|---|
| **Stack** | React + Vite + TypeScript (SPA) · Express + Drizzle · PostgreSQL · MinIO · Better Auth · PWA |
| **Skala** | Kecil (≤100 user concurrent), tapi skema dirancang tumbuh tanpa rewrite — per-periode scoping, pagination server-side |
| **Hosting** | VPS Hostinger + Nginx + PM2 |
