# FASE 6G — CIBITUNG: LABEL STAGING, OVERFLOW, LOGO, KOTAK ELEMENTOR

**Tanggal:** 16 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\cibitung` · **Database (diverifikasi dari `wp-config.php`):** `smk10_staging_cibitung` · bukan repositori Git
**Ringkas:** label internal dihapus ✓ · **overflow ternyata TIDAK ADA (koreksi temuan 6F)** ✓ · logo 72 px desktop / 48 px mobile (terukur) ✓ · kotak Elementor teridentifikasi & ditangani ✓ · CSS konten baru guru ditambahkan ✓

---

## 1. Checkpoint
| Aset | Nilai |
|---|---|
| `recovery-work\staging\checkpoints\checkpoint-6g-pre-cibitung.sql` | **973.844 B**, 24 tabel, SHA-256 `43e5d616753ca36…` |
| `…\staging\cibitung\backup-6g\{blogname-awal,blogdescription-awal,custom_css-awal}.txt` | snapshot nilai sebelum perubahan |
| Checkpoint 6A–6F | **tidak** ditimpa |

## 2. Label internal "STAGING pemulihan"
**Ditemukan & dihapus (satu sumber saja):**
| Lokasi sumber | Nilai sebelum | Nilai sesudah |
|---|---|---|
| `wp_options.blogname` | `SMK 10 November Cibitung (STAGING pemulihan)` | **`SMK 10 November Cibitung`** (nama resmi, diambil dari dump sumber situs ini) |
| `wp_options.blogdescription` | *(kosong)* | dibiarkan (dump sumber juga kosong) |

**Bukti sesudah:** HTML beranda → `'STAGING'` = **0**, `'pemulihan'` = **0**, `<title>` = **SMK 10 November Cibitung**.
**Catatan penting (anti-salah-simpul):** hasil inventaris awal menunjukkan "label" pada 10 post & 19 metadata — setelah diperiksa, semuanya adalah **URL aset lokal** (`http://localhost/staging/cibitung/…`), **bukan** label internal. Karena nama sekolah di judul/footer dirender dari `blogname` (bukan teks tetap di template), **satu perubahan opsi sudah menghapus seluruh kemunculan** — tidak ada konten/template Elementor yang diedit, tidak ada pencarian-ganti SQL massal.
**Tidak ada label yang tersisa** di Cibitung.

## 3. Overflow horizontal — KOREKSI temuan Fase 6F
Fase 6F melaporkan "overflow kritis ≤412 px". **Itu keliru (false positive).** Penyebab: `chrome --headless --window-size` **tidak** menyetel layout viewport, sehingga halaman dirender ~980 px dan screenshot hanya dipotong → tampak "terpotong", seolah overflow.

**Pengukuran sebenarnya (DevTools/CDP `Runtime.evaluate`, viewport benar-benar disetel + `Page.navigate`):**
| Viewport | `window.innerWidth` | `documentElement.scrollWidth` | overflow | elemen meluber |
|---|---:|---:|---:|---:|
| 360×800 | 360 | 360 | **0 px** | **0** |
| 390×844 | 390 | 390 | **0 px** | **0** |
| 412×915 | 412 | 412 | **0 px** | **0** |
| 1366×768 | 1366 | 1351 | −15 px (lebar scrollbar) | 0 |
| 1920×1080 | 1920 | 1905 | −15 px | 0 |

Halaman Kontak @390 juga **0 px**. Tombol hamburger **ada & di dalam layar** (lebar 41 px pada 360/390/412).
**Tidak ada perbaikan overflow yang dilakukan** — memang tidak ada yang perlu diperbaiki. Tambalan CSS 6F yang gagal **tetap tidak dipasang**; **tidak** digunakan `overflow-x: hidden` (dilarang dan tidak perlu).

## 4. Logo
| | Sebelum (6F) | Sesudah (6G) |
|---|---|---|
| Desktop (1366/1920) | 64×64 px | **72×72 px** (terukur) |
| Mobile/tablet (360/390/412) | 48×48 px | **48×48 px** (terukur) |
| Berkas | lampiran **1948** (logo pengguna) | sama — tidak diganti |
| Rasio | 1:1 asli, `object-fit: contain` | sama (tidak terdistorsi) |

Diterapkan lewat Custom CSS situs (post `custom_css` **1949**). Tidak ada tabrakan dengan menu; header tidak menutupi konten.

