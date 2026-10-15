# FASE 5D — PERBAIKAN MEDIA & UI STAGING KEBUMEN (laporan)

**Tanggal:** 14–15 Oktober 2026 · **Target:** `C:\xampp\htdocs\staging\kebumen` · DB `smk10_staging_kebumen` · `http://localhost/staging/kebumen`
**Checkpoint rollback:** `recovery-work\staging\checkpoints\checkpoint-04-pre-5d.sql` (2.003.111 B, SHA-256 `099798b8…`)
**Snapshot opsi sebelum perubahan:** `recovery-work\staging\backup-5d\opt-*.txt`

---

## 1. Masalah media: ditemukan & diperbaiki

### 1.1 Koreksi jumlah (transparansi)
Angka **27** pada Fase 5B berasal dari analisis teks WXR dengan ekstraksi path yang terbatas. Pengukuran ulang langsung dari database staging — dengan menangani **bentuk ter-escape JSON Elementor** (`\/wp-content\/uploads\/`) yang sebelumnya terlewat — menghasilkan **32 rujukan** ke berkas yang tidak ada. Angka inilah yang dipakai di sini.

### 1.2 Diagnosis dan klasifikasi

| Klasifikasi | Jml | Keterangan |
|---|---:|---|
| **Byte-IDENTIK di 3 situs saudara** | **29** | Tambun/Cibitung/Sragen punya berkas dengan path & nama sama dan SHA-256 identik ⇒ **aset template kit bersama** (kit “Skoola”), bukan foto khas Kebumen |
| **Hanya ada di salinan bukti Kebumen** | **1** | `elementor/thumbs/logo-smk-10-1-<hash>.webp` — turunan milik Kebumen sendiri |
| **Tidak ada di mana pun** | **2** | `elementor/thumbs/img_mp_001-<hash>.webp` · `2025/10/logo_SMK.png` |

**Penyebab (bukan beragam seperti dugaan awal):**
1. Berkas-berkas itu **tidak termasuk 122 lampiran** yang dimigrasikan (tidak ada di manifes) dan **tidak punya record lampiran** di DB.
2. Rujukannya di konten/`_elementor_data` berbentuk **URL**, bukan ID lampiran ⇒ tidak ada pemetaan ID yang bisa memperbaikinya; hanya penyediaan berkas (atau penggantian URL) yang menyelesaikannya.
3. Sebagian besar adalah aset kit yang di situs Kebumen **belum pernah tersalin** (ada di tiga situs lain), bukan file yang hilang karena serangan.

### 1.3 Perbaikan yang dijalankan (hanya yang terverifikasi)

Sumber berkas dipilih dengan aturan eksplisit: *byte identik lintas situs* = aset kit bersama (sah disalin) · *hanya di salinan bukti Kebumen* = berkas milik situs ini (sah disalin) · *tidak ada di mana pun* = **tidak ditebak**.

| Ukuran | Hasil |
|---|---|
| Berkas disalin | **30** (29 aset kit + 1 thumb Kebumen) |
| Verifikasi SHA-256 sebelum/sesudah | **30 COCOK**, 0 tidak cocok |
| Total byte | **4.565.510 B** |
| Berkas jenis kode (`.php/.js/…`) | **0** (ditolak oleh aturan skrip) |
| Berkas staging yang ditimpa | **0** (bila isi berbeda, penyalinan ditolak) |
| Rujukan tanpa berkas | **32 → 2** |
| Log per berkas | `kebumen\export\_5d-media-copy-log.tsv` (sumber + hash src & dst) |
| Rincian per sumber | `kebumen\export\_5d-media-sumber.tsv` (4 root dibandingkan) |

### 1.4 Dua rujukan yang tersisa (dilaporkan, tidak ditebak)

| Berkas | Lokasi rujukan | Status |
|---|---|---|
| `elementor/thumbs/img_mp_001-<hash>.webp` | `_elementor_page_settings` | **Turunan Elementor.** Berkas sumbernya (`2025/12/img_mp_001.webp`) kini **ada**; Elementor akan membangkitkan thumb-nya saat elemen yang memakainya dirender (belum, karena templat Pro yang memakainya tidak aktif). |
| `2025/10/logo_SMK.png` | `_elementor_page_settings` kit 5 & 433; `_elementor_data` Header (792) & Footer (799) | **Hilang di keempat root bukti.** Tidak diganti. Yang tampak di situs kini logo sah `1440 → 2025/12/logo-smk-10-1.png` (49.549 B, terverifikasi). **Butuh keputusan Anda** bila kit/templat ingin menunjuk logo itu. |

