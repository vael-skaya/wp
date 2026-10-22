# FINAL GO-LIVE READINESS — EMPAT WEBSITE SMK 10 NOVEMBER

**Tanggal:** 22 Oktober 2026 · **Tidak ada perubahan apa pun** pada fase ini · Produksi/Rumahweb tidak diakses · Tanpa push/commit

## KEPUTUSAN
# **TERTUNDA — BLOCKER KRITIS**

Walaupun staging **secara teknis layak**, tiga prasyarat belum terpenuhi dan **tidak boleh** dilewati. Rincian di §4.

---

## 1. Bukti yang sudah ada (dari fase-fase sebelumnya)

### 1a. Komponen tepercaya ✓
| Bukti | Sumber |
|---|---|
| WordPress dari **paket resmi** (7.0.7 / Kebumen 7.1.3), bukan salinan dari situs terkompromi | Fase 7, 8A |
| Tema **Astra** & **Elementor** dipasang dari repo resmi; WPForms Lite di Cibitung/Tambun | Fase 6C, 7 |
| **Tidak** ada `wp-compat`, Elementor Pro bajakan, atau plugin tak dikenal | Fase 6J, 6K-B |
| `uploads` staging: **0 berkas PHP/JS/HTML**, 0 IoC, 0 URL produksi | Fase 7 langkah 3 |
| Media Sragen: 418/418 `verified`; uploads Sragen sumber 2.421 berkas **bersih** | Fase 6K-B, 6K-C |
| Akun: **1 administrator per situs**, dibuat baru dengan sandi acak 32 karakter; **0 akun lama** diimpor | Fase 6J-AKUN, 7 |
| `DISALLOW_FILE_EDIT` aktif pada `wp-config` staging | Fase 7 |

### 1b. Sumber kompromi — penanganan ✓
- Payload `goodsdetails.php`, `shopsdetails.php`, `wp-slgnup.gz`, `hiroshi.zip`, phpFileManager, tema rogue (`wp_zqyq`, `uvisyfyj`) **tidak pernah disalin** — staging dibangun dari nol | Fase 6K-B, 7
- `cgi-bin.zip` (1,33 GB) **tidak diekstrak** ✓
- Dump SQL sumber **tidak diimpor**; konten dipulihkan lewat konverter WXR yang menyaring IOC | Fase 4C, 7
- 163 akun hasil serangan **tidak dimigrasikan** ✓

### 1c. QA fungsional ✓ (Fase 8F/8G)
HTTP 200 semua halaman publish · 0 gambar rusak · 0 resource 4xx · 0 console error · overflow halaman 0 · menu mobile berfungsi (aria-expanded) · fokus keyboard terlihat · locale `id_ID` di keempat situs · ikon Kontak terverifikasi visual

---

