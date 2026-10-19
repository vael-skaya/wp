# FASE 6K-D — RENCANA MIGRASI MEDIA & AUDIT KESIAPAN SRAGEN

**Tanggal:** 19 Oktober 2026 · **Sifat:** perencanaan offline, **tidak ada migrasi dijalankan** · **Tidak ada** staging/database dibuat, **tidak ada** impor, **tidak ada** arsip diekstrak

## 1. Ringkasan eksekutif
Media Sragen **siap dari sisi integritas**: manifest v3 memuat **418 kandidat, 418 `verified`** (107 berkas asli + 310 turunan + 1 `original_image`), total **21,1 MB**, tanpa path berbahaya/duplikat. **Yang belum siap**: (a) keputusan cakupan plugin/tema, (b) model akun baru, (c) `cgi-bin.zip` belum diinventaris, (d) payload docroot yang harus dikecualikan, (e) **paparan repo publik** yang memuat WXR staging + laporan forensik (lihat `FASE-6K-D-AUDIT-PAPARAN-REPOSITORY.md`) — perlu ditangani **sebelum** pekerjaan staging dimulai.
**Kesimpulan:** migrasi **layak direncanakan**, tetapi **belum boleh dijalankan** tanpa persetujuan eksplisit (§6).

## 2. Status input & bukti yang diperiksa
| Artefak | Ukuran | SHA-256 (potongan) | Status |
|---|---:|---|---|
| `FASE-6K-B-AUDIT-OFFLINE-SRAGEN.md` | 12.731 B | `9763e474b5a85457…` | ada ✓ |
| `FASE-6K-C-HASIL-VALIDASI-SRAGEN.md` | 9.803 B | `2f659e18f1f11973…` | ada ✓ |
| `FASE-6K-C-REKONSILIASI-MANIFEST-SRAGEN.md` | 7.936 B | `a2e086ca7aa405a5…` | ada ✓ |
| **`FASE-6K-C-MANIFEST-MEDIA-SRAGEN-v3.csv`** | 178.797 B | **`bf9ec91253007cca…`** | **referensi utama** ✓ |
| v2 / v1 (tidak dipakai) | 222.327 / 203.232 B | `0615976c…` / `c6e9fcf3…` | arsip, tidak diubah ✓ |
| Dump `smkq8953_sragen10.sql` | 80.897.366 B | `0448259bb0a11763…0b46f3af` | hash dihitung ulang, **cocok** ✓ |

**Validasi manifest v3:** 418 baris · 14 kolom · 107 lampiran unik · `candidate_type` = original 107 / derivative 310 / original_image 1 · `hash_validation_status` = **verified 418** · `migration_status` = candidate 418 · **path absolut 0 · traversal 0 · duplikat 0 · `YYYY/MM` ganda 0 · ekstensi berisiko 0 · direktori terlarang 0** · **semua 418 path ada di tree bukti** ✓ — konsisten 100 % dengan laporan rekonsiliasi ✓.

## 3. Kebijakan seleksi media (matriks keputusan)
> Integritas (SHA-256 cocok) **≠** aman. Kolom "pemeriksaan tambahan" adalah syarat **sebelum** berkas masuk staging.

