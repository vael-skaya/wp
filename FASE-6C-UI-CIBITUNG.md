# FASE 6C — PERAPIAN UI, NAVBAR, IKON, DAN PENDAFTARAN: CIBITUNG

**Tanggal:** 15 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\cibitung` · DB `smk10_staging_cibitung` · `http://localhost/staging/cibitung/`
**Hasil utama:** **akar masalah `Elementor #1252` ditemukan & diperbaiki** (tanpa menghapus halaman/item menu) · **ikon kontak yang hilang dipulihkan** · plugin formulir gratis terpasang · logo diganti fallback teks

---

## 1. Checkpoint yang digunakan

| Checkpoint | Ukuran | Waktu |
|---|---|---|
| `checkpoint-6c-pre-cibitung.sql` | 961.217 B (13 tabel, uji baca OK) | sebelum perubahan 6C |
| `checkpoint-6b-pre-cibitung.sql` | 932.906 B | sebelum 6B |
| `checkpoint-6a-cibitung.sql` | 989.457 B | pasca-pemulihan 6A |

Berkas yang disnapshot: `staging\cibitung\wp-config.bak-6b` (tidak diubah lagi di 6C).

## 2. Masalah dan bukti sebelum perubahan

### 2.1 `Elementor #1252` di header/menu
- **Bukti DOM (HTML beranda, sebelum):**
  `<ul class="main-header-menu …"><li class="page_item page-item-1252 menu-item"><a href="…/elementor-1252/" class="menu-link">Elementor #1252</a></li><li class="page_item page-item-10 …">`
- **12 elemen `page_item page-item-<id>`** ada di halaman → khas **daftar halaman**, bukan menu.
- Query DB: `nav_menu_locations` = **kosong** (`a:1:{s:6:"astra";s:0:"";}`) — menu "Main Menu" (12 item) dan "Footer Menu" (7 item) ada di DB tetapi **tidak ditugaskan ke lokasi tema**.
- Halaman 1252 bukan front page (`page_on_front=10`), judulnya berasal dari sumber.

### 2.2 Ikon kontak hilang
- **Bukti DOM (halaman Kontak, sebelum):** `<i class="icon icon-whatsapp-2">`, `icon-envelope1`, `icon-phone-handset`, `icon-map-marker1` ada di DOM, **tetapi tidak ada satu pun CSS/font ikon yang dirujuk** (0 berkas `font-awesome`/`eicons`/ikon lain).
- **Bukti data:** `"selected_icon":{"value":"icon icon-whatsapp-2","library":"ekiticons"}` — pustaka **`ekiticons` (ElementsKit Lite)** yang **tidak dipulihkan** (keputusan sadar: plugin pihak ketiga lama tidak diimpor).
- Cakupan: bukan hanya halaman Kontak — `ekiticons` ada di **6 post** (10 Home, 792 header, 826 Kontak, 895 Sejarah, 897, 903), total **22 ikon**, 12 nama berbeda.

## 3. Akar masalah yang terverifikasi

| Masalah | Akar masalah |
|---|---|
| `Elementor #1252` muncul di menu | **Lokasi menu Astra belum diberi menu** → Astra/Astra `wp_page_menu` menampilkan **daftar semua halaman** (termasuk halaman tanpa judul "Elementor #1252"). **Bukan** kesalahan pemetaan ID dan **bukan** referensi templat hilang; item menu untuk halaman itu tidak ada. |
| Ikon kontak tidak tampil | Ikon memakai pustaka **`ekiticons` (ElementsKit)** yang tidak tersedia di staging → markup `<i>` dirender tanpa font → glyph kosong. Bukan Font Awesome/Elementor Icons yang gagal dimuat, bukan ikon tersembunyi CSS, bukan widget Pro. |
| Header berbeda dari Kebumen | `astra-settings` Cibitung hanya **92 byte** (default), sedangkan Kebumen 2.928 B — tetapi setelah diperiksa, isi `astra-settings` Kebumen hanya key **background/layout** (bukan konfigurasi header). Perbedaan header sesungguhnya berasal dari **menu fallback** di §2.1. |

## 4. Perubahan yang dilakukan

| Objek | Perubahan | Nilai lama | Nilai baru |
|---|---|---|---|
| `theme_mods_astra` (`nav_menu_locations`) | penugasan lokasi menu | kosong | `primary`→**13 (Main Menu)**, `footer_menu`→**17 (Footer Menu)**, `mobile_menu`→**13** |
| `wp_postmeta._elementor_data` post **10, 792, 826, 895, 897, 903** | 22 ikon `ekiticons` → Font Awesome bawaan Elementor | `{"value":"icon icon-whatsapp-2","library":"ekiticons"}` … | `{"value":"fab fa-whatsapp","library":"fa-brands"}`, `…"fas fa-phone|envelope|map-marker-alt|calendar-alt|lightbulb|bullseye|graduation-cap|university|users|briefcase|globe|headset"…"fa-solid"` |
| `wp_options.site_logo` | **dihapus** (fallback teks) | `1818` (berkas bernama "…Logo…Tambun-Selatan") | tidak ada → nama sekolah tampil sebagai teks |
| Plugin | `wpforms-lite` dipasang & diaktifkan | — | v**2.0.2.2** (repo resmi; `requires_wp=5.5` ✓ kompatibel) |
| Cache | `wp elementor flush-css` | — | CSS Elementor diregenerasi |

