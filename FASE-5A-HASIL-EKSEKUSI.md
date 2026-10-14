# FASE 5A — HASIL EKSEKUSI (Staging WordPress di XAMPP)

- **Tanggal:** 2026-10-14 · **Pasangan:** `FASE-5A-PERSIAPAN-STAGING.md` (rencana + verifikasi awal)
- **Persetujuan diterima:** AP-1 penuh · AP-2/AP-3 unduh resmi (id_ID + wp-cli) · **AP-4 pilot 1 DB (Kebumen)** · **AP-5 pilot Kebumen saja**
- **Integritas bukti:** 4 dump `D:\backup-database-smk10november\` — SHA-256 **cocok** Fase 4C.1, **0 berkas berubah** (dicek di akhir fase)

> Laporan ini memisahkan **(A) berhasil**, **(B) menunggu persetujuan**, **(C) kendala** — setiap pernyataan disertai bukti perintah.

---

## A. Tindakan yang BERHASIL dilakukan

### A.1 AP-1 — Ekstensi PHP aktif + Apache di-restart ✅

| Item | Bukti |
|---|---|
| Cadangan sebelum perubahan | `C:\xampp\php\php.ini.bak-2026-10-14` — SHA-256 `6767e6a3…c72a` (identik dengan asli) |
| Baris diubah (3, semuanya unik) | **931** `;extension=gd`→`extension=gd` · **934** `;extension=intl`→`extension=intl` · **962** `;extension=zip`→`extension=zip` |
| php.ini sesudah | SHA-256 `d09c87fe…bf86`, 76.831 → 76.825 B (6 byte = 3×`;`) |
| PHP CLI | `8.2.12 | gd=ya | intl=ya | zip=ya`, tanpa peringatan startup |
| **SAPI web (Apache)** | `SAPI=apache2handler PHP=8.2.12`, `gd=ya | intl=ya | zip=ya | mysqli=ya | curl=ya | mbstring=ya | exif=ya | fileinfo=ya | openssl=ya` (probe sementara `_phpcheck.php` — **sudah dihapus**) |
| Batas unggah | `upload_max_filesize=512M`, `post_max_size=600M`, `memory_limit=512M` |
| Restart Apache | semua proses lama dihentikan (terverifikasi 0) → dinyalakan ulang terpisah; `http://localhost/` → **302**; PID 3152 (induk) + 11240 (:80) |

**Koreksi proses (jujur):** percobaan pertama `taskkill //F //IM` **gagal** (MSYS tidak menerjemahkan `//F`) dan sempat membuat proses httpd tambahan yang tak berguna. Lalu dihentikan bersih via PowerShell `Stop-Process -Force` (0 proses terverifikasi) dan dinyalakan ulang; hasil diverifikasi. **Tidak ada server yang ditinggal mati.**

### A.2 AP-2 / AP-3 — Unduhan resmi ✅

| Berkas | Ukuran | SHA-256 | Jenis |
|---|---|---|---|
| `recovery-work\staging\wordpress-id_ID.zip` | 38.837.302 B | `83c3c37f70f4fb649e49178c86723b88f3688adbab9aad025d7f70caf562b7f5` | Zip, root `wordpress/`, 4.644 entri |
| `recovery-work\staging\wp-cli.phar` | 7.142.777 B | `ce34ddd838f7351d6759068d09793f26755463b4a4610a5a5c0a97b68220d85c` | PHP phar, **ber-tanda tangan SHA1** ✓ |

Sumber resmi: `id.wordpress.org/latest-id_ID.zip` + kanal build resmi wp-cli. **Berkas WordPress dari backup terkompromi tidak dipakai.**

### A.3 AP-4 (pilot) — Database + akun DB ✅

| Item | Hasil |
|---|---|
| Database | `smk10_staging_kebumen` (utf8mb4/utf8mb4_unicode_ci) |
| Akun | `smk10_staging`@`localhost` & @`127.0.0.1` |
| Hak (dari `SHOW GRANTS`) | `SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES` **hanya pada `smk10_staging_%`** |
| Uji login akun baru | rc=0; hanya melihat `information_schema`, `smk10_staging_kebumen`, `test` ⇒ **tidak bisa mengakses DB lain** |
| Sandi | dibuat lokal → `…\recovery-work\staging\.dbpass` (**tidak dicetak**) |
| **`goly1264_ams`** | **tidak disentuh** — 29 tabel, sama sebelum & sesudah |