| # | Kelompok | Keputusan | Alasan/bukti | Pemeriksaan tambahan wajib | Risiko bila salah klasifikasi | Syarat perubahan status |
|---|---|---|---|---|---|---|
| 1 | **Berkas asli** (`_wp_attached_file`, 107) | **`candidate`** | Semua 107 verified (path+ukuran+SHA-256); hanya media (webp/jpg/png/jpeg/pdf) | MIME vs ekstensi; magic bytes; **impor ulang hash setelah disalin** | Bila rusak: gambar tidak tampil | Hanya turun status bila hash/MIME gagal |
| 2 | **Derivative `sizes[]`** (310) | **`candidate`** | 310/310 verified; ukuran thumbnail wajar | MIME/magic bytes; verifikasi nama-vs-dimensi | Thumbnail salah tampil (bukan risiko kode) | Boleh **dibuang** tanpa merusak: WP membuat ulang |
| 3 | **`original_image`** (1 — id 1061 `SPMB-GELOMBANG-II.webp`) | **`candidate`** | 733.614 B verified; berkas resolusi penuh di samping versi `-scaled` | magic bytes; konfirmasi bukan duplikat | Alternatif: cukup andalkan `-scaled` | Turun bila terbukti duplikat |
| 4 | **PDF asli** (id 1127 `KALDIK_2025_2026.pdf`, 2.855.173 B) + 4 pratinjau JPG | **`candidate`** (PDF) / **`candidate`** (pratinjau) | PDF verified; pratinjau = turunan (`-pdf*.jpg`) | PDF: validasi header `%PDF-`, **bukan** JS di dalam PDF, jumlah halaman; pratinjau: magic bytes JPG | PDF berbahaya bila berisi JS/embed | Turun bila PDF gagal validasi |
| 5 | **2.022 `unreferenced_quarantine`** | **`quarantine`** (bukan kandidat) | Tidak dirujuk DB; 1.701 media + 166 aset Elementor + sisanya log/cache/sesi | Kurasi manual oleh sekolah bila ada yang penting | Bila dimigrasikan borongan: membawa berkas tak dikenal | Hanya **per-file** setelah kurasi + pemeriksaan keamanan |
| 6 | **Log/cache/sesi/`fm_backup`** (`wc-logs`, `cartflows-logs`, `astra-sites`, `wpforms/cache`, `wp-file-manager-pro`, `ai-builder`, `template-kits`, `forminator`) | **`exclude`** | Berisi log transaksi/pelanggan, cache, cadangan penuh (berpotensi kredensial) | — (tidak diproses) | Kebocoran data pelanggan / kredensial | Tidak berubah (kecuali keputusan bisnis tertulis) |
| 7 | **Artefak docroot & ekstensi mencurigakan** (`goodsdetails.php`, `shopsdetails.php`, `wp-slgnup.gz`, 7 arsip dropper, `archive.php` ×2, tema `wp_zqyq`/`uvisyfyj`, `mu-plugins/sso-loader.php`, `footer.php` terinjeksi, `wp-compat/`) | **`exclude`** (mutlak) | IoC terverifikasi (Fase 3A/3A-ADDENDUM, hash + dekode statis) | — | Reinfeksi penuh situs | **Tidak pernah** — tidak ada jalur pemulihan status |

**Catatan:** status dalam manifest v3 **tidak diubah**; matriks ini adalah kebijakan, bukan penulisan ulang data ✓.

## 4. Tahapan migrasi yang direncanakan (belum dijalankan)
| # | Tahap | Pilihan teknis & catatan | Risiko / prasyarat |
|---|---|---|---|
| 1 | **Isolasi & pencatatan** | Direktori `C:\xampp\htdocs\staging\sragen`; catat versi PHP/WP/tema/plugin; `WP_HTTP_BLOCK_EXTERNAL true` | Butuh persetujuan pembuatan direktori |
| 2 | **WordPress baru** | Paket resmi (versi disepakati, mis. 7.0.x/7.1.3) + **hello-elementor** + Elementor 4.3.5 — **bukan** berkas dari sumber terkompromi (`index.php` sumber 0 byte) | Butuh persetujuan unduh paket |
| 3 | **DB staging** | `smk10_staging_sragen`, prefix eksplisit **`wp_`**; user DB khusus (bukan root) bila memungkinkan | Butuh persetujuan pembuatan DB |
| 4 | **Tinjau DB sumber** | Konversi dump→WXR **offline** (parser teks); **buang**: 30 akun lama, `wp2shell_*`/menu `x`/`proof`, opsi sensitif (`sso_token`), tabel plugin | Butuh persetujuan + sanitasi WXR |
| 5 | **Seleksi media** | Berdasarkan manifest v3 + matriks §3 (418 file) | — |
| 6 | **Penyalinan teraudit** | Salin per-file dengan log `sumber → tujuan → ukuran → SHA-256`; **tanpa** menyalin berkas di luar manifest | Butuh persetujuan |
| 7 | **Verifikasi pasca-salin** | Bandingkan ukuran + SHA-256 **di staging** dengan manifest | Gagal ⇒ file tidak diimpor |
| 8 | **Pemeriksaan keamanan berkas** | MIME vs ekstensi, magic bytes, **skor entropi/pola** untuk PDF & gambar (mis. PDF ber-JS, gambar ber-embedded payload); alat tepercaya lokal | Butuh alat yang disetujui |
| 9 | **Pemulihan attachment** | Impor WXR → `attach-make`/`fixmime`/`regen` (pola teruji 6A) | Risiko ID mismatch ⇒ pakai `id-mapping` |
| 10 | **Pengujian fungsional** | Halaman, tautan media, PDF, thumbnail, responsif 360/390/412/768/1366, konsol, 0 URL produksi | Butuh QA (pola 6G/6H) |
| 11 | **QA keamanan & rollback** | Scan IoC 0; checkpoint DB sebelum/dalam tiap tahap; rollback = drop DB + hapus direktori (persetujuan) | — |
| 12 | **Gerbang persetujuan** | Lihat §6 | — |