**Tidak dilakukan (patuh batasan):** menebak berkas pengganti · mengunduh dari situs produksi · mengekstrak `cgi-bin.zip` atau arsip lain · menghapus salinan `-1` (247 berkas tetap utuh).

### 1.5 Uji setelah perbaikan
- **V-11:** dari **270 rujukan media yang benar-benar dirender** pada 18 halaman/berita → **0 gagal dimuat** (sebelum 5D: 217 rujukan, 0 gagal — jumlah naik karena lebih banyak gambar kini tampil).
- **V-04:** 10 `_thumbnail_id` → seluruhnya menunjuk lampiran sah.
- Berkas di `uploads`: **720 → 759** (30 disalin + 9 turunan Elementor yang dibangkitkan otomatis).

---

## 2. Masalah UI: per halaman dan penyebabnya

Diperiksa di browser lokal (panel pratinjau Hermes) dan dibandingkan dengan HTML/CSS terender serta opsi situs lama dari dump.

| Prioritas | Gejala sebelum | **Penyebab (terbukti)** | Tindakan |
|---|---|---|---|
| 1. Header/logo/menu | Logo tidak muncul; menu belum menempati lokasi header/footer | Opsi `site_logo` **tidak ada**; `nav_menu_locations` belum ada; logo (lampiran 1440) ada tapi tak dirujuk | `site_logo=1440`; `nav_menu_locations`: primary→**11**, secondary_menu→**12** (pemetaan dari term menu lama 3 & 7) ✓ |
| 2. Home: hero, warna, tipografi, tombol | Hero/warna/tombol memakai gaya bawaan tema; tipografi global salah | **Kit Elementor aktif = 5 (Default Kit)**, bukan **433 (Kit Styles: Skoola)** ⇒ CSS kit tidak dibangkitkan | `elementor_active_kit=433` + `wp elementor flush-css` → **post-433.css terbentuk**; elemen kit **18 → 158** ✓ |
| 2. Latar & layout konten | Latar halaman berbeda | `astra-settings` masih bawaan (79 B) | Diimpor (2.915 B) — `theme-auto-version` dipertahankan 4.14.0 ✓ |
| 1. Footer | Area widget footer kosong | `sidebars_widgets` + `widget_block` tidak ada | Diimpor ✓ |
| 1. Tagline | Kosong | `blogdescription` tidak ada | Diimpor: **“Merajut Mimpi Penuh Cinta”** ✓ |
| 3. Kontak (form) | Widget **form** (Pro) tidak dirender | Elementor Pro tidak dipasang | **Belum** — usulan padanan gratis: plugin form gratis (mis. Contact Form 7) atau blok form WP; **butuh keputusan** |
| 4. Berita/arsip/single post | Memakai tata letak bawaan **Astra**, bukan desain “Single News”/“Archive News” | Templat **Theme Builder** (Header 792, Footer 799, Single 809, Archive 772) adalah fitur **Pro** ⇒ tidak aktif | **Belum** — padanan gratis tidak setara; **butuh keputusan** |
| 5. Halaman lain (Sejarah, Visi & Misi, Prestasi, Fasilitas, Pendaftaran, Kalender, Jurusan, Struktur) | Konten & layout tampil, warna belum kit | Kit tidak aktif | Kini memakai CSS kit: **post-897/899/901/903/969/971/984.css** terbentuk ✓ |
| 6. Responsif | — | — | Pada lebar panel (~600 px) tampil tombol menu ☰ dan hero bertumpuk (perilaku responsif normal Astra/Elementor) ✓ · **uji lebar desktop penuh belum bisa** karena `browser_exec` terkunci profil Chrome (§5) |

**Tidak dilakukan:** memasang Elementor Pro (tanpa lisensi) · menyalin kode PHP/tema/plugin lama · menulis CSS kustom · mengubah konten inti · mengubah desain secara spekulatif tanpa bukti.

---

## 3. Berkas CSS/template & metadata yang diubah

**Opsi (11, semuanya dapat dikembalikan via checkpoint-04):**
`elementor_active_kit=433` · `site_logo=1440` · `blogdescription="Merajut Mimpi Penuh Cinta"` · `astra-settings` (impor; `theme-auto-version` dipertahankan) · `theme_mods_astra` (impor + remap lokasi menu) · `widget_block` · `sidebars_widgets` · `elementor_disable_color_schemes=yes` · `elementor_disable_typography_schemes=yes` · `elementor_css_print_method=external` · `elementor_experiment-container=active`