### A.4 AP-5 + S-5 (pilot) — Instalasi WordPress bersih ✅

| Item | Hasil |
|---|---|
| Verifikasi arsip | root `wordpress/` OK · **versi 7.0.4** · syarat PHP ≥7.4 & MySQL ≥5.5.5 **terpenuhi** |
| Ekstraksi | `C:\xampp\htdocs\staging\kebumen` — **4.172 berkas** (layout diperbaiki: arsip berakar `wordpress/`) |
| Berkas inti | `index.php`, `wp-settings.php`, `wp-load.php`, `wp-login.php`, `wp-admin/index.php`, `wp-includes/version.php`, `wp-content/{themes,plugins}` → **semua ADA** |
| `wp-config.php` | baru (1.554 B): prefiks **`wp_`**, kredensial dari `.dbpass`, **8 SALT baru lokal**, `DISALLOW_FILE_EDIT=true`, `WP_ENVIRONMENT_TYPE=staging` |
| `wp core install` | **“Success: WordPress installed successfully.”** (dengan `--skip-email`) |
| **Skema DB** | **12 tabel resmi `wp_`**: commentmeta, comments, links, options, postmeta, posts, term_relationships, term_taxonomy, termmeta, terms, usermeta, users ⇒ **skema WordPress resmi, bukan salinan 138 tabel lama** |
| Admin | `kebumen_admin` (ID 1, administrator, `staging@localhost.local`) — akun **baru** |
| siteurl/home | `http://localhost/staging/kebumen` ✔ |
| Uji HTTP | `/` → **200** · `/wp-login.php` → **200** · `/wp-admin/` → **302** (normal) |
| Tema & plugin | `twentytwentyfive` aktif; plugin hanya bawaan (`akismet`, `hello` — **nonaktif**) ⇒ **pristine** |

**Akses pilot:** `http://localhost/staging/kebumen/wp-login.php` — pengguna `kebumen_admin`, sandi di `…\recovery-work\staging\.adminpass` (tidak ditampilkan).
**Path kredensial:** `.dbpass` (sandi DB) · `.adminpass` (sandi admin) · `.my_staging.cnf` (opsi klien MySQL).

---

## B. ⚠️ Insiden keamanan pada fase ini — dilaporkan sendiri, sudah ditutup

**Kejadian:** pada percobaan pertama `wp core install`, pembuat sandi gagal karena **bug path MSYS→Windows** (`FileNotFoundError: '/c/Users/…/.adminpass'`). Akibatnya `--admin_password="$(cat …)"` menjadi kosong dan **wp-cli membangkitkan sandi sendiri lalu mencetaknya** (`Admin password: …`) ⇒ nilai itu **terekspos di transkrip percakapan**.

**Dampak:** terbatas pada **akun staging lokal** (localhost; bukan produksi, bukan akun situs sekolah). Tidak ada sistem lain yang memakai sandi itu.

**Penanganan:** sandi **dirotasi seketika** — sandi baru 28 karakter dibuat lokal → `.adminpass` → `wp user update 1 --user_pass=…` ⇒ **“Success: Updated user 1.”** Sandi lama **batal**.

**Perbaikan prosedur:** berkas rahasia dibuat memakai **path native Windows** (`C:/…`), dan sandi tidak lagi melewati teks perintah yang terlihat; verifikasi dilakukan tanpa mencetak nilai.

---

## C. Menunggu persetujuan (belum dijalankan)

| Kode | Tindakan | Catatan |
|---|---|---|
| **B-1** | Buat 3 DB lagi: `smk10_staging_{tambun,cibitung,sragen}` | memperluas pilot → 4 situs |
| **B-2** | Buat 3 instalasi WordPress bersih + `wp-config.php` masing-masing | idem |
| **B-3** | Pasang plugin/tema dari **repo resmi**: `elementor`, tema `hello-elementor` (Kebumen juga `astra`); `elementor-pro` **hanya bila lisensi tersedia** | unduhan tambahan + mengubah isi instalasi |
| **B-4** | Tulis konverter **dump→WXR** (offline) untuk `kbm_`, impor ke pilot, jalankan **V-01…V-10** | impor konten ke staging; dump/produksi tidak disentuh |
| **B-5** | (Opsional) cabut hak akun staging atas DB bawaan `test` | kebersihan hak |

