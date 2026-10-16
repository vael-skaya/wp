# FASE 6F — QA RESPONSIVITAS ANDROID & DESKTOP: CIBITUNG

**Tanggal:** 16 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\cibitung` · **Database (diverifikasi dari `wp-config.php`):** `smk10_staging_cibitung` · URL `http://localhost/staging/cibitung/`
**Ringkas:** ukuran logo **diperbaiki & terukur** (48 px mobile / 64 px desktop) · **desktop LULUS** · **cacat overflow horizontal pada layar ≤412 px DITEMUKAN dan DIDOKUMENTASIKAN (belum terselesaikan)**

---

## 1. Ruang lingkup dan waktu pengujian
Pukul 04:33–05:5x WIB (16 Okt 2026). Lingkup: ukuran/proporsi logo, header & navigasi, overflow horizontal, konten/gambar/kolom, dan formulir pada viewport Android & desktop. **Bukan** redesign; identitas visual, konten, struktur, warna, dan tipografi dipertahankan.

## 2. Direktori staging dan database yang diverifikasi
`C:\xampp\htdocs\staging\cibitung` → `DB_NAME = 'smk10_staging_cibitung'` (dibaca dari `wp-config.php`; kredensial tidak dicantumkan) · `siteurl = http://localhost/staging/cibitung` · direktori **bukan** repositori Git.

## 3. Checkpoint yang dibuat
| Aset | Nilai |
|---|---|
| `recovery-work\staging\checkpoints\checkpoint-6f-pre-cibitung.sql` | **950.232 B**, 24 tabel, SHA-256 `171048630f4d57fd…` |
| `…\staging\cibitung\backup-6f\opsi-logo-awal.txt` | `site_logo=1948` |
| `…\staging\cibitung\backup-6f\astra-settings-awal.txt` | salinan setelan awal (81 B) |
| Checkpoint lama (6A–6E) | **tidak** ditimpa |

## 4. Halaman dan viewport yang diuji
Chrome **headless** lokal (profil terpisah) → **emulasi viewport sungguhan** (bukan perangkat fisik):

| Viewport | Beranda | Kontak | Pendaftaran | Sejarah |
|---|---|---|---|---|
| 360×800, 390×844, 412×915, 768×1024, 1366×768, 1920×1080 | ✔ (6/6) | 390 & 1366 | 390 & 1366 | 390 & 1366 |

