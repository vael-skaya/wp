# FASE 6F — QA RESPONSIVITAS ANDROID & DESKTOP: TAMBUN

**Tanggal:** 16 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\tambun` · **Database (diverifikasi dari `wp-config.php`):** `smk10_staging_tambun` · URL `http://localhost/staging/tambun/`
**Ringkas:** logo Tambun **tidak ditukar dengan logo Cibitung** · ukuran logo **diperbaiki & terukur** (48 px mobile / 64 px desktop) · **desktop LULUS** · **cacat overflow horizontal ≤412 px DITEMUKAN (sama seperti Cibitung), belum terselesaikan**

---

## 1. Ruang lingkup dan waktu pengujian
Pukul 04:33–05:5x WIB (16 Okt 2026). Lingkup sama dengan Cibitung: proporsi logo, header/navigasi, overflow, konten/gambar/kolom, formulir. Bukan redesign; identitas visual Tambun dipertahankan.

## 2. Direktori staging dan database yang diverifikasi
`C:\xampp\htdocs\staging\tambun` → `DB_NAME = 'smk10_staging_tambun'` (dari `wp-config.php`; kredensial tidak dicantumkan) · `siteurl = http://localhost/staging/tambun` · bukan repositori Git.

## 3. Checkpoint yang dibuat
| Aset | Nilai |
|---|---|
| `recovery-work\staging\checkpoints\checkpoint-6f-pre-tambun.sql` | **911.559 B**, 24 tabel, SHA-256 `293a8201e4eafbc6…` |
| `…\staging\tambun\backup-6f\opsi-logo-awal.txt` | `site_logo=1316` |
| `…\staging\tambun\backup-6f\astra-settings-awal.txt` | salinan setelan awal (81 B) |
| Checkpoint lama (6A–6E) | **tidak** ditimpa |