---

## D. Kendala yang masih ada

| # | Kendala | Dampak | Jalan keluar |
|---|---|---|---|
| **K-1** | **Selisih versi:** paket resmi terbaru = **7.0.4**, sedangkan Fase 3A mencatat Tambun & Sragen pada **7.1.3** (Cibitung 6.9.1, Kebumen 6.9.9) | staging memakai inti lebih lama dari yang dilaporkan produksi | Tetapkan target: 7.0.4 (stabil resmi) atau tunggu 7.1.x. **Tidak menghalangi impor konten** |
| **K-2** | **Kelengkapan `posts`** (76–87 % postmeta yatim, Fase 4D §2.4) | bisa berarti sebagian konten tak ada di dump | Bandingkan dengan DB hosting (butuh izin) atau terima konten yang ada |
| **K-3** | Sebagian media mungkin hilang (karantina AV) | lampiran tanpa berkas | Verifikasi V-04 saat impor |
| **K-4** | `php` di PATH = `C:\php` (8.4.22, **tanpa** gd/zip) | wp-cli bisa memakai PHP yang salah | **Selalu** pakai `C:\xampp\php\php.exe` (sudah diterapkan) |
| **K-5** | DB bawaan `test` terlihat akun staging | bukan data kasus | opsional B-5 |
| **K-6** | **Nama berkas ini berbeda dari yang diminta** | permintaan awal menyebut `FASE-5A-PERSIAPAN-STAGING.md`; berkas itu tetap ada sebagai rencana + ringkasan status terbaru, hasil eksekusi dipisah ke berkas ini agar rencana tidak hilang | Saka dapat meminta penggabungan bila diinginkan |

---

## G. B-3 & B-4 — hasil lanjutan (disetujui 2026-10-14 malam)

### G.1 Keputusan Saka atas usulan lanjutan
- **B-1/B-2 (3 DB + 3 instalasi lain): DITOLAK untuk sekarang** — Saka: *"Pastikan satu situs berhasil dipulihkan sebelum memperluas staging."* ⇒ tidak ada DB/instalasi tambahan yang dibuat.
- **B-3 (plugin/tema resmi): DISETUJUI** ⇒ dijalankan pada pilot Kebumen.
- **B-4 (konverter dump→WXR): DISETUJUI SEBAGIAN** — *"hanya tulis konverternya dulu (belum impor), lalu saya tinjau hasilnya dahulu."* ⇒ konverter ditulis; **impor belum dilakukan**.

### G.2 B-3 — plugin & tema dipasang dari repo resmi ✅

| Item | Hasil (terverifikasi) |
|---|---|
| Tema **aktif di situs lama** (dari dump, bukan dugaan) | **`astra`** (`template`/`stylesheet` = `astra`, `current_theme` = `Astra`); pembanding: Tambun memakai `hello-elementor` |
| Plugin terpasang | **`elementor` 4.3.5 — aktif** (repo wordpress.org) |
| Tema terpasang | **`astra` 4.14.0 — aktif**; `template`/`stylesheet` di DB staging kini `astra` ✔ (sama dengan situs lama) |
| Uji HTTP | halaman depan **200**, `/wp-admin/` **302** |
| Plugin lain | `akismet` (nonaktif), `hello` (nonaktif) |

**Catatan penting (kendala baru):** situs lama juga memakai **Elementor Pro 3.30.1** (lisensi aktif s/d 01.01.2030). **Elementor Pro tidak tersedia di repo publik** (produk komersial). Tanpa Pro, widget Pro di `_elementor_data` tidak akan dirender. Keputusan diperlukan: (a) pakai akun/lisensi Elementor sekolah untuk memasang Pro, atau (b) terima tampilan sebagian (widget Pro kosong).