## 2. Celah bukti keamanan yang MASIH relevan
| # | Celah | Dampak |
|---|---|---|
| G-1 | **Tidak ada pemindaian ulang webshell** atas isi staging saat ini. Dua percobaan saya **timeout** (I/O lambat) dan **tidak menghasilkan data** | Tidak dapat menyatakan staging bebas webshell ✗ |
| G-2 | **`wp core verify-checksums` tidak dijalankan** (butuh jaringan) | Integritas berkas inti WordPress **belum terverifikasi** dari checksum resmi ✗ |
| G-3 | Riwayat pemindaian bersih **bukan jaminan** bebas webshell | Pernyataan "bersih" harus tetap bersyarat ✓ |
| G-4 | Repo `vael-skaya/wp` **masih PUBLIK** dengan WXR memuat **18 pola email** | Paparan data; belum ditangani ✗ |
| G-5 | Sandi admin staging disimpan lokal (`C:\Users\sakha\.smk10\`, salinan identik di `recovery-work\staging\`) | Perlu rotasi saat pindah produksi ✓ |

## 3. Prasyarat operasional — belum terverifikasi
| # | Item | Status |
|---|---|---|
| O-1 | DNS 4 domain & akses cPanel | **BELUM** ✗ |
| O-2 | HTTPS/SSL aktif + canonical redirect | **BELUM** ✗ |
| O-3 | Versi PHP & `memory_limit` di hosting (staging 40M → target ≥256M) | **BELUM** ✗ |
| O-4 | Kapasitas disk & batas database hosting | **BELUM** ✗ |
| O-5 | **Restore drill** (uji pulihkan dari backup) | **BELUM PERNAH DIUJI** ✗ |
| O-6 | Email pengiriman (SMTP/forwarder) | **BELUM** ✗ |
| O-7 | Email resmi sekolah per situs | **BELUM DIKONFIRMASI** ❓ |
| O-8 | Teks kebijakan privasi final | **DRAF**, belum disetujui ❓ |

## 4. Blocker yang menahan keputusan
1. **G-1/G-2** — integritas & kebersihan berkas staging belum diverifikasi ulang → jalankan scan lokal + checksum saat I/O memungkinkan.
2. **G-4** — repo publik dengan data sensitif → privatkan (izin sudah ada, butuh `gh auth login`).
3. **O-5** — belum ada restore drill → wajib sebelum menyentuh produksi.
4. **O-1/O-2/O-3** — prasyarat hosting belum diperiksa (memang hanya bisa saat migrasi, tapi **harus menjadi langkah pertama**, bukan diasumsikan).

---

## 5. RUNBOOK GO-LIVE (satu situs per satu situs)

**Fase 0 — Persiapan (lokal, sebelum menyentuh hosting)**
1. Selesaikan G-1/G-2 (scan + checksum) dan rotasi kredensial (G-5).
2. Buat dump bersih per situs dari staging (`wp db export`) + `wp-content/uploads` + daftar plugin/tema.
3. **Uji restore drill**: pulihkan dump ke staging baru di XAMPP → verifikasi 200 + login → hapus staging uji.
4. Privatkan repo (G-4).

**Fase 1 — Verifikasi hosting (read-only, tanpa perubahan)**
5. Konfirmasi akses cPanel, DNS 4 domain, SSL, PHP ≥8.1, `memory_limit ≥256M`, kapasitas disk/DB.
6. **Backup kondisi produksi saat ini** (jika ada) sebelum menyentuh apa pun.

**Fase 2 — Pemasangan (SITUS PERTAMA: Sragen — paling sedikit konten)**
7. Buat database baru + user khusus (jangan pakai DB situs lain).
8. Pasang WordPress **dari paket resmi** (bukan dari staging lama), upload Astra/Elementor dari repo resmi.
9. Impor konten **hanya** dari dump staging yang sudah diaudit; **jangan** menyalin root/`wp-config`/akun lama.
10. Buat **akun admin baru** dengan email sekolah; hapus akun default.
11. `wp search-replace` URL staging → domain produksi (dry-run dulu, lalu eksekusi).
12. Verifikasi: 12 halaman 200, aset termuat, locale `id_ID`, menu berfungsi, ikon Kontak tampil.

**Fase 3 — Gerbang verifikasi**
13. Jika situs 1 lulus semua kriteria → ulangi untuk Kebumen, lalu Cibitung, lalu Tambun.
14. **Kriteria stop:** halaman 500, aset 404 massal, locale berubah, login gagal, atau data hilang → rollback.

**Fase 4 — Rollback**
15. Jika gagal: pulihkan backup hosting (langkah 6) + hapus instalasi baru; jangan menimpa database lama sebelum backup diverifikasi.

---

## 6. Konfirmasi batasan
Tidak ada login/akses Rumahweb ✓ · tidak ada perubahan produksi/DNS ✓ · `goly1264_ams` & `Downloads\admin.php` tidak disentuh ✓ · dump/evidence/manifest/checksum/checkpoint tidak diubah ✓ · tanpa push/commit/ubah repo ✓ · tanpa rahasia di laporan ✓ · tanpa layanan berbayar ✓

## 7. Keterbatasan laporan ini
- Scan keamanan ulang **tidak selesai** (2× timeout) — **tidak ada** data baru dari fase ini ✗
- Semua pernyataan "bersih" mengacu pada hasil fase sebelumnya ✓, bukan verifikasi baru ✗
- Saya **tidak** menyatakan situs bebas webshell secara mutlak ✓