## 4. Halaman dan viewport yang diuji
Chrome **headless** lokal (emulasi viewport sungguhan, bukan perangkat fisik): beranda pada **360×800, 390×844, 412×915, 768×1024, 1366×768, 1920×1080**; Kontak, Pendaftaran, Sejarah pada **390×844 & 1366×768** — **sebelum** dan **sesudah** perubahan. Bukti di `…\staging\tambun\qa6f\{before,after}\` (12 + 19 PNG, termasuk `overflow-cek-360x800.png` tanpa `--hide-scrollbars`).

## 5. Kondisi logo sebelum dan sesudah
| | Sebelum | Sesudah |
|---|---|---|
| Berkas | lampiran **1316** "1. Logo SMK 10 November Tambun Selatan" (berkas ada, 276.210 B) — **identitas Tambun, tidak diganti** | sama (tidak diganti) |
| Kelas render | `site-logo-img` (Astra header builder) | sama |
| Aturan ukuran | tidak ada → dirender besar | CSS terarah: `64px` desktop & `48px` ≤921px, `object-fit: contain`, `flex: 0 0 auto` |

**Konfirmasi penting:** logo Cibitung (lampiran 1948) **tidak** dipakai di Tambun, dan sebaliknya — kedua situs tetap terpisah.

## 6. Ukuran logo aktual (terukur)
CSS berlaku: `height/width 64px` (desktop) dan `48px` (≤921px) pada gambar persegi 740×740 ⇒ **tinggi render 64 px (desktop) / 48 px (mobile)**.
Pengukuran piksel screenshot (heuristik, angka = bagian dalam lambang):

| Viewport | tinggi | lebar |
|---|---:|---:|
| 360×800 / 390×844 / 412×915 | **41 px** | 38 px |
| 768×1024 | 41 px | 38 px |
| 1366×768 / 1920×1080 | **55 px** | 50 px |

## 7. Hasil pemeriksaan header dan navigasi
- **Desktop (1366 & 1920): LULUS** — logo ~64 px, nama sekolah sejajar, menu desktop rapi (Home, Profil ▾, Akademik ▾, Fasilitas Sekolah, Prestasi / Berita/Artikel, Kontak), tanpa tabrakan, header tidak menutupi konten, hero & tombol Pendaftaran normal.
- **Mobile (≤412): CACAT overflow** — lihat §8.

## 8. Masalah responsivitas yang ditemukan
### 8.1 Overflow horizontal ≤412 px (**belum terselesaikan**)
Gejala identik dengan Cibitung: judul situs "SMK 10 November Tambun (STAGING pemulihan)", heading hero "Unggul, Mandiri dan Berbudaya", dan tombol **Pendaftaran** **terpotong di tepi kanan**; tombol menu/hamburger **tidak terlihat**. `<meta name="viewport" content="width=device-width, initial-scale=1">` benar dan `ast-mobile-header`/`mobile-menu-toggle` ada di HTML ⇒ **lebar dokumen > lebar viewport**.
**Akar masalah: BELUM TERBUKTI** (kandidat: elemen Elementor `id 5326c860` ber-`width` 720 px & `width_tablet` 540 px — sama pada kedua situs, konsisten dengan gejala tetapi tidak terbukti tunggal). Percobaan CSS terarah `max-width:100%!important` pada ≤767 px **tidak berhasil** ⇒ **sudah ditarik kembali**; Tambun dan Cibitung kini hanya memuat perbaikan logo.
**Rekomendasi:** diagnosis dengan DevTools browser nyata (element picker + `documentElement.scrollWidth`), lalu perbaikan pengaturan layout Elementor bila terbukti ⇒ **perlu persetujuan** (lintas halaman).

### 8.2 Belum diverifikasi — blok gelap di bawah WELCOME
Sama seperti Cibitung; dugaan **artefak lazy-load pada headless**, bukan cacat situs. Tidak diklaim sebagai masalah.

### 8.3 Yang diperiksa dan tidak bermasalah
Tidak ada `min-width`/`overflow-x`/`white-space:nowrap` yang mencurigakan di CSS inline; kontainer, tombol, dan kartu lain menyesuaikan lebar; halaman Kontak/Pendaftaran/Sejarah tetap 200 dan tidak rusak selain overflow global §8.1; **tidak ada formulir** pada halaman yang diuji (`<form>` = 0) ⇒ uji formulir tidak dapat dijalankan.

## 9. Perubahan CSS atau pengaturan yang dilakukan
| Objek | Perubahan |
|---|---|
| Post `custom_css` **ID 1412** (mekanisme Custom CSS bawaan WordPress/Astra) + theme mod `custom_css_post_id = 1412` | CSS **terarah pada logo Tambun**: `height/width 64px` desktop, `48px` ≤921px, `object-fit: contain`, `flex: 0 0 auto` (359 byte) |
| Percobaan aturan overflow §8.1 | **dibatalkan** (tidak ada di CSS akhir) |

Tidak ada berkas tema/plugin/int WordPress, menu, konten, Elementor data, atau tema yang diubah.

## 10. Bukti visual
`…\staging\tambun\qa6f\{before,after}\` — termasuk `final-beranda__*.png` (6 viewport), `final-beranda__android-390x844.png`, `overflow-cek-360x800.png`.

## 11. Hasil pengujian ulang
| Uji | Hasil |
|---|---|
| Logo proporsional 6 viewport | **LULUS** (48/64 px, rasio asli, tidak terdistorsi) |
| Logo Tambun (bukan Cibitung) | **LULUS** |
| Header & menu desktop | **LULUS** |
| Beranda /kontak /pendaftaran /sejarah | **200** keempatnya |
| Overflow horizontal ≤412 px | **GAGAL — tercatat §8.1** |
| Menu mobile (toggle) | **tidak dapat dipakai pada ≤412 px** (§8.1) |
| Console JavaScript | **tidak dapat diperiksa** |
| Permintaan ke domain produksi | **0** |

## 12. Masalah yang belum terselesaikan
1. Overflow horizontal ≤412 px (§8.1) — sama seperti Cibitung, menunggu diagnosis DevTools + persetujuan.
2. Blok gelap (§8.2) — verifikasi browser nyata.
3. Ukuran logo 48/64 px adalah batas atas brief; bisa dinaikkan bila diinginkan (ubah CSS post 1412).

## 13. Daftar berkas yang berubah
**Database `smk10_staging_tambun`:** +1 baris `wp_posts` (custom_css ID 1412) & `wp_postmeta` terkait; theme mod `custom_css_post_id` = 1412.
**Sistem berkas staging:** tidak ada berkas tema/plugin/int yang diubah. Bukti QA disimpan di luar instalasi (`recovery-work\staging\tambun\qa6f`).
**Tidak berubah:** konten, menu, lampiran, Elementor data, `wp-config.php`, `.htaccess`, `site_logo` (tetap 1316), `site_icon`.

## 14. Keterbatasan pengujian
Emulasi viewport headless (**bukan Android fisik**) · tanpa console JS/`scrollWidth` ⇒ akar §8.1 belum pasti · pengukuran logo heuristik · blok gelap belum diverifikasi · halaman lain tidak diuji satu per satu · halaman **berita masih kosong** (sumber Tambun tidak punya berita — bukan masalah responsif).

## 15. Konfirmasi: tidak ada perubahan pada situs produksi
Produksi tidak disentuh; tidak ada deployment; **0 permintaan** ke domain produksi.

## 16. Konfirmasi: situs lain di luar cakupan tidak diubah
Staging **Cibitung** hanya diubah sesuai lingkupnya sendiri; **Kebumen** dan **Sragen** tidak disentuh; **proyek AMS** tidak disentuh.

## 17. Konfirmasi database `goly1264_ams`
**Tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah** selama Fase 6F. Dump yang dibuat hanya `checkpoint-6f-pre-cibitung.sql` dan `checkpoint-6f-pre-tambun.sql`.

## 18. Konfirmasi berkas `admin.php` di Downloads
`C:\Users\sakha\Downloads\admin.php` **dibiarkan persis** (84.070 B, mtime 2026-10-13 12:30:18) — tidak dihapus, dipindah, diganti nama, diedit, dijalankan, atau dipindai. Folder/berkas sumber logo tidak diubah.

---

## Prosedur rollback
1. **Hanya batalkan perbaikan CSS 6F Tambun:** `wp post delete 1412 --force` lalu `wp theme mod remove custom_css_post_id`.
2. **Rollback penuh 6F Tambun:** `mysql -u root smk10_staging_tambun < recovery-work\staging\checkpoints\checkpoint-6f-pre-tambun.sql`.
3. Tidak ada berkas tema yang perlu dikembalikan; jangan menyentuh lampiran logo Tambun (1316) atau aset situs lain.