**Catatan versi:** Elementor repo sekarang **4.3.5** sedangkan situs lama **3.30.4**; tema **Astra 4.14.0** sedangkan lama tidak terekam versinya di artefak.

### G.3 B-4 — konverter dump→WXR (mode kering) ✅ belum impor

Berkas: `recovery-work\tools\convert_dump_to_wxr.py` — membaca dump **langsung (tanpa MySQL, tanpa impor)**, read-only terhadap sumber.

Hasil mode kering untuk `kbm_` (Kebumen):

| Ukuran | Nilai |
|---|---|
| SHA-256 dump yang dipakai | `d91e42bfa1e480995f8c0ea2af869a6d028f85ab05fffb9fd0ca85f02fa1eedc` |
| **post DIPILIH** | **174** (halaman, berita, lampiran, item menu, templat Elementor, `wp_navigation`, `wp_global_styles`, `wpdmpro`) |
| post DIKELUARKAN | `revision` 214 · `oembed_cache` 172 · `customize_changeset` 78 · `auto-draft` 2 · `trash` 1 · `envato_tk_import` 1 = **468** |
| postmeta siap ditulis | **357** (kunci allowlist: `_elementor_data`, `_elementor_edit_mode`, `_elementor_version`, `_elementor_template_type`, `_wp_page_template`, `_thumbnail_id`, `_wp_attached_file`, `_elementor_inline_svg`, `_menu_item_*`) |
| postmeta **DIBLOKIR karena IoC** | **0** (penjaga `wp2shell`/`sso_token`/`_codehidden`/`shellcode`/`wp_edvl`/`wp_zqyq`/`uvisyfyj`/`wp-wordfence` aktif) |
| postmeta DITUNDA (PHP-serialized → langkah terpisah) | **199**: `_wp_attachment_metadata` 121 · `_elementor_page_assets` 24 · `_menu_item_classes` 19 · `_elementor_css` 14 · `_elementor_page_settings` 14 · `_elementor_conditions` 4 · `__wpdm_files`/`__wpdm_access`/`_elementor_screenshot` 1 masing-masing |
| Berkas keluaran | `kebumen\export\kebumen-staging-pilot.wxr.dryrun.txt` (rencana + daftar 174 item) · `kebumen\export\kebumen-media-manifest.tsv` (174 baris lampiran) |

**Keputusan desain yang terdokumentasi (penting):**
1. **Nilai PHP-serialized TIDAK ditulis ke WXR** — karena `maybe_serialize()` WordPress akan **men-serialisasi ulang** string yang sudah terserialisasi (double-serialization) sehingga data rusak saat impor. Nilai ini dikirim pada **langkah kedua terpisah** (mis. `wp post meta update` atau skrip PHP terkontrol).
2. **Nilai JSON (mis. `_elementor_data`) TETAP ikut** — itu data desain, bukan serialisasi PHP. (Bug awal saya menunda semua nilai >1024 B sehingga `_elementor_data` ikut tertunda; **sudah diperbaiki dan diverifikasi**.)
3. **Penulis (author) tidak diekspor sebagai akun.** Semua post dipetakan ke admin staging; blok `<wp:author>` tidak dibuat ⇒ ** tidak ada akun penyerang yang bisa masuk**. Atribusi penulis dipulihkan manual setelah verifikasi.
4. Mode `--emit` (penulisan berkas WXR) **belum diaktifkan** — menunggu tinjauan Saka atas rencana ini.

### G.4 Insiden kecil: sisa path salah `C:\c` (MENUNGGU KEPUTUSAN)