**Berkas:**
- **30 berkas media baru** di `wp-content/uploads` (hash terverifikasi; daftar di `_5d-media-copy-log.tsv`).
- **13 berkas CSS turunan** Elementor dibangkitkan ulang di `uploads/elementor/css/` (`base-desktop/mobile.css`, `post-433.css`, `post-10/826/895/897/899/901/903/969/971/984.css`) — **turunan**, bukan sumber; aman dibuang dan diregenerasi.
- **Tidak ada** perubahan tema/plugin/anak-tema, tidak ada file PHP, tidak ada CSS kustom yang saya tulis.
- Snapshot nilai lama: `recovery-work\staging\backup-5d\opt-*.txt`.

---

## 4. Bukti sebelum / sesudah

| Item | Berkas |
|---|---|
| **SEBELUM** (halaman depan, sebelum 5D) | `recovery-work\staging\screenshots-5d\SEBELUM-5d-halaman-depan.png` (369.070 B) |
| **SESUDAH** halaman depan | Tangkapan jendela Hermes + panel pratinjau (terlihat di percakapan): **logo sekolah tampil**, hero dengan semboyan “Cerdas Dalam Berpikir / Jujur Dalam Bertindak”, tombol **Pendaftaran** berwarna kit + tombol **Profil Sekolah** outline, eyebrow “WELCOME”, tipografi kit |
| Bukti teks sesudah | `SESUDAH-home.html` (190.283 B) · `SESUDAH-kontak.html` (134.495 B) · `SESUDAH-sejarah.html` (124.136 B) |

Perbedaan pokok: **logo**, **warna & tipografi kit aktif**, **gaya tombol**, **tagline**, **widget footer**, dan latar halaman — semuanya berubah dari tampilan bawaan Astra menjadi tampilan kit Skoola.

---

## 5. Hasil pengujian, masalah tersisa, rollback

**V-01 … V-11 (dijalankan ulang setelah perbaikan): 11 LULUS, 0 GAGAL, 0 SEBAGIAN**
Termasuk: V-01 jumlah objek per tipe · V-02 23 halaman/berita · V-03 122 lampiran & berkasnya · V-04 thumbnail · V-05 menu 19 item/2 menu/19 relasi · V-06 0 URL produksi (3 baris alamat email dipertahankan) · **V-07 JSON Elementor valid; kit 158 elemen** · V-08 12 halaman HTTP 200 · V-09 IoC 0 · V-10 HTML bebas aset produksi · **V-11 270 rujukan media, 0 gagal**.

**V-12 (tambahan):** seluruh aset `.css`/`.js` pada 5 halaman utama → **HTTP 200** (tidak ada aset gagal).

**Error PHP:** 1 peringatan (bukan fatal) — `PHP Warning: Undefined array key "body_background_image_tablet"` di `elementor/includes/base/controls-stack.php:1499` saat memproses pengaturan kit lama yang tidak memiliki varian tablet. Halaman tetap 200 dan tampilan normal. Opsi: menambahkan kunci kosong pada `_elementor_page_settings` (mengubah data hasil pemulihan ⇒ butuh persetujuan).
**Error JavaScript:** konsol browser **tidak dapat diperiksa** karena `browser_exec` terkunci (§6); diperiksa sebagai gantinya lewat pemuatan seluruh aset (semua 200) dan render halaman yang benar. **Keterbatasan ini jujur saya sampaikan.**

**Masalah yang masih tersisa (butuh keputusan Anda):**
1. `2025/10/logo_SMK.png` (2 rujukan di pengaturan kit + templat Header/Footer) — berkasnya tidak ada di mana pun.
2. Widget **Pro** yang belum setara: `form` (Kontak), `posts`/`testimonial-carousel` (Home), dan 4 templat Theme Builder (Header, Footer, Single News, Archive News).
3. Peringatan PHP `body_background_image_tablet` (kosmetik).
4. Uji lebar desktop penuh & uji interaksi lanjutan (batas tool browser).
5. 120 salinan `-1` di `uploads` (sengaja **tidak** dihapus sesuai instruksi).

**Rollback:** `mysql -u root smk10_staging_kebumen < recovery-work\staging\checkpoints\checkpoint-04-pre-5d.sql` (mengembalikan seluruh perubahan 5D: 30 berkas media tetap ada di disk dan tidak mengganggu; untuk menghapusnya perlu tindakan terpisah dengan persetujuan).

**Batasan yang dipatuhi:** produksi, tiga situs lain, `goly1264_ams`, dump sumber, salinan `-1`, dan arsip `cgi-bin.zip` **tidak disentuh**; tidak ada kode dari backup yang dijalankan; tidak ada plugin/MCP dipasang; tidak ada kredensial diminta.
