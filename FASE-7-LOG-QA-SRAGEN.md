# FASE 7 — LOG QA SRAGEN

**Tanggal:** 19 Oktober 2026 · Semua hasil di bawah benar-benar dijalankan; yang belum dijalankan dinyatakan belum.

| # | Uji | Hasil | Bukti |
|---|---|---|---|
| 1 | Staging dapat dibuka | **LULUS** | `/`, `/sejarah/`, `/kontak/`, `/struktur-organisasi/` → 200 |
| 2 | Tabel & prefix | **LULUS** | 13 tabel `wp_*` |
| 3 | Akun lama tidak diimpor | **LULUS** | `wp_users` = 1 (`sragen_admin`); impor `--authors=skip` |
| 4 | Media tersalin & hash cocok | **LULUS** | 418 disalin (107 + 311 sudah ada), **0 gagal** |
| 5 | Uploads bebas PHP/JS/HTML | **LULUS** | `find` → 0 |
| 6 | IoC | **LULUS** | opsi/meta 0 / 0 |
| 7 | URL domain produksi | **LULUS** | 24 penggantian, sisa 0 |
| 8 | Menu | **PARSIAL** | 19 item, 2 menu; penugasan lokasi belum diverifikasi |
| 9 | JSON Elementor | **GAGAL (1)** | 18 valid, 1 tidak valid |
| 10 | Jumlah lampiran | **GAGAL** | 61 post vs 107 sumber |
| 11 | Jumlah artikel | **GAGAL** | 1 vs ~96 sumber |
| 12 | Thumbnail | **PARSIAL** | 8/61 |
| 13 | QA responsif 360/390/412/768/1366 | **BELUM** | — |
| 14 | Konsol JavaScript | **BELUM** | — |
| 15 | Label staging pada judul situs | **GAGAL** | masih "(STAGING pemulihan)" |
| 16 | Perbaikan UI | **BELUM** | — |
| 17 | Warning PHP baru | **BELUM** | — |

**Kesimpulan:** staging layak ditinjau sebagai **kerangka** (halaman terbuka, media lengkap & bersih), **belum** layak disebut pulih penuh.
