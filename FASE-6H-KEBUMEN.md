# FASE 6H — KEBUMEN (verifikasi label + penyesuaian ukuran logo)

**Tanggal:** 16 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\kebumen` · **Database (diverifikasi dari `wp-config.php`):** `smk10_staging_kebumen` · bukan repositori Git
**Ringkas:** label bersih ✓ · tagline resmi utuh ✓ · **logo terlalu besar → diperkecil via CSS (dari 115–232 px menjadi 44–64 px), terverifikasi di 5 viewport** ✓ · satu-satunya situs yang diubah pada fase ini

## 1. Direktori & database
`C:\xampp\htdocs\staging\kebumen` → `smk10_staging_kebumen` (dari `wp-config.php`; kredensial tidak dicantumkan) · `siteurl = http://localhost/staging/kebumen`.

## 2. Checkpoint (dibuat sebelum perubahan)
| Aset | Nilai |
|---|---|
| `recovery-work\staging\checkpoints\checkpoint-6h-pre-kebumen.sql` | **2.115.226 B**, 13 tabel |
| `…\staging\kebumen\backup-6h\keadaan-awal.txt` | `site_logo=1440` · `custom_css_post_id` belum ada · theme mods awal |
| Checkpoint 5E.1 / 6C / 6G | tidak ditimpa |

## 3. Status label staging
| Uji | Hasil |
|---|---|
| `blogname` | **SMK 10 November Kebumen** ✓ |
| `blogdescription` | **Merajut Mimpi Penuh Cinta** ✓ (tagline resmi — dipertahankan, bukan label) |
| `'STAGING'` / `'pemulihan'` di HTML beranda | **0 / 0** ✓ |
| `<title>` | SMK 10 November Kebumen – Merajut Mimpi Penuh Cinta ✓ |
| Kemunculan "staging" lain | hanya URL aset lokal `localhost/staging/kebumen/…` ✓ |

## 4. Tugas tambahan — ukuran logo (sebelum & sesudah, terukur via CDP)
**Identifikasi elemen (tidak berasumsi dari Cibitung/Tambun):** logo Kebumen = lampiran **1440** (`2025/12/logo-smk-10-1.png`), dirender dengan class **`.custom-logo`** (Astra), **tanpa batasan ukuran apa pun** (`max-height: none`) dan **belum punya Custom CSS**.

| Viewport | SEBELUM | SESUDAH | Target brief |
|---|---|---|---|
| 360×800 | *belum diukur* | **31×44 px** | mobile 40–48 |
| 390×844 | **115×108 px** ✗ | **35×44 px** ✓ | mobile 40–48 |
| 412×915 | *belum diukur* | **38×44 px** ✓ | mobile 40–48 |
| 768×1024 | **232×218 px** ✗✗ | **51×48 px** ✓ | tablet ~48 |
| 1366×768 | **221×208 px** ✗✗ | **64×60 px** ✓ | desktop 56–64 |

**Kesimpulan: ukuran lama memang tidak proporsional** (3–4× lebih besar dari header/teks nama sekolah) ⇒ penyesuaian dijalankan. Rasio asli dipertahankan (`height` tetap, `width: auto`, `object-fit: contain`) sehingga logo **tidak terdistorsi/terpotong**; **berkas logo tidak disentuh** (murni CSS).

**CSS yang dipakai (selector diverifikasi berlaku di Kebumen):**
```css
.site-logo-img img, .custom-logo { height: 60px; width: auto; max-height: 60px; object-fit: contain; flex: 0 0 auto; }
@media (max-width: 921px) { .site-logo-img img, .custom-logo { height: 48px; max-height: 48px; } }
@media (max-width: 544px) { .site-logo-img img, .custom-logo { height: 44px; max-height: 44px; } }
```

## 5. Verifikasi setelah perubahan
| Uji | Hasil |
|---|---|
| Overflow dokumen (360/390/412/768/1366) | **0 px** (−14 px di desktop = scrollbar) ✓ |
| Tinggi header | 86 px (mobile) · 80 px (tablet) · 161 px (desktop, 2 baris menu seperti semula) — tidak membengkak ✓ |
| Togol menu mobile | 41 px, ada & di dalam layar ✓ |
| Judul situs | "SMK 10 November Kebumen" tampil utuh ✓ |
| Beranda HTTP | **200** ✓ |
| Elemen di luar viewport (`meluber`) | 38–46 (panel menu off-canvas tersembunyi) — **tidak** menambah lebar dokumen (overflow 0) ✓ |

Screenshot: `…\staging\kebumen\qa6h\logo-sesudah@{390x844,1366x768}.png`.

## 6. Daftar berkas/pengaturan/record yang berubah
1. Post **`custom_css` baru ID 1975** (judul `qa-6h-logo-kebumen`, isi CSS di atas).
2. Theme mod **`custom_css_post_id` = 1975** (sebelumnya belum ada).
**Tidak berubah:** konten, menu, data Elementor, tema/plugin/berkas inti, lampiran, `site_logo` (tetap 1440), permalink, tagline. **Tidak ada** perubahan logo Cibitung/Tambun dari pekerjaan ini.

## 7. Console / network
Console JavaScript tidak diperiksa khusus untuk Kebumen (tidak ada hook dijalankan pada sesi ukur Kebumen). Tidak ada permintaan ke domain produksi; semua aset yang diminta bersumber dari `localhost/staging/kebumen`.

## 8. Masalah yang belum terselesaikan (dicatat, **tidak** diperbaiki — di luar cakupan)
1. Navigasi Kebumen masih memakai **fallback daftar-halaman** (`page_item`, termasuk "Laman Contoh") karena lokasi `mobile_menu`/`footer_menu` belum ditugaskan — dilaporkan sejak Fase 6C, menunggu persetujuan.
2. 3 nilai `_elementor_data` invalid (revisi lama, tidak dirender) — tidak diperbaiki (spekulatif).
3. Pengujian konten guru nyata belum dilakukan untuk Kebumen.

## 9. Keterbatasan pengujian
Emulasi viewport CDP (**bukan Android fisik**) · halaman lain di Kebumen tidak diuji ulang (hanya beranda) · console JS tidak diperiksa untuk Kebumen · angka "sebelum" untuk 360 & 412 px tidak diukur (baru diukur setelah perubahan).

## 10. Konfirmasi
- **Produksi tidak disentuh**; tidak ada deployment; tidak ada impor ulang; tidak ada unduhan media dari produksi.
- **Sragen tidak memiliki staging** ⇒ tidak ada perubahan untuk Sragen; tidak ada nama database yang ditebak.
- **Database `goly1264_ams` tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, dan tidak diubah.**
- **`C:\Users\sakha\Downloads\admin.php` dibiarkan persis** (84.070 B, mtime 2026-10-13 12:30:18). Berkas sumber logo pengguna tidak diubah.

## 11. Rollback
1. Ringan: `wp post delete 1975 --force` lalu `wp theme mod remove custom_css_post_id` ⇒ logo Kebumen kembali ke ukuran besar seperti sebelumnya (nama/thumbnail tidak berubah).
2. Penuh: `mysql -u root smk10_staging_kebumen < recovery-work\staging\checkpoints\checkpoint-6h-pre-kebumen.sql`.
