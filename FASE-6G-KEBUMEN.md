# FASE 6G — KEBUMEN: PENGHAPUSAN LABEL STAGING (lingkup terbatas)

**Tanggal:** 16 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\kebumen` · **Database (diverifikasi dari `wp-config.php`):** `smk10_staging_kebumen` · bukan repositori Git
**Lingkup fase ini untuk Kebumen = penghapusan label internal saja.** Tidak ada perubahan logo, tata letak, atau responsivitas (di luar cakupan; logo Kebumen tidak disentuh).

---

## 1. Checkpoint
| Aset | Nilai |
|---|---|
| `recovery-work\staging\checkpoints\checkpoint-6g-pre-kebumen.sql` | **2.087.822 B**, 13 tabel, SHA-256 `265cb7741d3df39…` |
| `…\staging\kebumen\backup-6g\{blogname-awal,blogdescription-awal,custom_css-awal}.txt` | snapshot sebelum perubahan |
| Checkpoint 5E.1 / 6C / lainnya | tidak ditimpa |

## 2. Label internal yang ditemukan dan dihapus
| Lokasi sumber | Sebelum | Sesudah |
|---|---|---|
| `wp_options.blogname` | `SMK 10 November Kebumen (STAGING pemulihan)` | **`SMK 10 November Kebumen`** (nama resmi dari dump sumber) |
| `wp_options.blogdescription` | `Merajut Mimpi Penuh Cinta` | **dibiarkan** (sudah tagline resmi sekolah — bukan label internal) |

**Bukti sesudah:** HTML beranda → `'STAGING'` **0**, `'pemulihan'` **0**, `<title>` = **SMK 10 November Kebumen – Merajut Mimpi Penuh Cinta**.

**Catatan anti-salah-simpul:** inventaris awal menunjukkan 21 post & 26 metadata "memuat label"; setelah diperiksa, kemunculan itu adalah **URL aset lokal** (`http://localhost/staging/kebumen/…`) dan nama kelas/berkas — **bukan** label internal. Karena nama situs dirender dari `blogname`, satu perubahan opsi menghapus seluruh kemunculan; **tidak ada** konten/template Elementor yang diedit dan **tidak ada** penggantian teks massal.

## 3. Label yang masih ada & alasan
- `blogdescription` "Merajut Mimpi Penuh Cinta" **dipertahankan** — ini tagline resmi sekolah (ada di dump sumber), bukan label internal.
- Tidak ada label internal lain yang ditemukan pada Kebumen.

## 4. Daftar berkas/pengaturan yang berubah
1. `wp_options.blogname` → `SMK 10 November Kebumen`.
**Tidak berubah:** konten, menu, tema, plugin, lampiran, data Elementor, logo (`site_logo` tetap 1440), CSS, permalink, berkas inti.

## 5. Pengujian
| Uji | Hasil |
|---|---|
| Beranda HTTP | **200** |
| Label "STAGING"/"pemulihan" di HTML | **0 / 0** |
| Judul halaman `<title>` | nama resmi + tagline resmi ✓ |
| Uji responsivitas/logo | **tidak dilakukan** (di luar lingkup fase ini untuk Kebumen) |
| Console/网络 | tidak diperiksa (tidak ada perubahan aset) |

## 6. Keterbatasan
- Hanya label yang diuji pada Kebumen; **tidak** ada QA responsif/logo di fase ini.
- Uji halaman lain (selain beranda) tidak dijalankan untuk label — namun karena label hanya berasal dari `blogname`, seluruh halaman otomatis bersih; klaim ini berbasis mekanisme render, **bukan** pemeriksaan tiap halaman.
- Emulasi/perangkat: tidak relevan (tidak ada uji visual).

## 7. Masalah yang belum terselesaikan
- Kebumen masih memakai **navigasi fallback daftar-halaman** (`page_item`) karena lokasi `mobile_menu`/`footer_menu` belum ditugaskan — **dilaporkan sejak 6C, belum diubah** (menunggu persetujuan).
- 3 nilai `_elementor_data` invalid (revisi lama) — tidak terkait fase ini.

## 8. Konfirmasi
- **Produksi tidak disentuh**; tidak ada deployment; **0 permintaan** ke domain produksi.
- **Sragen tidak ada staging** ⇒ tidak ada perubahan/diagnosis untuk Sragen; tidak ada nama database yang ditebak.
- Staging **Cibitung** & **Tambun** hanya diubah sesuai lingkup masing-masing.
- **Database `goly1264_ams` tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, dan tidak diubah** — tidak ada perintah apa pun yang menargetkannya.
- **`C:\Users\sakha\Downloads\admin.php` dibiarkan persis** (84.070 B, mtime 2026-10-13 12:30:18) — tidak dihapus/dipindah/diganti nama/diedit/dijalankan/dipindai.

## 9. Rollback
1. Ringan: `wp option update blogname "SMK 10 November Kebumen (STAGING pemulihan)"`.
2. Penuh: `mysql -u root smk10_staging_kebumen < recovery-work\staging\checkpoints\checkpoint-6g-pre-kebumen.sql`.
