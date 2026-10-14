# FASE 4D — Rencana Migrasi & Persiapan Staging (Empat WordPress)

- **Sumber (dipertahankan, read-only):** `D:\backup-database-smk10november\` (4 dump; SHA-256 diverifikasi ulang, cocok dengan Fase 4C.1)
- **Folder kerja pemulihan:** `C:\Users\sakha\forensik-smk10november\recovery-work\`
- **Tanggal:** 2026-10-14
- **Sifat fase ini:** perencanaan + inventarisasi. **Tidak** ada impor SQL, perubahan database, penghapusan berkas, jaringan, atau sentuhan ke hosting produksi.

---

## 1. Status kesiapan lingkungan lokal (Tahap A)

### 1.1 Yang **sudah** tersedia — terverifikasi

| Komponen | Temuan nyata | Layak untuk staging? |
|---|---|---|
| **Web server** | Apache **2.4.58** (XAMPP) — **sedang berjalan**, PID 11508 di port 80 & 443 | ✅ ya |
| **Database server** | MariaDB **10.4.32** (XAMPP) — **sedang berjalan**, PID 21632 di port 3306 | ✅ ya |
| **PHP (XAMPP)** | **8.2.12**; ekstensi aktif: `mysqli`, `pdo_mysql`, `mbstring`, `curl`, `openssl`, `exif`, `fileinfo` | ⚠️ kurang `gd`, `zip`, `intl` |
| **PHP (mandiri)** | **8.4.22** (`C:\php\php.ini`); `mysqli`, `pdo_mysql`, `mbstring`, `curl`, `openssl`, `json` | ⚠️ kurang `gd`, `zip`, `exif`, `fileinfo`, `intl` |
| Klien DB | `C:\xampp\mysql\bin\mysql.exe`, `mysqldump.exe` | ✅ ya |
| Alat pendukung | `git`, `node`, `composer` tersedia | ✅ ya |
| Perangkat keras | Disk C: **107 GB bebas** (53 % terpakai), RAM **±11,9 GB** | ✅ cukup (dump 343 MB + media ±1,04 GB) |

**Temuan penting (dicek, bukan diasumsikan):**
- **`htdocs` XAMPP masih bersih** — hanya memuat dashboard XAMPP; belum ada situs lain yang bisa bentrok.
- **MariaDB lokal sudah memiliki 1 database lain: `goly1264_ams`** → **bukan milik kasus ini**. Rencana staging **tidak boleh menyentuhnya**; semua database staging akan diberi nama berawalan `smk10_staging_`.
- **Ekstensi `gd` dan `zip` belum aktif di kedua PHP.** WordPress memerlukan keduanya (thumbnail gambar; pemasangan/ekspor tidak akan berjalan benar tanpa `zip`).
- **`upload_max_filesize` C:\php hanya 2 MB** (XAMPP: 512 MB) → impor WXR akan gagal bila memakai PHP C:\php tanpa diubah.
- **`wp-cli` tidak terpasang**; Docker tidak terpasang.

### 1.2 Pilihan instalasi lokal — rekomendasi

**Rekomendasi (paling sederhana, memakai yang sudah ada):** gunakan **XAMPP** sebagai staging — Apache + MariaDB + PHP 8.2.12 sudah berjalan, hanya perlu (a) mengaktifkan `gd`+`zip`, (b) menyediakan paket WordPress resmi, (c) menyediakan `wp-cli`.

| Alternatif | Pertimbangan |
|---|---|
| **XAMPP yang sudah ada** | ✅ paling sedikit perubahan; sudah berjalan; htdocs kosong. **Dipilih.** |
| Laragon / LocalWP / Herd | Perlu pemasangan baru (unduhan + installer) → lebih banyak perubahan sistem |
| Docker | Tidak terpasang; paling berat untuk kebutuhan ini |
| PHP C:\php + MySQL terpisah | Perlu penyesuaian ekstensi & konfigurasi lebih banyak |

### 1.3 Langkah yang **memerlukan persetujuan Saka** (belum dijalankan)

| # | Tindakan | Mengapa perlu persetujuan |
|---|---|---|
| A-1 | Mengaktifkan `extension=gd` dan `extension=zip` di `C:\xampp\php\php.ini` (dan/atau `C:\php\php.ini`) | mengubah berkas konfigurasi sistem |
| A-2 | Menyediakan **paket WordPress resmi** (unduh dari wordpress.org) | butuh **jaringan**; juga menyelesaikan penghambat §8-1 |
| A-3 | Menyediakan **`wp-cli.phar`** | butuh **jaringan** |
| A-4 | Membuat **4 database staging + 1 user DB khusus** di MariaDB lokal | mengubah database server lokal (perlu kredensial root/DB) |
| A-5 | (Opsional) menaikkan `upload_max_filesize`/`post_max_size` bila memakai PHP `C:\php` | mengubah konfigurasi sistem |

**Kesimpulan Tahap A: lingkungan lokal TIDAK sepenuhnya siap, tetapi sudah ±80 % siap.** Tidak ada yang diinstal/diubah pada fase ini.

---

## 2. Matriks migrasi empat situs (Tahap B)

### 2.1 Konten inti per situs (angka dari parser, bukan perkiraan)

| | Tambun | Cibitung | Kebumen | Sragen |
|---|---|---|---|---|
| Halaman (`page`, publish) | **11** | **11** | **11** | **11** |
| Berita (`post`, publish) | 0 | 0 | **12** | 0 |
| `post` draf/auto-draft/trash | 1 auto-draft | 4 draf page | 2 auto-draft + 1 trash | 1 auto-draft |
| Templat Elementor (`elementor_library`) | 8 | 7 | 8 | 8 |
| Item menu (`nav_menu_item`) | 22 | 22 | 19 | 22 |
| `wp_navigation` | 1 | 1 | 1 | 1 |
| Lampiran (`attachment`) | 120 | 153 | 122 | 107 |
| Revisi (`revision`) | 101 | 153 | 214 | 55 |
| `customize_changeset` | 34 | 89 | 78 | 22 |
| `oembed_cache` | 45 | 149 | 172 | 29 |
| `request` (Elementor) | 34 | 89 | 0 | 22 |
| `custom-code` | 0 | 0 | 0 | **5 (semua KOSONG)** |
| `wpdmpro` | 0 | 0 | 1 | 0 |
| Total baris `posts` | 378 | 681 | 642 | 284 |

**Catatan penting:** **kesebelas halaman utama identik di keempat situs** — id dan slug sama (`10 /home`, `895 /sejarah`, `897 /visi-misi`, `899 /struktur-organisasi`, `901 /jurusan-konsentrasi-keahlian`, `903 /kalender-akademik`, `969 /fasilitas`, `971 /prestasi`, `778 /berita-artikel`, `826 /kontak`, `984 /pendaftaran`) ⇒ berasal dari templat yang sama (konsisten dengan provisi 2025-10-30 di Fase 4C.2).

**Urutan prioritas konten (V1 = wajib, V2 = bila waktu memungkinkan):**
- **V1:** 11 halaman tiap situs; menu navigasi; 12 berita Kebumen; 8 templat Elementor; media (lampiran) yang direferensikan halaman.
- **V2:** 2 `wp_global_styles` (Kebumen); 1 `wpdmpro` (Kebumen, hanya bila plugin Download Manager dipertahankan); `_oembed_*`/`oembed_cache` (dibuat ulang otomatis).

### 2.2 Metadata penentu desain (baris nyata per situs)

| meta_key | Tambun | Cibitung | Kebumen | Sragen | Keputusan |
|---|---|---|---|---|---|
| `_elementor_data` | 1.034* | 1.098* | 1.187* | 1.041* | **migrasikan hanya untuk post_id yang ada** (§2.4) |
| `_elementor_edit_mode` | 1.069* | 1.142* | 1.222* | 1.076* | migrasikan (allowlist) |
| `_elementor_version` | 1.064* | 1.133* | 1.217* | 1.071* | migrasikan |
| `_elementor_template_type` | 1.069* | 1.142* | 1.222* | 1.076* | migrasikan |
| `_elementor_page_settings` | 983* | 1.029* | 1.118* | 980* | migrasikan |
| `_wp_page_template` | 1.052* | 1.116* | 1.205* | 1.059* | migrasikan |
| `_elementor_css` | 0 | 32 | 17 | 0 | **jangan** (dibuat ulang) |
| `_elementor_element_cache` | 0 | 30 | 16 | 0 | **jangan** (cache) |
| `_elementor_conditions` / `_elementor_source` | — | — | 40 / 22 | 40 / 22 | jangan (khusus kit) |
| `_thumbnail_id` | 0 | 0 | 11 | 0 | migrasikan (Kebumen) |
| `_wp_attached_file` | **120** | **153** | **122** | **107** | migrasikan (cocok = jumlah attachment) |
| `_menu_item_*` | 56 | 111 | 19 | 44 | migrasikan kecuali yang menuju `wp2shell.*` |

\* Angka bertanda bintang adalah **baris termasuk metadata yatim** (§2.4) — bukan seluruhnya boleh dimigrasikan.

### 2.3 Akun

| Kelompok | Tambun | Cibitung | Kebumen | Sragen | Perlakuan |
|---|---|---|---|---|---|
| Akun sah kandidat (kelas "belum cukup bukti"/verifikasi) | 2 | 6 | 1 | 3 | **dibuat ulang** setelah verifikasi manual, bukan disalin |
| Akun indikasi kuat | 39 | 84 | 1 | 27 | **tidak dibuat ulang** |
| Total | 41 | 90 | 2 | 30 | — |

Metode: **jangan** memindahkan tabel `users`/`usermeta`. Buat akun baru di instalasi bersih dengan peran minimum (administrator hanya untuk yang benar-benar perlu), lalu **ganti seluruh SALT & kredensial**.

### 2.4 Integritas referensial — temuan yang mengubah rencana (uji `rel_check.py`)

| Situs | `posts` | `postmeta` | post_id unik | **baris postmeta YATIM** | post_id yatim |
|---|---|---|---|---|---|
| Tambun | 378 | 8.213 | 1.247 | **6.862 (83,6 %)** | 983 |
| Cibitung | 681 | 9.361 | 1.410 | **7.382 (78,9 %)** | 1.045 |
| Kebumen | 642 | 9.160 | 1.377 | **6.938 (75,7 %)** | 1.003 |
| Sragen | 284 | 8.167 | 1.234 | **7.118 (87,2 %)** | 1.023 |

- **Sebagian besar `postmeta` menunjuk `post_id` yang tidak ada di `posts`.** Didominasi metadata Elementor (`_elementor_data`, `_elementor_edit_mode`, `_elementor_version`, `_elementor_pro_version`, `_wp_page_template`).
- **Pola ini nyaris identik di keempat database** (`_elementor_edit_mode`: 950/970/1.003/1.003) ⇒ hampir pasti **residu proses impor templat/Elementor kit**, bukan kegiatan penyerang.
- **Pengecualian yang justru IoC:** post_id besar bergaya epoch (`1707223188`, `1765455876`, …) — 33/75/0/20 id — yaitu baris `_menu_item_*` yang menunjuk item menu **`x`/`proof`** Fase 4C.2. Kebumen **0** (konsisten: menu `wp2shell` di Kebumen hanya ada di changeset).
- **Konsekuensi wajib:** migrasi postmeta **hanya** untuk `post_id` yang ikut dimigrasikan. Metadata yatim **tidak dimigrasikan**, dan pertanyaan "apakah `posts` lengkap" dicatat sebagai **kendala §8-2** (lihat di bawah) karena memengaruhi kelengkapan konten.

---

## 3. Matriks sumber → target, kolom, filter, metode (Tahap C)

Target: **4 instalasi WordPress terpisah**, masing-masing database sendiri, **skema dibuat oleh instalasi resmi** (bukan menyalin skema lama). Prefiks target disarankan `wp_` (standar) — **bukan** `smk_/cbt_/kbm_/srg_`.

| Tabel sumber | Tujuan | Kolom yang dibutuhkan | Filter / pengecualian | Relasi yang dijaga | Metode | Validasi |
|---|---|---|---|---|---|---|
| `posts` | `wp_posts` | `ID, post_author, post_date, post_content, post_title, post_excerpt, post_status, post_name, post_parent, menu_order, post_type, post_mime_type, guid, post_password(‑)` | tipe: `page`, `post`, `attachment`, `nav_menu_item`, `elementor_library`, `wp_navigation`, (`wp_global_styles` hanya Kebumen). Status: `publish`/`draft`. **Buang** `revision`, `oembed_cache`, `customize_changeset`, `request`, `auto-draft`, `trash` | `post_parent` (halaman↔revisi dibuang), `attachment`↔`post_parent` | WXR (`wp:post_type`) atau `wp-cli` + PHP | jumlah per tipe/status sama; slug & judul sama; nol `revision` |
| `postmeta` | `wp_postmeta` | `post_id, meta_key, meta_value` | **allowlist** kunci Elementor + `_wp_attached_file`, `_thumbnail_id`, `_wp_page_template`, `_menu_item_*`; **hanya post_id yang ikut**; buang baris menuju `wp2shell.*` | post_id → posts yang dimigrasikan | WXR `<wp:postmeta>` | jumlah meta per halaman sama; `_elementor_data` identik (hash) untuk sampel |
| `terms`, `term_taxonomy`, `term_relationships` | sama | `term_id, name, slug, taxonomy, parent, description, count` + relasi | hanya taksonomi yang dipakai (`category`, `nav_menu`, `elementor_library_type`, `post_tag` bila ada) | term↔post | WXR (`wp:category`, `wp:tag`, `wp:term`) | jumlah istilah & relasi sama; menu utuh di editor |
| `termmeta` | `wp_termmeta` | `term_id, meta_key, meta_value` | hanya Kebumen (4 baris) | — | WXR/`wp term meta` | 4 baris |
| `users`/`usermeta` | **tidak disalin** | — | buat ulang akun hasil verifikasi | — | UI/WP-CLI | jumlah akun = daftar bersih; tidak ada `session_tokens` |
| `options` | `wp_options` | allowlist: `siteurl, home, blogname, blogdescription, admin_email(‑ganti), timezone_string, date_format, time_format, start_of_week, posts_per_page, show_on_front, page_on_front, page_for_posts, permalink_structure, template(=hello-elementor), stylesheet` + opsi Elementor/tema yang sah | **buang** `_codehidden`, `sso_token`, `wp2shell_*`, `fm_key`, `mnx_wp_auth`, `__wpdm_enc_key`, `mailserver_pass`, `active_plugins`, `cron`, `session_tokens`, semua `_transient*`, 9 nilai serialized INVALID | `page_on_front`/`page_for_posts` → ID baru | dibuat di instalasi bersih, lalu ditimpa sebagian | `siteurl`/`home` benar; `_codehidden`=0; `sso_token`=0 |
| `snippets` (Code Snippets) | `wp_snippets` (dibuat plugin) | — | **jangan impor**; tinjau & buat ulang manual bila perlu | — | manual | 0 snippet tak dikenal |
| `mailpoet_*`, `woocommerce_*`, `cartflows_*`, `frmt_*`, `wpf_*`, `jet_*`, `gla_*`, `actionscheduler_*`, `jetpack_sync_queue`, `mailpoet_scheduled_tasks` | tabel plugin | — | **jangan impor** (data pelanggan/sesi/antrean) | — | dibuat oleh plugin di target | 0 data pelanggan lama |
| tabel 0 baris (378 entri) | — | — | tidak ada yang dimigrasikan | — | — | — |

**Metode migrasi yang direkomendasikan (kompatibel WordPress):** **ekspor WXR** yang dihasilkan oleh konverter **offline** yang membaca dump SQL langsung (tanpa menjalankan MySQL, tanpa pernah memuat data penyerang ke mesin DB), lalu `wp import` di instalasi bersih — alternatif/validasinya: impor dump ke DB **sandbox terpisah** lalu `wp export`. **Keduanya belum dijalankan**; ekspor final belum dibuat sampai metode terbukti menjaga struktur (§7 item 7).

---

## 4. Daftar konten yang diprioritaskan (Tahap B/E)

**Berita asli hanya ada di Kebumen (12 publish + 1 trash)** — sisanya hanya halaman statis:

| Situs | Konten prioritas V1 |
|---|---|
| **Tambun** | 11 halaman (Home, Sejarah, Visi & Misi, Struktur Organisasi, Jurusan/Konsentrasi Keahlian, Kalender Akademik, Fasilitas Sekolah, Prestasi, Berita/Artikel, Kontak, Pendaftaran) + 1 `wp_navigation` + 8 templat Elementor + 120 lampiran |
| **Cibitung** | 11 halaman + `wp_navigation` + 7 templat Elementor + 153 lampiran |
| **Kebumen** | 11 halaman + **12 berita** (MPLS 2026; LAPORAN BOSP Tahap 1 2026; HUT RI Ke-81; Pelatihan Bahasa Jepang; Fashion Show; Pelantikan OSIS Karanggayam; Cek Kesehatan Gratis Puskesmas 1 Karanggayam; PSTS Ganjil 2026; GEBYAR; Upacara Bendera Senin) + `wp_navigation` + 8 templat + 122 lampiran |
| **Sragen** | 11 halaman + `wp_navigation` + 8 templat Elementor + 107 lampiran |

**Tidak diprioritaskan / dibuang:** `revision` (523 total), `oembed_cache` (395), `customize_changeset` (223), `request` (145), 4 halaman "Elementor #NNNN" (Cibitung, draf), 2 post berjudul "x" (Cibitung — embed bawaan, lihat §6), 5 `custom-code` Sragen (**isinya KOSONG**), 1 auto-draft/p­ost per situs, 1 trash (Kebumen — tinjau isinya sebelum dibuang).

---

## 5. Metode mempertahankan metadata & relasi WordPress

1. **WXR sebagai format utama.** Setiap dokumen diekspor sebagai `<item>` dengan `<wp:postmeta>` untuk kunci allowlist; taksonomi sebagai `<category>`/`<wp:term>`; menu sebagai item `nav_menu_item` + metas `_menu_item_*` (relasi menu-medan **wajib** ikut, karena WordPress membangun menu dari `_menu_item_*`, bukan dari kolom post).
2. **ID lama tidak dipertahankan** — WordPress menetapkan ID baru saat impor; relasi dipulihkan lewat **pemetaan ID** (`old_id → new_id`) yang dicatat oleh alat konverter dan diverifikasi setelah impor. Ini alasan `page_on_front`/`page_for_posts`/`_thumbnail_id`/`menu_item_object_id` harus **diterjemahkan** setelah impor.
3. **Elementor:** wajib ada (a) plugin Elementor (+ Elementor Pro bila situs memakainya: Cibitung/Kebumen aktif), (b) tema `hello-elementor` (semua situs memakainya) dan `astra` (Kebumen). Tanpa keduanya, `_elementor_data` tidak akan dirender. `_elementor_css`/`_elementor_element_cache` sengaja dibuang dan dibangun ulang.
4. **Media:** berkas berasal dari salinan bukti `D:\smk10november\<situs>\wp-content\uploads\` (**2.569 / 2.827 / 2.681 / 2.421 berkas: 242 MB / 307 MB / 298 MB / 193 MB**) — **bukan** dari database. Impor lampiran lewat `wp media import` dengan peta `_wp_attached_file`, lalu verifikasi keberadaan berkas per lampiran (sebagian berkas mungkin hilang/dikarantina AV — catatan Fase 2).
5. **Verifikasi serialisasi:** nilai yang akan ditulis ulang ke `wp_options` divalidasi lebih dulu memakai validator teks yang sama seperti Fase 4C.3 (tanpa `unserialize()` objek tak tepercaya).

---

## 6. Data yang dikecualikan (Tahap D) — larangan migrasi

**Tidak boleh dibawa tanpa pembersihan & verifikasi khusus:**

| # | Data | Alasan (bukti) |
|---|---|---|
| D-1 | **`_codehidden`** (Tambun, 10.831 B) | base64 → 8.120 B PHP terobfuskasi (`file_put_contents`, `curl_exec`, `exec`) — Fase 4C.3 I-01 |
| D-2 | `sso_token`; `wp2shell_*`; `fm_key`; `mnx_wp_auth`; `__mnx_versions`/`active_plugins` yang memuat plugin palsu | Fase 4C.1/4C.3 |
| D-3 | 151–159 akun administrator tak sah + `session_tokens`/`capabilities` terkait | Fase 4C.2 |
| D-4 | Plugin, tema, dan snippet tak tepercaya (`wp-wordfence-security`, `wp_edvl`, `wp_zqyq`, `uvisyfyj`, `wp-compat`, `sso-loader.php`, snippet non-contoh) | Fase 3A/4C.3 |
| D-5 | Item menu / changeset menuju `wp2shell.research` / `wp2shell.com` (+ baris `_menu_item_*` yatim terkait) | Fase 4C.2 §5.1, Fase 4D §2.4 |
| D-6 | Seluruh cache, transient, sesi, antrean lama (`_transient*`, `_site_transient*`, `_elementor_*cache*`, `woocommerce_sessions`, `actionscheduler_*`, `jetpack_sync_queue`, `mailpoet_scheduled_tasks`) | dibuat ulang |
| D-7 | **9 nilai serialized yang tidak lolos validasi** | Fase 4C.3 §3.1 |
| D-8 | **Kredensial & SALT lama** (termasuk `wp-config.php` lama secara utuh) | diganti seluruhnya |

**Tambahan dari fase ini (perlu verifikasi sebelum dipakai):**
- 2 post berjudul **"x"** (Cibitung, id 1669 & 1685): isinya embed bawaan WordPress ke halaman situs sendiri (tanpa host asing, tanpa kode) → **bukan IoC terbukti, tetapi tidak perlu dimigrasikan**; status "perlu verifikasi".
- 4 halaman **"Elementor #NNNN"** (Cibitung: 1252 publish, 1275/1281/1341 draf): isi berisi gambar domain sendiri; **nama otomatis** → perlu verifikasi, default **tidak dimigrasikan**.
- 5 `custom-code` (Sragen): **isi 0 B**, hanya setelan plugin (`_codes_*`) → aman, **tidak perlu** dimigrasikan.

---

## 7. Prosedur validasi hasil migrasi

**V-01 Kelengkapan:** jumlah `page`/`post`/`attachment`/`nav_menu_item` di target = daftar sumber yang lolos filter (per situs). Selisih harus 0.
**V-02 Kesetaraan isi:** untuk **semua** halaman + berita: hash `post_content` (setelah normalisasi line-ending) sama dengan sumber. Untuk sampel ≥10 dokumen: hash `_elementor_data` sama.
**V-03 Relasi:** setiap `_thumbnail_id`, `page_on_front`, `page_for_posts`, `menu_item_object_id` menunjuk ID **baru** yang valid; semua item menu muncul di Appearance → Menus.
**V-04 Media:** setiap lampiran punya berkas di `wp-content/uploads` target; jumlah berkas = jumlah lampiran yang dimigrasikan; tidak ada `_wp_attached_file` tanpa berkas.
**V-05 Akun & rahasia:** jumlah akun = daftar bersih; peran minimum; **0** `session_tokens`; seluruh SALT/kunci baru.
**V-06 Pemindaian IoC pada target (wajib, hasil harus 0):**
`wp db query` / ekspor teks lalu cari: `_codehidden`, `sso_token`, `wp2shell`, `wp2shell.research`, `wp2shell.com`, `fm_key`, `wp_edvl`, `wp_zqyq`, `uvisyfyj`, `wp-wordfence`, `sso-loader`, `shellcode`, `cupiUploader`.
**V-07 Konfigurasi:** `siteurl`/`home` = domain situs yang benar; `users_can_register=0`; `default_role=subscriber`.
**V-08 Fungsional:** beranda, satu halaman Elementor, satu berita, satu menu, satu gambar tampil benar; `wp-admin` dapat diakses akun sah.
**V-09 Integritas berkas:** pemindaian berkas target (tidak ada webshell/file-manager tak sah) + hash inti WordPress terhadap paket resmi.
**V-10 Perbandingan menyeluruh:** `wp db check` (MariaDB) bersih; tidak ada baris yatim baru (`postmeta` tanpa `posts`).

---

## 8. Kendala yang benar-benar menghalangi

| # | Kendala | Dampak | Cara menyelesaikan |
|---|---|---|---|
| **1** | **Paket WordPress resmi tidak tersedia lokal** (dicek Downloads/Desktop/Documents — tidak ada). Instalasi bersih **tidak boleh** memakai salinan di `D:\smk10november`/`cgi-bin.zip` karena kompromi | **Menghalangi** pembuatan staging & target | Unduh `wordpress-<versi>.zip` resmi (**butuh jaringan → persetujuan**) |
| **2** | **`posts` tampak tidak lengkap relatif `postmeta`** (76–87 % metayatim, nyaris identik antar situs) | Bila ada halaman/post yang hilang dari dump, migrasi akan **kehilangan konten** | Bandingkan dengan DB di hosting produksi **atau** dengan salinan berkas/HTML; **butuh persetujuan** (akses hosting/OTP) — atau terima bahwa konten sah = 11 halaman + berita yang ada |
| **3** | Ekstensi PHP `gd`/`zip` belum aktif | Impor/thumbnail gagal | Aktifkan di php.ini (**persetujuan**) |
| **4** | `wp-cli` tidak ada; `upload_max_filesize` C:\php = 2 MB | Impor manual/lambat | Sediakan `wp-cli.phar` (**jaringan/persetujuan**) atau pakai PHP XAMPP |
| **5** | Kredensial MariaDB lokal & 4 DB staging belum ada | Tidak bisa membuat DB staging | Saka memberi izin + kredensial root lokal (**persetujuan**) |
| **6** | Sebagian berkas media mungkin hilang (karantina AV — Fase 2) | Lampiran tanpa berkas | Verifikasi V-04; ambil ulang dari hosting bila perlu |
| **7** | Timestamp lintas-situs `_codehidden` vs berkas belum dikaitkan ke log akses | Urutan serangan tak dipastikan | Log akses hosting (di luar izin) — **tidak menghalangi migrasi** |
| **8** | Kebijakan privasi data pelanggan (`mailpoet_subscribers`, checkout) | Tak boleh dipindahkan sembarangan | Putuskan: buang atau ekspor terpisah dengan dasar kewenangan |

---

## 9. Langkah siap dikerjakan berikutnya (urutan operasional)

**Langkah terkecil untuk mulai membangun staging** (butuh persetujuan A-1…A-5):
1. **A-1**: aktifkan `gd`+`zip` di php.ini → restart Apache.
2. **A-2/A-3**: sediakan `wordpress-<versi>.zip` resmi + `wp-cli.phar` (unduhan).
3. **A-4**: buat 4 DB staging `smk10_staging_{tambun,cibitung,kebumen,sragen}` + 1 user DB khusus (tanpa menyentuh `goly1264_ams`).
4. Buat 4 instalasi WordPress bersih di `C:\xampp\htdocs\staging\<situs>\` (skema resmi; tanpa data lama).
5. **Buktikan metode pada SATU situs dulu (Kebumen — paling sedikit IoC, konten berita terbanyak)**: konverter dump→WXR untuk `kbm_` → `wp import` → jalankan V-01…V-10.
6. Bila bukti validasi lolos: baru buat ekspor final untuk keempat situs di `recovery-work\<situs>\export\`, lalu migrasi sisanya.
7. **Tidak** menyentuh produksi sampai staging keempat situs lolos V-06 dan V-09.

**Status pekerjaan:**
- ✅ **Selesai di fase ini:** inventarisasi lingkungan (nyata, terverifikasi); struktur `recovery-work` + `INDEX.md`; matriks migrasi 4 situs; daftar konten prioritas; daftar pengecualian; prosedur validasi; ukuran integritas referensial (yatim) & media; uji allowlist 7 record.
- 🔜 **Direncanakan, belum dijalankan:** semua langkah 1–7 di atas; **konverter dump→WXR belum ditulis** (menunggu metode disetujui); **tidak ada berkas ekspor dibuat**.
- 🚫 **Tidak dilakukan (sesuai aturan):** impor SQL, perubahan DB, penghapusan berkas, jaringan, sentuhan hosting produksi, menjalankan kode temuan.

---

## 10. Kepatuhan aturan fase ini

| Aturan | Status |
|---|---|
| Pertahankan seluruh dump SQL sumber | **Dipatuhi** — 4 berkas, SHA-256 cocok, 0 perubahan |
| Tidak menjalankan payload/kode temuan | **Dipatuhi** — `_codehidden` hanya didekode di memori pada fase sebelumnya; fase ini tidak menyentuhnya |
| Tidak mengakses jaringan/produksi | **Dipatuhi** |
| Tidak mengarang hasil | **Dipatuhi** — semua angka dari parser/artefak; kendala dinyatakan sebagai kendala |
| Bedakan selesai vs rencana | **Dipatuhi** — §9 |
| Tidak menambah audit forensik baru | **Dipatuhi** — pemeriksaan tambahan (yatim, daftar konten, 7 record allowlist) semuanya langsung untuk keputusan migrasi |
| Tidak membuat ekspor final sebelum metode terbukti | **Dipatuhi** — `export\` masih kosong |
| Tidak mengimpor/mengubah DB, tidak menghapus berkas | **Dipatuhi** |

---

## Kesimpulan status

**Lingkungan lokal: belum sepenuhnya siap, tetapi ±80 % siap** — Apache + MariaDB **sudah berjalan**, `htdocs` bersih, disk/RAM cukup; yang kurang: ekstensi PHP `gd`/`zip`, paket WordPress resmi, `wp-cli`, dan kredensial/DB staging (semua butuh persetujuan, tidak ada yang dijalankan).

**Sudah disiapkan:** `recovery-work\` (4 folder situs + `export/validasi/staging-notes`, `tools\`, `staging\`) + `INDEX.md`; matriks migrasi lengkap; daftar konten prioritas (termasuk 12 berita Kebumen); daftar larangan migrasi; prosedur validasi V-01…V-10; tiga kendala teknis yang harus diselesaikan lebih dulu.

**Langkah operasional paling kecil untuk mulai: setujui A-1 + A-2 + A-4** (aktifkan `gd`/`zip`, sediakan paket WordPress resmi, buat 4 DB staging) — setelah itu staging Kebumen dapat dibangun, dan metode migrasi dibuktikan pada satu situs sebelum menyentuh tiga situs lain.

**Laporan ini:** `C:\Users\sakha\forensik-smk10november\recovery-work\FASE-4D-RENCANA-MIGRASI.md`
**Artefak pendukung:** `recovery-work\INDEX.md`, `audit-backup\_4d-konten.tsv`, `_4d-konten-run.txt`, `_4d-keys.tsv`/`_4d-keys-run.txt`, `_4d-rel-run.txt`