Bukti: 12 screenshot **sebelum** + 19 screenshot **sesudah** di `…\staging\cibitung\qa6f\{before,after}\` (termasuk `overflow-cek-360x800.png` **tanpa** `--hide-scrollbars`).

## 5. Kondisi logo sebelum dan sesudah
| | Sebelum | Sesudah |
|---|---|---|
| Berkas | `logo-SMK-10-November-Cibitung.png` (lampiran **1948**, berkas pengguna) | sama (tidak diganti) |
| Aturan ukuran | **tidak ada** — gambar 740×740 dirender besar | CSS terarah: `height/width 64px` (desktop) & `48px` (≤921px), `object-fit: contain`, `flex: 0 0 auto` |
| Terlihat | besar/mendominasi header | jelas, proporsional, tidak mendominasi |

## 6. Ukuran logo aktual (terukur)
CSS yang berlaku (diverifikasi ada di HTML tersaji): `max-height:64px`/`height:64px` (desktop) dan `48px` (≤921px) dengan `width:auto`/`width:48-64px` pada gambar **persegi 740×740** ⇒ **tinggi render = 64 px (desktop)** dan **48 px (mobile/tablet)**.
Pengukuran piksel dari screenshot (deteksi warna lambang, metode heuristik — angka = bagian dalam lambang, bukan seluruh perisai):
| Viewport | tinggi terukur | lebar terukur |
|---|---:|---:|
| 360×800 / 390×844 / 412×915 | **41 px** | 38 px |
| 768×1024 | 41 px | 38 px |
| 1366×768 / 1920×1080 | **55 px** | 50 px |

## 7. Hasil pemeriksaan header dan navigasi
- **Desktop (1366 & 1920): LULUS** — logo ~64 px di kiri, nama sekolah sejajar, **menu desktop tampil rapi dalam 2 baris** (Home, Profil ▾, Akademik ▾, Fasilitas Sekolah, Prestasi / Berita/Artikel, Kontak), dropdown tertutup rapi, **tidak ada tabrakan/terpotong**, header tidak menutupi konten, hero & tombol Pendaftaran normal.
- **Mobile (≤412): CACAT** — lihat §8.

## 8. Masalah responsivitas yang ditemukan
### 8.1 KRITIS/OPSIONAL-TAMPILAN — overflow horizontal pada viewport ≤412 px (**belum terselesaikan**)
Bukti: pada 360/390/412 px, **teks terpotong di tepi kanan** — judul situs "SMK 10 November Cibitung (STAGING pemulihan)", heading hero "Unggul, Berkarakter, Kompeten, Kreatif, dan Inovatif", dan tombol **Pendaftaran**; **tombol menu (hamburger) tidak terlihat** karena terdorong keluar viewport. Terjadi **identik pada Tambun** ⇒ sistemik, bukan khas Cibitung. `<meta name="viewport" content="width=device-width, initial-scale=1">` **benar** dan `ast-mobile-header`/`mobile-menu-toggle` **ada di HTML** ⇒ masalah = **lebar dokumen > lebar viewport**, bukan meta viewport.
**Akar masalah: BELUM TERBUKTI.** Kandidat yang diperiksa: elemen Elementor `id 5326c860` ber-`width` **720 px** dengan `width_tablet` **540 px** (masih satuan px) — konsisten dengan gejala tetapi **tidak terbukti** sebagai satu-satunya penyebab.
**Percobaan perbaikan:** CSS terarah `@media (max-width:767px){ .elementor-element{max-width:100%!important} …}` → **tidak berhasil** (teks tetap terpotong) ⇒ **sudah DITARIK KEMBALI** (CSS kini hanya berisi perbaikan logo). Saya **tidak** menambah tambalan spekulatif.
**Rekomendasi:** diagnosis lanjutan memakai DevTools browser nyata (element picker + `document.documentElement.scrollWidth` + inspeksi per-elemen) untuk menemukan elemen terlebar; perbaikan kemungkinan menyentuh pengaturan layout Elementor (beberapa section) ⇒ **perlu persetujuan** karena bersifat lintas-halaman.

### 8.2 Belum diverifikasi — blok gelap di bawah seksi WELCOME
Terlihat blok hitam di bawah WELCOME pada screenshot mobile. **Dugaan: gambar lazy-load yang belum tergambar di headless** (artefak alat), **bukan** cacat situs — perlu dipastikan di browser nyata. Tidak diklaim sebagai masalah.

### 8.3 Lain-lain yang diperiksa dan TIDAK ditemukan masalah
Tidak ada `min-width`/`overflow-x` di CSS inline; tidak ada `white-space: nowrap`; kontainer/kolom lain (WELCOME, tombol, kartu) menyesuaikan lebar dengan benar; **tidak ada gambar meregang** (di dalam seksi yang benar); halaman Kontak & Pendaftaran & Sejarah tetap 200 dan layoutnya tidak rusak pada 390 px selain overflow global di §8.1; **tidak ada formulir** pada halaman yang diuji (`<form>` = 0) ⇒ pengujian formulir §4D tidak dapat dijalankan.

## 9. Perubahan CSS atau pengaturan yang dilakukan
| Objek | Perubahan |
|---|---|
| Post `custom_css` **ID 1949** (mekanisme Custom CSS bawaan WordPress/Astra) + theme mod `custom_css_post_id = 1949` | CSS **terarah hanya pada logo**: `height/width: 64px` desktop, `48px` ≤921px, `object-fit: contain`, `flex: 0 0 auto` (359 byte) |
| Percobaan aturan overflow §8.1 | **dibatalkan** — tidak ada di CSS akhir |

Tidak ada berkas tema/plugin/int WordPress yang diubah; tidak ada perubahan menu, konten, Elementor data, permalink, atau tema.

## 10. Bukti visual
`…\staging\cibitung\qa6f\` — `before/` (12), `after/` (19) termasuk `final-beranda__{android-360x800,android-390x844,android-412x915,tablet-768x1024,desktop-1366x768,desktop-1920x1080}.png`, `overflow-cek-360x800.png`, `fix-beranda__android-390x844.png`. Pengukuran: `q6f-ukur-logo-awal-v2.json`, `q6f-ukur-logo-final.json`.

## 11. Hasil pengujian ulang (sesudah perubahan)
| Uji | Hasil |
|---|---|
| Logo proporsional di 6 viewport | **LULUS** (48 px mobile, 64 px desktop; rasio asli terjaga; tidak terdistorsi) |
| Logo memakai berkas pengguna (lambang Cibitung) | **LULUS** (lampiran 1948, bukan logo Tambun) |
| Header & menu desktop | **LULUS** |
| Beranda /kontak /pendaftaran /sejarah | **200** untuk keempatnya |
| Overflow horizontal ≤412 px | **GAGAL — tercatat sebagai cacat (§8.1)** |
| Menu mobile (toggle) | **tidak dapat dipakai pada ≤412 px** karena terdorong keluar viewport (§8.1) |
| Console JavaScript | **tidak dapat diperiksa** (headless tanpa console) |
| Permintaan ke domain produksi | **0** |

## 12. Masalah yang belum terselesaikan
1. **Overflow horizontal ≤412 px** (§8.1) — butuh diagnosis DevTools lanjutan + kemungkinan perubahan layout Elementor (persetujuan).
2. Blok gelap (§8.2) — perlu verifikasi browser nyata.
3. Ukuran logo 48/64 px adalah **batas atas** dari brief; jika sekolah ingin logo lebih besar (mis. 72/96 px), cukup ubah satu nilai di CSS post 1949 — menunggu preferensi Anda.

## 13. Daftar berkas yang berubah
**Database `smk10_staging_cibitung`:** +1 baris `wp_posts` (custom_css ID 1949) & `wp_postmeta` terkait; theme mod `custom_css_post_id` = 1949 (tersimpan di `wp_options.theme_mods_astra`).
**Sistem berkas staging:** tidak ada berkas tema/plugin/int yang diubah. Bukti QA (screenshot/json) disimpan **di luar** instalasi WordPress (`recovery-work\staging\cibitung\qa6f`).
**Tidak berubah:** konten, menu, lampiran (termasuk 1818), Elementor data, `wp-config.php`, `.htaccess`, `site_logo` (tetap 1948), `site_icon` (tetap 0).

## 14. Keterbatasan pengujian
- Pengujian memakai **emulasi viewport Chrome headless**, **bukan perangkat Android fisik**.
- Tidak ada akses **console JavaScript** maupun pengukuran `scrollWidth` → akar overflow §8.1 belum pasti.
- Pengukuran tinggi logo bersifat **heuristik piksel** (bagian dalam lambang); nilai CSS (48/64 px) adalah angka yang pasti berlaku.
- Blok gelap §8.2 belum diverifikasi.
- Halaman lain (mis. `home`, `prestasi`, `fasilitas`) tidak diuji satu per satu.

## 15. Konfirmasi: tidak ada perubahan pada situs produksi
Produksi (keempat situs) tidak disentuh; tidak ada deployment; **0 permintaan** ke domain produksi selama pengujian.

## 16. Konfirmasi: situs lain di luar cakupan tidak diubah
Staging **Tambun** hanya diubah sesuai lingkupnya sendiri (CSS logo milik Tambun); **Kebumen** dan **Sragen** **tidak disentuh**; **proyek AMS** tidak disentuh.

## 17. Konfirmasi database `goly1264_ams`
**Tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah, dan tidak dijadikan target perintah apa pun** selama Fase 6F. Hanya dua dump dibuat: `checkpoint-6f-pre-cibitung.sql` dan `checkpoint-6f-pre-tambun.sql` (§3 laporan masing-masing situs).

## 18. Konfirmasi berkas `admin.php` di Downloads
`C:\Users\sakha\Downloads\admin.php` **dibiarkan persis** (84.070 B, mtime 2026-10-13 12:30:18): tidak dihapus, dipindah, diganti nama, diedit, dijalankan, atau dipindai. Folder sumber logo juga tidak diubah (hash berkas sumber tetap `d55fcb09c3b74503…`).

---

## Prosedur rollback
1. **Hanya batalkan perbaikan CSS 6F:** `wp post delete 1949 --force` lalu `wp theme mod remove custom_css_post_id` → header kembali memakai logo besar seperti sebelum 6F.
2. **Rollback penuh 6F:** `mysql -u root smk10_staging_cibitung < recovery-work\staging\checkpoints\checkpoint-6f-pre-cibitung.sql`.
3. Tidak perlu rollback berkas tema (tidak ada yang diubah). Jangan menyentuh lampiran 1818 dan berkas sumber pengguna.