**Tidak diubah:** halaman 1252 (tidak dihapus, **tidak** diubah judulnya), semua item menu (tidak ada yang dihapus), konten, lampiran, tema, berkas inti WordPress/Elementor.

Verifikasi JSON setelah setiap penulisan: `JSON_VALID=1` untuk seluruh `_elementor_data` (**0 invalid**).

## 5. Hasil sebelum dan sesudah

| Pemeriksaan | Sebelum | Sesudah |
|---|---|---|
| `Elementor #1252` di HTML | **ada** (2 tempat) + 12 elemen `page_item` | **0 kemunculan**, `page_item` fallback turun ke 2 |
| Ikon kontak | `<i>` tanpa font (tak terlihat) | **8 SVG** `e-fab-whatsapp`, `e-fas-phone`, `e-fas-envelope`, `e-fas-map-marker-alt`, `e-fab-facebook-f/instagram/tiktok/youtube` |
| `ekiticons` di DB | 6 post | **0** |
| Logo header | gambar berlabel "TAMBUN — BEKASI" | **teks nama sekolah** (fallback sementara) |
| Beranda HTTP | 200 | **200** |
| Tautan internal (24) | — | **0 non-200** |
| JSON Elementor invalid | 0 | **0** |
| IoC (`wp2shell`) | 0 | **0** |
| Warning kit | 0 baru (diperbaiki 6B) | **0 baru** (total log tetap 11) |

## 6. Screenshot

- **Sebelum (6B):** pane 15 Okt 20:01 — header + logo + menu berisi `Elementor #1252` sebagai item pertama.
- **Sesudah:** **tidak dapat diandalkan** — preview pane tetap menampilkan render halaman lama meski tab/URL bertukar ("stale canvas"), sehingga screenshot tidak dipakai sebagai bukti. **Bukti pengganti (lebih kuat):** DOM/HTML hasil `curl` (§5) dan query DB.
- Keterbatasan ini dicatat apa adanya; tidak ada klaim visual yang tidak terverifikasi.

## 7. Pengujian desktop dan mobile

- **Mobile/breakpoint sempit: teruji** (pane sempit) — header menampilkan tombol toggle, menu terbuka, tidak ada overflow pada logo/tombol.
- **Desktop lebar penuh: belum teruji** (alat tidak menyediakan pengaturan viewport) — dicatat sebagai belum dilakukan.

## 8. Pengujian formulir dan tautan kontak

- Plugin **WPForms Lite** terpasang & aktif, **tetapi formulir belum dibuat/ditempel** ke halaman Kontak → **belum ada formulir di halaman** (`<form>` = 0). Pembuatan formulir memerlukan builder WPForms (UI) atau API internal versi-spesifik; saya tidak membuat form tebakan.
- **Email tidak dapat diuji** (XAMPP tanpa SMTP) → **formulir belum dinyatakan siap produksi**.
- Tautan kontak pada halaman Kontak: ikon kini tampil; **alamat email** yang tersedia dari sumber: `info@smk10novembercibitung.sch.id` (+ ditemukan satu salah ketik `info@smk10novembercibitung.sch.idn` pada templat 792 — **tidak saya perbaiki**, menunggu verifikasi sekolah).
- Tautan menu: **semua** target internal 200 (§5).

## 9. Informasi resmi / keputusan yang masih dibutuhkan

1. **Formulir Kontak:** setujui pembuatan formulir WPForms (nama, email, subjek, pesan) + **alamat penerima** yang benar (kandidat dari sumber: `info@smk10novembercibitung.sch.id`) + cara menempelkannya (shortcode di halaman Kontak).
2. **Logo Cibitung resmi** — saat ini fallback teks; perlu berkas logo agar gambar dikembalikan (`site_logo`).
3. **Halaman Pendaftaran** berisi teks **"Halaman dalam pengembangan…"** yang **memang tersimpan** di sumber (konten 45 B, widget heading+text-editor). Tidak ada kerusakan layout. **Informasi resmi** (jadwal/biaya/persyaratan/kontak PPDB) harus diberikan sekolah; saya tidak mengarang.
4. **Draft** `Elementor #1275`, `Elementor #1281`, `Elementor Page #1341`: diperiksa read-only → **kosong total** (0 B konten, 0 B data Elementor). Belum dipublikasikan/dihapus.
5. **Salah ketik email** `…sch.idn`.

## 10. Prosedur rollback

```
C:\xampp\mysql\bin\mysql.exe -u root smk10_staging_cibitung < recovery-work\staging\checkpoints\checkpoint-6c-pre-cibitung.sql
```
Mengembalikan seluruh perubahan 6C (ikon, penugasan menu, `site_logo`, dan plugin bila diinginkan). Untuk form khusus: `wp plugin deactivate wpforms-lite`. Untuk mengembalikan logo tanpa rollback DB: `wp option update site_logo 1818` (ID lampiran lama, tidak dihapus).