Karena saya lagi-lagi mengirim path bergaya MSYS (`/c/Users/…`) ke Python Windows, satu jalannya menulis 2 berkas ke lokasi salah: `C:\c\Users\sakha\forensik-smk10november\recovery-work\kebumen\export\` (**2 berkas, 28.196 B**, hanya keluaran mode kering konverter).

Saya **mencoba menghapusnya** tetapi perintahnya **diblokir sistem karena mengandung `rm -rf`** dan tidak disetujui. Sesuai aturan, saya **tidak mengulang/menyiasati** — penghapusan **menunggu keputusan Saka**. Isinya tidak berbahaya (laporan teks) dan tidak mengganggu; cukup dihapus manual atau beri izin.

### G.5 Kendala baru
| # | Kendala | Dampak |
|---|---|---|
| **K-7** | **Elementor Pro tidak bisa dipasang dari repo publik** (komersial, lisensi sekolah) | widget Pro tidak dirender sampai lisensi tersedia |
| **K-8** | 199 nilai postmeta berserialisasi harus dimigrasikan pada langkah kedua | perlu skrip terpisah sebelum tampilan 100% setara |
| **K-9** | `C:\c` sisa path saya | Saka menghapus manual (keputusan 2026-10-14) |

### G.6 B-4 lanjutan — berkas WXR Dihasilkan & Divalidasi ✅ (IMPOR TETAP BELUM)

Keputusan Saka: **"hanya hasilkan berkas WXR dulu (--emit), jangan impor."** ⇒ mode `--emit` diimplementasikan; impor **tidak** dijalankan.

Keluaran pada `recovery-work\kebumen\export\`:

| Berkas | Ukuran | Isi |
|---|---|---|
| `kebumen-staging-pilot.wxr` | **1.599.301 B** (5.417 baris) | WXR 1.2, siap ditinjau |
| `kebumen-staging-pilot.wxr.dryrun.txt` | 21.033 B | rencana + daftar 174 item |
| `kebumen-media-manifest.tsv` | 7.385 B | 174 lampiran → path uploads + ada/tidak di salinan bukti |

Hasil validasi `tools\validate_wxr.py` (tanpa impor):

| Pemeriksaan | Hasil |
|---|---|
| XML well-formed | **YA** (`xml.etree` berhasil mem-parse) |
| Jumlah item | **174** — cocok persis dengan rencana |
| Sebaran `wp:post_type` | attachment 122 · nav_menu_item 19 · page 11 · post 10 · elementor_library 8 · wp_global_styles 2 · wp_navigation 1 · wpdmpro 1 |
| Sebaran `wp:status` | inherit 122 · publish 52 |
| `wp:postmeta` / term | **357** / 39 |
| Penulis (`dc:creator`) unik | **hanya `kebumen_admin`** ⇒ tidak ada akun penyerang ikut |
| **String IoC di dalam berkas** | **0 (bersih)** — `wp2shell`, `sso_token`, `_codehidden`, `shellcode`, `fm_key`, `wp_edvl`, `wp_zqyq`, `uvisyfyj`, `sso-loader` |
| `_elementor_data` (data desain) | **ikut, 18×** |
| `_wp_attached_file` / `_menu_item_url` | 122× / 19× |

**Perbaikan teknis yang dilakukan pada konverter (ditemukan lewat validasi):**
1. `pubDate` item semula memakai format MySQL mentah → diubah ke **RFC 2822** (`rfc()`), karena parser importer (SimplePie) lebih menyukai format itu.
2. Aturan penundaan salah menunda `_elementor_data` (data desain) → diperbaiki agar hanya nilai **PHP-serialized** yang ditunda.
3. `_elementor_pro_version` salah digolongkan serialized → dikeluarkan dari daftar tunda (ditunda 217 → **199**).
4. Sisa kode mati (blok XML yang seolah menulis berkas) → diberi komentar status yang jujur.

**Yang BELUM dilakukan (dengan sengaja):** impor WXR ke pilot, unggah 122 berkas media dari salinan bukti, dan 199 nilai postmeta berserialisasi (K-8).

### G.7 K-7 — analisis pemakaian widget Elementor (PRO vs FREE) ✅ SELESAI

Skrip `tools\pro_widgets.py` menyisir `_elementor_data` (JSON) keempat dump, menghitung `widgetType`, dan menggolongkan **FREE / PRO / belum-pasti** (yang belum pasti dilaporkan apa adanya).

**Dua bug saya temukan & perbaiki dalam proses ini** (dicatat demi kejujuran):
1. Regex pertama tidak cocok karena JSON di dalam dump **ter-escape** (`\"widgetType\":\"heading\"`).
2. Lebih penting: skrip **tidak punya fallback berlapis**, sehingga nilai `_elementor_data` besar gagal di jalur cepat dan **dilewati diam-diam** → hasil pertama "0 widget" (menyesatkan). Setelah fallback dipasang + penghitung baris-tak-terparse (kini **0**), angka nyata muncul.

