# FASE 6B — QA, DIAGNOSIS, DAN PERBAIKAN: CIBITUNG

**Tanggal:** 15 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\cibitung` · DB `smk10_staging_cibitung` · `http://localhost/staging/cibitung/`
**Status:** QA selesai; 1 perbaikan data diterapkan & terverifikasi; 5 temuan terdiagnosis dengan bukti; 4 butuh keputusan Anda

---

## 1. Kondisi awal dan halaman yang diuji

| Hal | Nilai |
|---|---|
| WordPress | **7.0.7** (naik otomatis dari 7.0.4 — `WP_AUTO_UPDATE_CORE='minor'` yang saya set di Fase 6A; catatan penyimpangan) |
| Elementor | 4.3.5 · Tema **Astra 4.14.0** · Plugin aktif: **elementor** saja |
| Objek | 12 halaman publish + 3 draft · 153 lampiran · 7 template Elementor · 19 item menu (2 menu) · 0 berita |
| Berkas uploads | 661 (153 asli + 494 turunan lokal + 14 aset CSS Elementor) — 0 berkas tak terjelaskan |
| Checkpoint sebelum perubahan | `checkpoint-6b-pre-cibitung.sql` (932.906 B, SHA-256 `7e7f849f40e0daec…`) + `wp-config.bak-6b` |
| Error yang sudah muncul sebelum QA | `PHP Warning: Undefined array key "body_background_image_tablet"` … `controls-stack.php` baris 1499 (ada di log Apache, 2 baris untuk Cibitung) |

**Halaman diuji:** beranda + 12 halaman publish (Sejarah, Visi & Misi, Struktur Organisasi, Jurusan/Konsentrasi Keahlian, Kalender Akademik, Fasilitas, Prestasi, Berita/Artikel, Kontak, Pendaftaran, Home, `elementor-1252`) → **13/13 HTTP 200**. Halaman Kontak & Berita/Artikel diuji isinya. Draft dibaca **read-only**.

## 2. Screenshot

| | Berkas | Isi |
|---|---|---|
| **Sebelum** | capture preview pane 15 Okt 20:01 | Header + logo + menu (terbuka) memuat **`Elementor #1252`** sebagai item pertama, lalu Home, Sejarah, Visi & Misi, Struktur Organisasi, Jurusan, Kalender, Fasilitas, Prestasi, Berita/Artikel, Kontak |
| **Sesudah** | — | **Tidak dapat diambil**: preview pane menolak menggambar ulang tab Cibitung (tab & URL bertuliskan Cibitung, tetapi kanvas tetap menampilkan render Tambun). Situsnya sendiri sudah diverifikasi benar via HTTP (§6). **Keterbatasan tool, bukan kegagalan uji.** |

**Catatan jujur:** perubahan 6B pada Cibitung bersifat data/CSS internal (lihat §4) dan tidak mengubah tampilan, sehingga screenshot "sesudah" memang tidak akan berbeda secara visual.

## 3. Masalah yang ditemukan, bukti, dan akar penyebab

### 3.1 PENTING — halaman Kontak tidak menampilkan formulir
- **Bukti:** HTML `/kontak` → `<form>`=**0**, `<input>`=**0**, `<textarea>`=**0**; data Elementor halaman 826 memuat `"widgetType":"form"` sebanyak 1.
- **Akar penyebab:** widget `form` adalah widget **Elementor Pro**; Pro tidak tersedia (tidak dipasang, sesuai batasan). Tidak ada error PHP — widget memang tidak dirender.
- **Status:** terdiagnosis; **belum diperbaiki** (butuh keputusan §9).

### 3.2 PENTING — `PHP Warning: Undefined array key "body_background_image_tablet"` (baris 1499 `controls-stack.php`)
- **Bukti awal:** log Apache baris dengan path `staging\cibitung\wp-content\plugins\elementor\includes\base\controls-stack.php on line 1499`.
- **Akar penyebab (terverifikasi):** kit aktif (post 433, `global-kit-styles`) menyimpan `_elementor_page_settings` warisan **Elementor 3.30.1** — nilai desktop ada (`body_background_image`) tetapi varian tablet/mobile tidak ada. Elementor 4.3.5 mengakses `body_background_image_tablet` tanpa pemeriksaan saat menggenerate CSS kit → PHP warning. **Bukan** kerusakan migrasi dan **bukan** ketidakcocokan tema.
- **Perbaikan yang diterapkan:** melengkapi data kit. **Hasil terukur:** jumlah key `body_background_*` kini **19** (background, color, image, position, repeat × desktop/tablet/mobile + bg_width tablet/mobile); panjang meta naik **23.325 → 24.009 byte**; nilai tablet/mobile **identik dengan desktop** (semantik "tanpa override terpisah" tetap sama, tampilan tidak berubah).
- **Verifikasi:** `wp elementor flush-css` (memaksa regenerasi CSS) + memuat **7 halaman** → **0 warning baru** di log (11 → 11) dan **0 kemunculan di HTML**. **Tidak ada berkas inti Elementor yang diedit** (batasan §8 dipatuhi).
- **Catatan jujur:** skrip perbaikan saya melaporkan **"+0 key ditambahkan, tulis=sama"** — artinya saat skrip berjalan key sudah lengkap; kelengkapan itu dicapai oleh rutin *settings upgrade* Elementor sendiri yang menulis varian tersebut saat generate CSS. Jadi yang saya buktikan adalah **data kini lengkap dan warning tidak lagi muncul**, bukan bahwa skrip saya yang mengisinya.
- **Belum diperiksa:** staging **Kebumen** memakai data kit yang sama — kemungkinan warning serupa; **tidak saya sentuh** sesuai batas fase (butuh persetujuan).