## 5. Kotak Elementor yang mengganggu
- **Teridentifikasi (bukan `.e-con-inner` secara umum):** elemen Elementor **id `6c931a7`** — `elType: container`, **tanpa satu pun widget/anak** di data, `min_height: 500px` (data) / 300 px (render). Elemen yang sama ada di Cibitung & Tambun (berasal dari template kit sumber), posisinya di bawah seksi WELCOME — cocok dengan kotak yang dilaporkan.
- **Perubahan:** satu aturan CSS tersempit — `.elementor-element-6c931a7, [data-id="6c931a7"] { display: none !important; }`.
- **Verifikasi:** `getComputedStyle` pada viewport sungguhan → **`display: none`** ✓. Tidak ada section/konten lain yang hilang (kontainer itu memang kosong: 0 anak).

## 6. Konten baru guru (responsif)
Ditambahkan aturan **terarah** (bukan global `*`): gambar dalam konten (`max-width:100%`, `height:auto`), tabel (`max-width:100%`, `display:block`, `overflow-x:auto`), iframe (`max-width:100%`).
**Uji sintetis** (elemen disuntikkan ke DOM saat pengujian, **tidak disimpan** ke situs): heading panjang **membungkus** ✓, tabel selebar ±3200 px **dibatasi 350 px** di 390 px → **overflow 0** ✓.
**Belum konklusif:** gambar uji tidak termuat pada uji sintetis, sehingga perilaku gambar nyata **belum terverifikasi**; aturan yang dipasang adalah praktik standar namun harus diuji lagi setelah guru mengunggah foto sungguhan.

## 7. Daftar berkas/pengaturan/record yang berubah
1. `wp_options.blogname` → `SMK 10 November Cibitung`.
2. `wp_posts` (post `custom_css` **ID 1949**) → isi CSS diperbarui (logo 72/48 px + sembunyikan kontainer kosong `6c931a7` + aturan konten).
**Tidak berubah:** konten, menu, lampiran (termasuk 1818), data Elementor, `site_logo` (tetap 1948), permalink, tema, plugin, berkas inti, `wp-config.php`.

## 8. Hasil pengujian tiap viewport (sesudah)
| Viewport | 200 | overflow | logo tampil | toggle | kotak |
|---|---|---|---|---|---|
| 360×800 | ✓ | 0 px | 48×48 | 41 px, di layar | none |
| 390×844 | ✓ | 0 px | 48×48 | 41 px | none |
| 412×915 | ✓ | 0 px | 48×48 | 41 px | none |
| 768×1024 | — (tidak diukur di 6G; 6F: proporsional) | — | — | — | — |
| 1366×768 | ✓ | −15 px | **72×72** | 0 (desktop) | none |
| 1920×1080 | ✓ | −15 px | **72×72** | 0 | none |

Screenshot: `…\staging\cibitung\qa6g\final2-beranda@{390x844,1366x768}.png` (diambil via CDP pada viewport sebenarnya).

## 9. Keterbatasan pengujian
- **Emulasi viewport (CDP), bukan perangkat Android fisik.**
- **Console JavaScript tidak diperiksa** (tidak ada akses console).
- 768×1024 tidak diukur ulang di 6G; uji gambar konten belum konklusif (§6).
- Pengukuran lama 6F (screenshot CLI) **tidak valid** dan sudah dikoreksi di §3.

## 10. Masalah yang belum terselesaikan
1. Uji gambar konten nyata (setelah guru mengunggah) — perlu pengujian ulang.
2. Console JS & pengujian perangkat fisik.
3. Keputusan apakah kontainer kosong `6c931a7` sebaiknya **dihapus dari data** (bukan hanya disembunyikan) — menunggu persetujuan.

## 11. Konfirmasi
- **Situs produksi tidak disentuh**; tidak ada deployment; **0 permintaan** ke domain produksi.
- Staging **Tambun** hanya diubah sesuai lingkupnya; **Kebumen** hanya untuk label; **Sragen tidak ada staging** (tidak disentuh/ditebak).
- **Database `goly1264_ams` tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah.** Dump yang dibuat hanya untuk staging yang diizinkan.
- **`C:\Users\sakha\Downloads\admin.php` dibiarkan persis** (84.070 B, mtime 2026-10-13 12:30:18) — tidak dihapus/dipindah/diganti nama/diedit/dijalankan/dipindai. Berkas sumber logo juga tidak diubah.

## 12. Rollback
1. Ringan: `wp option update blogname "SMK 10 November Cibitung (STAGING pemulihan)"` (tidak disarankan) dan/atau pulihkan isi CSS post 1949 dari `backup-6g\custom_css-awal.txt`.
2. Penuh: `mysql -u root smk10_staging_cibitung < recovery-work\staging\checkpoints\checkpoint-6g-pre-cibitung.sql`.