**A. Seluruh isi dump (termasuk 1.169 baris `_elementor_data` YATIM yang tak akan dimigrasikan):**

| Situs | nilai `_elementor_data` | widget FREE | widget PRO | belum pasti |
|---|---|---|---|---|
| Tambun | 1.034 | 50.652 | 2.735 | 260 |
| Cibitung | 1.098 | 52.908 | 2.779 | 260 |
| **Kebumen** | **1.187** | **55.756** | **2.958** | **270** |
| Sragen | 1.041 | 51.147 | 2.769 | 260 |

**B. Hanya post yang BENAR-BENAR dimigrasikan (angka yang menentukan) — Kebumen:**
Dari 174 post yang diekspor, hanya **18** yang membawa `_elementor_data`:

| Ukuran | Nilai |
|---|---|
| widget **FREE** | **209** |
| widget **PRO** | **15** |
| belum pasti (semuanya Pro) | 3 |
| ⇒ proporsi Pro | **≈ 8 %** |

PRO yang dipakai: `posts` 4 · `testimonial-carousel` 3 · `nav-menu` 2 · `theme-archive-title` 1 · `theme-post-featured-image` 1 · `theme-post-content` 1 · `share-buttons` 1 · `search-form` 1 · `form` 1. Belum pasti: `archive-posts` 2, `post-info` 1 (keduanya Pro).
Widget gratis terbanyak: `heading` 64 · `image-box` 34 · `icon-box` 29 · `image` 24 · `text-editor` 15 · `button` 15.

**Kesimpulan:** ~92 % konten memakai widget gratis. Yang hilang tanpa Pro terbagi dua:
1. **Theme Builder** (`theme-post-*`, `theme-archive-title`, `archive-posts`, `post-info`) → templat Single/Archive (8 item `elementor_library`) tidak aktif; halaman & berita tetap utuh, hanya tata letaknya kembali ke bawaan tema.
2. **Widget Pro di dalam halaman** (`posts`, `testimonial-carousel`, `nav-menu`, `form`, `search-form`, `share-buttons`) → 15 instance.

⇒ **Rekomendasi: lanjutkan pilot TANPA Pro.** Tujuan pilot adalah membuktikan metode (konten + 122 media + menu + desain dasar), dan itu tidak terblokir; Pro dapat ditambahkan kapan saja bila lisensi sekolah tersedia (plugin Pro bisa dipasang menyusul tanpa mengulang impor konten).

**C. DAFTAR RINCI — 18 post yang membawa data Elementor (diminta Saka)**
Skrip: `tools\pro_detail.py` (juga mendukung `--ids`/daftar id untuk situs lain).

| id | tipe | template_type | PRO | judul | widget PRO yang dipakai |
|---:|---|---|---:|---|---|
| 809 | elementor_library | single-post | **5** | **Single News** | theme-post-featured-image, theme-post-content, share-buttons, search-form, posts |
| 10 | page | wp-page | **4** | **Home** | posts ×2, testimonial-carousel ×2 |
| 1155 | elementor_library | page | **2** | Home (templat) | posts, testimonial-carousel |
| 826 | page | wp-page | **1** | **Kontak** | form |
| 799 | elementor_library | footer | **1** | Footer | nav-menu |
| 792 | elementor_library | header | **1** | Header | nav-menu |
| 772 | elementor_library | archive | **1** | Archive News | theme-archive-title |
| 984 | page | wp-page | 0 | Pendaftaran | — |
| 971 | page | wp-page | 0 | Prestasi | — |
| 969 | page | wp-page | 0 | Fasilitas Sekolah | — |
| 903 | page | wp-page | 0 | Kalender Akademik | — |
| 901 | page | wp-page | 0 | Jurusan/Konsentrasi Keahlian | — |
| 899 | page | wp-page | 0 | Struktur Organisasi | — |
| 897 | page | wp-page | 0 | Visi & Misi | — |
| 895 | page | wp-page | 0 | Sejarah | — |
| 433 | elementor_library | kit | 0 | Kit Styles: **Skoola – Template kit for university** | — |
| 17 | elementor_library | page | 0 | Coming Soon | — |
| 6 | elementor_library | kit | 0 | Default Kit | — |