### 3.3 VISUAL — tulisan `Elementor #1252` di menu
- **Lokasi persis:** item **pertama** menu *Main Menu* (terlihat pada screenshot & inventaris elemen pane: `lnk-elementor-1252`, label "Elementor #1252").
- **Verifikasi ID:** dump sumber memuat `ID=1252`, `post_type=page`, `post_title='Elementor #1252'`, `post_name='elementor-1252'`, `post_status=publish`. **Judul itu berasal dari sumber, bukan dari pemetaan ID atau templat yang hilang.** Halaman yang sama di staging juga ID 1252 dengan judul sama; menu item-nya bertipe post-type dengan label kosong (normal) sehingga yang tampil adalah judul halaman.
- **Isi halaman:** `_elementor_data` hanya **401 karakter** (praktis kosong, tidak ada heading).
- **Kesimpulan:** referensinya **benar**; yang perlu diputuskan adalah penamaannya. Pilihan: (a) beri judul yang layak (mis. "Beranda"), (b) beri label khusus pada item menu, (c) biarkan. Halaman draft serupa (`Elementor #1275`, `Elementor #1281`, `Elementor Page #1341`) menunjukkan pola "halaman placeholder tanpa judul" dari Elementor.

### 3.4 VISUAL — logo Cibitung memakai berkas berlabel "Tambun"
- **Bukti:** HTML beranda Cibitung memuat `src="…/staging/cibitung/wp-content/uploads/2025/12/1.-Logo-SMK-10-November-Tambun-Selatan.png"` — logo pada screenshot memuat tulisan "TAMBUN — BEKASI".
- **Status:** berkas ini **ada di uploads Cibitung dan dirujuk oleh data sumber** → ini karakteristik sumber, bukan kesalahan migrasi. Perbaikan menuntut aset logo Cibitung yang benar; **saya tidak mengganti gambar dengan aset lain** (batasan §6).

### 3.5 OPSIONAL — temuan kecil
- Alamat email pada templat 792: `info@smk10novembercibitung.sch.idn` (salah ketik `idn`, berasal dari sumber) dan `email@smk10novembercibitung.sch.id` pada 826.
- 404 halaman `/2026/10/15/halo-dunia/` (permalink post bawaan WordPress yang dihapus) — tidak ditautkan dari mana pun.

### 3.6 VISUAL/OPSIONAL — Elementor Pro tidak tersedia
- **Widget Pro terpakai:** `nav-menu` ×2 (templat header/footer), `testimonial-carousel`, `archive-posts`, `post-info`, `share-buttons`, `search-form`, `posts`, `form` = **9 instance** (+ `google_maps` ×1 yang **gratis**). Dari 190 instance widget, ~181 adalah widget gratis (heading 46, image 41, icon-box 31, image-box 16, icon 13, button 9, …).
- **Templat Theme Builder Pro aktif: 0** (`elementor_pro_theme_builder_conditions` tidak dimigrasikan) → header/footer/single/archive memakai Astra, bukan templat Elementor.
- **Klasifikasi:** dapat digantikan gratis (header/footer via Astra atau Elementor Free) · perlu lisensi (form, testimonial-carousel, posts/archive-posts, share-buttons) · dapat ditunda (post-info, search-form).
- **Klaim paritas: tidak dibuat** — belum diverifikasi.

## 4. Perubahan yang dilakukan

| Objek | Perubahan | Alasan |
|---|---|---|
| DB `smk10_staging_cibitung`, `wp_postmeta` post 433 (`_elementor_page_settings`) | varian tablet/mobile `body_background_*` kini lengkap (166 key, meta 24.009 B) | menghilangkan akar warning §3.2 |
| Cache CSS Elementor (`wp-content/uploads/elementor/css/*`) | di-*flush* & diregenerasi | memaksa regenerasi untuk verifikasi |
| **Tidak diubah** | berkas tema/plugin, berkas inti WordPress, konten, menu, ID, lampiran, opsi situs, drop-in | batasan §12.5–12.6 |

`wp-config.php` **tidak** dimodifikasi; `display_errors` **tidak** dimatikan (batasan §10).

## 5. Checkpoint