## 5. Risiko utama
1. **Reinfeksi** bila ada berkas/akun/opsi dari sumber ikut terbawa ⇒ mitigasi: manifest + sanitasi WXR + scan IoC (0 wajib).
2. **Data pelanggan** (log Midtrans, `fm_backup`) bocor ke staging ⇒ dikecualikan mutlak.
3. **PDF/berkas berisi payload aktif** ⇒ perlu pemeriksaan tahap 8 (SHA-256 tidak mendeteksi).
4. **Media tak dirujuk** (2.022) mungkin berisi konten penting ⇒ keputusan kurasi sekolah, bukan otomatis.
5. **Elementor Pro tidak tersedia** ⇒ widget Pro tetap tak tampil (masalah sama seperti 3 situs lain).
6. **Ekspektasi pengguna**: staging ≠ produksi; tidak ada deployment pada fase ini.
7. **Paparan repo publik** (WXR staging + laporan forensik) ⇒ lihat artefak audit terpisah; ini risiko **keamanan informasi**, bukan migrasi.

## 6. Persetujuan yang masih diperlukan
| Kode | Persetujuan | Untuk apa |
|---|---|---|
| **AP-1** | Buat direktori staging + database `smk10_staging_sragen` | Tahap 1–3 |
| **AP-2** | Unduh/pakai paket resmi WP + hello-elementor + Elementor | Tahap 2 |
| **AP-3** | Pemilihan cakupan plugin (hanya inti+Elementor? tambah WPForms? WooCommerce/Midtrans?) | Tahap 2/9 |
| **AP-4** | Ekstraksi terpandu `cgi-bin.zip` (bila sekolah ingin memeriksa 1.701 media tak dirujuk) | Pekerjaan terpisah |
| **AP-5** | Kurasi manual media tak dirujuk | Tahap 5 |
| **AP-6** | Alat pemeriksaan keamanan berkas (tahap 8) | Tahap 8 |
| **AP-7** | Penanganan paparan repo publik (lihat artefak D) | Sebelum staging |
| **AP-8** | Perbaikan lampiran/heading kedua/tinggi banner 6K-A (opsional) | Site lain |

## 7. Konfirmasi
Tidak ada staging/database dibuat; tidak ada impor, penyalinan media, perubahan URL, aktivasi plugin/tema, ekstraksi arsip, atau eksekusi berkas ✓ · sumber (dump, tree bukti, inventaris, checksum, manifest v1/v2/v3, laporan lama) **tidak diubah** ✓ · produksi, `goly1264_ams`, proyek AMS, dan `C:\Users\sakha\Downloads\admin.php` **tidak disentuh** ✓ · tanpa `git push`/commit ✓.
