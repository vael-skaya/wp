# FASE 6G — TAMBUN: LABEL STAGING, OVERFLOW, LOGO, KOTAK ELEMENTOR

**Tanggal:** 16 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\tambun` · **Database (diverifikasi dari `wp-config.php`):** `smk10_staging_tambun` · bukan repositori Git
**Ringkas:** label internal dihapus ✓ · **overflow tidak ada (koreksi 6F)** ✓ · logo Tambun 72 px desktop / 48 px mobile ✓ (identitas tidak ditukar) · kotak Elementor ditangani ✓ · CSS konten guru ditambahkan ✓

---

## 1. Checkpoint
| Aset | Nilai |
|---|---|
| `recovery-work\staging\checkpoints\checkpoint-6g-pre-tambun.sql` | **930.777 B**, 24 tabel, SHA-256 `8a7b54dea69fc37…` |
| `…\staging\tambun\backup-6g\{blogname-awal,blogdescription-awal,custom_css-awal}.txt` | snapshot sebelum perubahan |
| Checkpoint 6A–6F | tidak ditimpa |

## 2. Label internal "STAGING pemulihan"
| Lokasi | Sebelum | Sesudah |
|---|---|---|
| `wp_options.blogname` | `SMK 10 November Tambun (STAGING pemulihan)` | **`SMK 10 November Tambun Selatan`** (nama resmi dari dump sumber situs ini) |
| `wp_options.blogdescription` | *(kosong)* | dibiarkan (dump sumber juga kosong) |

**Bukti sesudah:** HTML beranda → `'STAGING'` **0**, `'pemulihan'` **0**, `<title>` = **SMK 10 November Tambun Selatan**.
Sama seperti Cibitung: kemunculan "label" pada inventaris awal adalah **URL aset lokal** (`localhost/staging/tambun/…`), bukan label; tidak ada template/konten Elementor yang perlu diedit dan **tidak ada penggantian teks massal**.

## 3. Overflow horizontal — KOREKSI temuan Fase 6F
Temuan 6F **adalah false positive**: `chrome --headless --window-size` tidak menyetel layout viewport (halaman dirender ~980 px, screenshot dipotong) — bukan overflow.

**Pengukuran sebenarnya (CDP + `Emulation.setDeviceMetricsOverride`):**
| Viewport | innerWidth | scrollWidth | overflow | elemen meluber |
|---|---:|---:|---:|---:|
| 360×800 | 360 | 360 | **0 px** | **0** |
| 390×844 | 390 | 390 | **0 px** | **0** |
| 412×915 | 412 | 412 | **0 px** | **0** |
| 1366×768 | 1366 | 1351 | −15 px (scrollbar) | 0 |
| 1920×1080 | 1920 | 1905 | −15 px | 0 |

Kontak @390: **0 px**. Tombol hamburger **ada & di dalam layar** (41 px pada 360/390/412).
**Tidak ada perubahan CSS overflow** yang dilakukan (tidak diperlukan); **tidak** memakai `overflow-x: hidden`.

## 4. Logo
| | Sebelum (6F) | Sesudah (6G) |
|---|---|---|
| Desktop | 64×64 px | **72×72 px** |
| Mobile/tablet | 48×48 px | **48×48 px** |
| Berkas | lampiran **1316** "1. Logo SMK 10 November Tambun Selatan" | sama — **tidak** ditukar dengan logo Cibitung |
| Rasio | 1:1, `object-fit: contain` | sama |

Diterapkan melalui Custom CSS situs (post `custom_css` **1412**).

## 5. Kotak Elementor
Elemen Elementor **id `6c931a7`** — `container` **tanpa anak/widget** (dari template kit sumber, sama seperti Cibitung), `min_height` besar → tampil sebagai kotak kosong di bawah seksi WELCOME.
**Perubahan:** satu aturan terarah `.elementor-element-6c931a7, [data-id="6c931a7"] { display:none !important; }`.
**Verifikasi:** `getComputedStyle` pada viewport sungguhan → **`display: none`** ✓; tidak ada elemen/section lain yang terpengaruh; kontainer itu memang kosong, jadi **tidak ada konten yang hilang**.

## 6. Konten baru guru (responsif)
Aturan terarah ditambahkan: gambar konten (`max-width:100%;height:auto`), tabel (`max-width:100%;display:block;overflow-x:auto`), iframe (`max-width:100%`).
**Uji sintetis** (disuntikkan ke DOM saat uji, tidak disimpan): heading panjang membungkus ✓; tabel ±3200 px **dibatasi 350 px** pada 390 px → **overflow 0** ✓.
**Belum konklusif:** gambar uji tidak termuat ⇒ perilaku gambar nyata belum terverifikasi.

## 7. Daftar berkas/pengaturan/record yang berubah
1. `wp_options.blogname` → `SMK 10 November Tambun Selatan`.
2. `wp_posts` (post `custom_css` **ID 1412**) → isi CSS diperbarui (logo 72/48 px, sembunyikan kontainer kosong, aturan konten).
**Tidak berubah:** konten, menu, lampiran, data Elementor, `site_logo` (tetap 1316), permalink, tema, plugin, berkas inti.

## 8. Hasil pengujian tiap viewport (sesudah)
| Viewport | 200 | overflow | logo tampil | toggle | kotak |
|---|---|---|---|---|---|
| 360×800 | ✓ | 0 px | 48×48 | 41 px | none |
| 390×844 | ✓ | 0 px | 48×48 | 41 px | none |
| 412×915 | ✓ | 0 px | 48×48 | 41 px | none |
| 1366×768 | ✓ | −15 px | **72×72** | 0 (desktop) | none |
| 1920×1080 | ✓ | −15 px | **72×72** | 0 | none |

Screenshot: `…\staging\tambun\qa6g\final2-beranda@{390x844,1366x768}.png`. Halaman **berita tetap kosong** (sumber Tambun tidak memiliki berita — bukan masalah responsif).

## 9. Keterbatasan pengujian
Emulasi viewport CDP (**bukan Android fisik**) · console JavaScript tidak diperiksa · 768×1024 tidak diukur ulang di 6G · uji gambar konten belum konklusif · pengukuran 6F tidak valid dan sudah dikoreksi.

## 10. Masalah yang belum terselesaikan
1. Uji gambar konten nyata setelah guru mengunggah.
2. Console JS & perangkat fisik.
3. Keputusan: hapus permanen kontainer kosong `6c931a7` dari data Elementor (bukan hanya disembunyikan)?

## 11. Konfirmasi
- **Produksi tidak disentuh**; 0 permintaan ke domain produksi; tidak ada deployment.
- Staging **Cibitung** hanya sesuai lingkupnya; **Kebumen** hanya label; **Sragen tidak ada staging** (tidak ditebak).
- **`goly1264_ams` tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah.**
- **`Downloads\admin.php` dibiarkan persis** (84.070 B, mtime 2026-10-13 12:30:18). Berkas sumber logo tidak diubah.

## 12. Rollback
1. Ringan: pulihkan isi CSS post 1412 dari `backup-6g\custom_css-awal.txt`; dan/atau kembalikan `blogname`.
2. Penuh: `mysql -u root smk10_staging_tambun < recovery-work\staging\checkpoints\checkpoint-6g-pre-tambun.sql`.