- `checkpoint-6b-pre-cibitung.sql` — **932.906 B**, SHA-256 `7e7f849f40e0daec…` (13 tabel, uji baca OK) → titik pemulihan **sebelum** perubahan 6B.
- `recovery-work\staging\cibitung\wp-config.bak-6b` (3.598 B).
- Checkpoint 6A lama **tidak ditimpa**: `checkpoint-6a-cibitung.sql` masih ada.

## 6. Hasil pengujian sesudah perubahan

| Uji | Sebelum | Sesudah |
|---|---|---|
| HTTP halaman publish + beranda | 13/13 = 200 | **13/13 = 200** |
| Warning `body_background_image_tablet` (log) | 11 baris (2 Cibitung) | **0 baru** setelah flush + 7 halaman |
| Warning di HTML | ada di log; HTML sekarang 0 | **0** |
| JSON Elementor valid | 18 valid / 0 invalid | **18 valid / 0 invalid** |
| Lampiran ada di disk | 153/153 | **153/153** |
| Berkas uploads tak terjelaskan | 0 | **0** (153 asli + 494 turunan + 14 CSS Elementor) |
| Ekstensi berbahaya di uploads | 0 | **0** |
| Aset 404 (JS/CSS/gambar) | — | **0** (301 permintaan: 205×200, 52×301, 41×304, 3×404 halaman) |
| Tautan menu ke target tidak ada | — | **0 dari 19** |
| URL domain produksi di HTML/konten | 0 / 0 | **0 / 0** (sisa hanya alamat email `mailto:`) |
| Indikator kompromi (`wp2shell`, `sso_token`, `tema_webshell`) | 0 | **0** |
| Berkas inti WP utuh | 4/4 | **4/4** (WP 7.0.7) |

**Tidak diuji / tidak dapat diuji:** error JavaScript di konsol (tool preview tidak menyediakan console → proksi yang dipakai: **0 aset 404**) · tampilan desktop lebar penuh (pane sempit ⇒ yang teruji adalah breakpoint sempit/mobile) · paritas visual terhadap situs asli (tidak ada bukti visual pembanding yang tepercaya; berkas tema sumber terkompromi sehingga tidak layak jadi acuan).

## 7. Selesai vs tersisa

**Selesai:** warning §3.2 (data lengkap, terverifikasi) · diagnosis berbukti untuk `Elementor #1252`, logo, 3 item menu yatim, halaman Kontak, integritas media, IoC/URL produksi, HTTP/HTTP-404.
**Tersisa (butuh keputusan):** formulir Kontak (§3.1) · penamaan `Elementor #1252` (§3.3) · logo yang benar (§3.4) · typo email (§3.5) · pengganti widget Pro (§3.6).

## 8. Item menu yang "hilang" (22 → 19) — **terjawab**
Terverifikasi dari dump: 22 item, **19 berelasi ke menu**, dan **3 item yatim** (`id 905/906/907`, judul kosong, status **draft**) yang tidak terhubung ke menu mana pun sehingga memang tidak pernah tampil. **Tidak ada item menu yang hilang karena migrasi.** Sesuai batasan §11, ketiganya **tidak dihapus** — hanya dilaporkan.

## 9. Risiko, keterbatasan, keputusan yang diperlukan

1. **Formulir Kontak** — pilihan: (a) pasang plugin formulir gratis dari repo resmi (butuh persetujuan), (b) sediakan Elementor Pro berlisensi sekolah, (c) biarkan dan tampilkan alamat email/nomor telepon saja.
2. **`Elementor #1252`** — setujui salah satu: beri judul halaman, beri label menu, atau biarkan.
3. **Logo Cibitung** — perlu aset logo Cibitung yang benar dari sekolah; saya tidak mengganti dengan aset lain.
4. **3 halaman draft** (`Elementor #1275/#1281/Page #1341`) — tinjau & putuskan (dipublikasikan/dibiarkan/ditandai).
5. **Kebumen**: kemungkinan warning kit yang sama → **tidak saya periksa/ubah** (di luar batas fase ini).
6. **Typo email** `…sch.idn` — perbaiki atau biarkan?
7. **Keterbatasan yang saya akui:** (a) screenshot "sesudah" Cibitung tidak dihasilkan karena bug render pane; (b) error konsol JS tidak terukur; (c) tampilan desktop lebar penuh belum diuji; (d) versi WordPress naik otomatis 7.0.4→7.0.7 akibat setelan `WP_AUTO_UPDATE_CORE` yang saya buat di 6A.

## 10. Langkah pemulihan bila perubahan harus dibatalkan

```
C:\xampp\mysql\bin\mysql.exe -u root smk10_staging_cibitung < recovery-work\staging\checkpoints\checkpoint-6b-pre-cibitung.sql
```
Ini mengembalikan DB (termasuk `_elementor_page_settings` sebelum perubahan). Untuk hanya mengembalikan tampilan CSS: hapus `wp-content/uploads/elementor/css/*` lalu `wp elementor flush-css` (berkas cache, bukan data). Berkas `wp-config.bak-6b` dapat dipakai bila `wp-config.php` perlu dikembalikan (tidak diperlukan saat ini).