**Bacaan atas daftar ini (penting untuk keputusan):**
1. **8 dari 11 halaman bersih total dari Pro** (Pendaftaran, Prestasi, Fasilitas, Kalender, Jurusan, Struktur, Visi & Misi, Sejarah) — isi & desainnya utuh tanpa Pro.
2. Hanya **3 halaman** berkepentingan Pro: **Home** (posts + testimonial-carousel), **Kontak** (form), dan templat Home duplikat.
3. **4 templat Theme Builder** (Header `nav-menu`, Footer `nav-menu`, Single News `theme-post-*`, Archive News `theme-archive-title`) memerlukan Pro. Tanpa Pro, keempatnya tidak aktif → header/footer/single/archive kembali ke tata letak bawaan **Astra** (isi tetap utuh, tampilan berubah).
4. Situs memakai **template kit pihak ketiga "Skoola"** (id 433) — warna/typografi global kit itu tersimpan di `_elementor_page_settings` yang **DITUNDA sebagai serialized (K-8)**; jadi langkah K-8 wajib dijalankan agar warna/font ikut, terlepas dari Pro.
5. Opini jujur: tanpa Pro, hasil pilot akan **terlihat berbeda** dari situs aslinya (header/footer/single/archive berganti ke Astra, 3 halaman punya widget kosong). Bila tujuan pilot adalah "membuktikan metode pemulihan data", ini tidak masalah; bila tujuannya "membuktikan tampilan setara", Pro diperlukan.

---

## E. Kepatuhan batasan

| Batasan | Status |
|---|---|
| Tidak mengimpor dump SQL lama | **Dipatuhi** — DB staging hanya berisi 12 tabel skema baru, **0 konten lama** |
| Tidak menyentuh hosting/produksi | **Dipatuhi** — jaringan hanya untuk unduhan resmi yang disetujui |
| Tidak memakai WordPress dari backup terkompromi | **Dipatuhi** |
| Tidak menjalankan kode/plugin/tema/payload mencurigakan | **Dipatuhi** — hanya inti resmi; plugin bawaan nonaktif |
| Tidak menyentuh DB di luar ruang lingkup | **Dipatuhi** — `goly1264_ams` hanya dihitung jumlah tabelnya |
| Tidak mengklaim sebelum berhasil | **Dipatuhi** — 3 DB/3 instalasi dinyatakan **belum** dibuat |
| Tidak memasang server alternatif | **Dipatuhi** — XAMPP yang ada |
| Tidak menampilkan sandi/rahasia | **Dipatuhi kecuali satu insiden yang dilaporkan & sudah dirotasi (B)** |

---

## F. Langkah berikutnya

1. **B-1/B-2** (izin): replikasi A.3–A.4 untuk Tambun, Cibitung, Sragen.
2. **B-3**: plugin/tema resmi pada pilot → uji tampilan.
3. **B-4**: konverter `dump→WXR` untuk Kebumen → impor → **V-01…V-10**.
4. Lulus validasi pilot ⇒ ulangi untuk tiga situs lain.
5. Uji akhir: **0 IoC** di target (`_codehidden`, `sso_token`, `wp2shell*`, `fm_key`, `wp_edvl`, `wp_zqyq`, `uvisyfyj`, `wp-wordfence`).

**Status:** lingkungan lokal **siap**; **pilot Kebumen berhasil dan berjalan**; tiga situs lain + impor konten menunggu izin; tidak ada kendala yang menghalangi langkah berikutnya.

**Artefak fase:** `recovery-work\tools\activate_ext.py`, `setup_db_kebumen.py`, `stage_kebumen.py`, `fix_layout_kebumen.py`, `staging\wordpress-id_ID.zip`, `staging\wp-cli.phar`, `staging\.dbpass`, `staging\.adminpass`, `staging\.my_staging.cnf`, `php.ini.bak-2026-10-14`.
