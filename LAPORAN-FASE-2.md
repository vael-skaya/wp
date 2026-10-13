# Laporan Forensik Fase 2 — Salinan Situs WordPress `D:\smk10november`

- **Sifat pekerjaan:** read-only / non-remediasi. Tidak ada file di `D:\smk10november` yang ditulis, dihapus, dipindah, diganti nama, atau diubah metadatanya.
- **Tidak dilakukan:** eksekusi file dari folder bukti, koneksi jaringan/hosting/DB/kredensial, pembacaan isi rahasia `wp-config.php`.
- **Waktu pemeriksaan:** 2026-10-13, mulai 12:42 WIB (05:42 UTC).
- **Direktori laporan (di luar bukti):** `C:\Users\sakha\forensik-smk10november\`
- **Artefak laporan:** `manifest/`, `sha256-priority.txt`, `sha256-manifest.txt`, `inventory.txt`, `extract/`, `tools/`

---

## 0. Koreksi terhadap pemeriksaan pendahuluan

| Klaim awal | Fakta terverifikasi |
|---|---|
| "`sso-loader.php` tidak ditemukan" | **SALAH** — pencarian awal terpotong `head -20`. File ada di `wp-content/mu-plugins/sso-loader.php` (root) dan `smk10novembersragen.sch.id/wp-content/mu-plugins/sso-loader.php`, **hash identik** `337fee21376d07b2…`. |
| "`wp-slgnup.gz` = kandidat" | Terkonfirmasi malware SEO-cloaking (lihat F-06). |
| "`wp-content.txt` entri mencurigakan" | **Bukan malware** — isinya output `tree` ("Folder PATH listing for volume Data") = artefak persiapan salinan (lihat P-03). |

---

## 1. Inventarisasi & manifest

| Metrik | Nilai |
|---|---|
| Total file | **38.906** |
| Total direktori | 5.064 |
| Symlink / junction | **0** |
| File root install | 23 |
| Instalasi terpisah | 4 (root + cibitung + kebumen + sragen) |

### Versi WordPress per instalasi (dari `wp-includes/version.php`)

| Instalasi | `$wp_version` | `$wp_db_version` |
|---|---|---|
| root `D:\smk10november` | **7.1.3** | 61833 |
| `smk10novembercibitung.sch.id` | **6.9.1** | 60717 |
| `smk10novemberkebumen.sch.id` | **6.9.9** | 60717 |
| `smk10novembersragen.sch.id` | **7.1.3** | 61833 |

### Manifest SHA-256

- `sha256-priority.txt` — 751 file kunci (file root tiap instalasi, mu-plugins, plugin suspek, `.htaccess`, file PHP/arsip non-standar).
- `sha256-manifest.txt` — manifest penuh, dibangun **inkremental per direktori** (`manifest/sha256-<dir>.txt`) agar tahan terhadap timeout.
- **Status: SELESAI & TERVERIFIKASI.** 38.906 entri, 0 error baca, dan **uji silang manifest ⇄ daftar file nyata: COCOK SEMPURNA** (0 entri hanya-di-manifest, 0 file terlewat).
- **Catatan koreksi mandiri:** direktori `cgi-bin/` ternyata **kosong**, sehingga `xargs` (tanpa `-r`) tetap menjalankan `sha256sum` tanpa argumen dan menghasilkan hash **stdin kosong**: `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855 *-`. Ini artefak pipeline saya, **bukan** file di folder bukti. Manifest sudah dinormalisasi; entri tersebut tidak dihitung sebagai file.
- `cgi-bin/` = direktori **kosong** (mtime 2026-01-05 17:50, mendahului jendela penyalinan).

**Catatan kinerja:** `du` dan hashing satu-pass sebelumnya macet (>23 menit tanpa hasil) — indikasi kuat Windows Defender melakukan scanning real-time pada setiap akses file.

---

## 2. Temuan TERKONFIRMASI

### F-01 — Defacement halaman utama (root)
- **Path:** `index.html` (9.598 B, mtime **2026-10-12 21:21:10**)
- **SHA-256:** `537f3e5665dd556650541c9128a0edd09e30ddcf9199cb80c78a622a68c82306`
- **Bukti:** `<title>REHAN23</title>`, `<meta name="description" content="hacked by rehan23">`, teks defacer "Medan-Newbie | CyberPunks | Kamtiez | And All Defacer Indonesia!!", iframe YouTube tersembunyi + YouTube IFrame API.
- **Risiko:** Tinggi (reputasi). **Keyakinan:** Tinggi.
- **Catatan:** ini file terbaru di root install — **9 jam setelah** jendela mtime penyalinan (11:34–12:21), sehingga mtime ini tidak dapat dijelaskan oleh proses copy. Perlu ditelusuri terpisah.

### F-02 — Webshell eksekusi perintah di plugin palsu "Wordfence"
- **Lokasi:** `smk10novembercibitung.sch.id/wp-content/plugins/wp-wordfence-security/`
- **File & hash:**

| File | SHA-256 | Fungsi |
|---|---|---|
| `proc.php` (562 B) | `a8a2bb81720f50258555598c8af80f478dd698cbb9c6127a2d798f4ed360eb48` | `proc_open($_GET['data-admin'], …)` → output perintah di-echo |
| `pcntl.php` (892 B) | `4f9ff8be285193ae0f66b10b3ff7892284b6f288941fb6e5f799e8b9902b61ef` | `pcntl_exec('/bin/sh', ['-c', $_GET['data-admin']])` → RCE |
| `wp-wordfence-security.php` (358 B) | `be7e3a1905b9439b9fe526962f2eb51ceeee9dd84050f587dc8f692232917881` | header plugin palsu saja (`Author: exe`, "Adds a custom notification text") |
| `info.php` (20 B) | `9c187470da5b287bf03552bcc36ef74d3a636125aa7e8492f64c3ef89c046ee3` | `<?php phpinfo(); ?>` |
| `.htaccess` (32 B) | `513d953c015debdbb2a6604f8f7a89dad2ccbee32cfe5c3600a43ae4ef659d2f` | `Options -Indexes` / `Allow from all` |

- **Analisis:** kode RCE **eksplisit dan tidak terobfuskasi**; tidak ada keraguan interpretatif. Nama plugin meniru produk keamanan resmi (Wordfence asli bernama `wordfence`).
- **Risiko:** **Kritis.** **Keyakinan:** Tinggi.

### F-03 — Webshell RCE bernama-tema (root & sragen)
- `wp-content/themes/wp_edvl/404.php` (1.892 B) — SHA-256 `2c37260b3eb8eee8d156d1af1d088d78cbd29ddc183753c6ca2ed883dc05b258`
- `smk10novembersragen.sch.id/wp-content/themes/wp_zqyq/404.php` (1.892 B) — SHA-256 `a1165eec7ef1ee43202e9617c0d6e5a297450858e67a143b06b718373b09388d`
- **Mekanisme:** gate `?g=admin0995` (root) / `?g=admin0978` (sragen); payload `POST[c]` = base64 **XOR** dengan key `TTTDowpf` / `VKl1tNpC`; eksekusi via `system` → `passthru` → `shell_exec` → `exec`.
- **Risiko:** **Kritis.** **Keyakinan:** Tinggi.

### F-04 — Webshell upload file (sragen)
- **Path:** `smk10novembersragen.sch.id/wp-content/themes/uvisyfyj/index.php` (368 B)
- **SHA-256:** `7adfdaa88c62766e5343813d90dba0f2087d034ffb76402d8ae82e14aa757bf9`
- **Bukti:** gate `?g=admin888`; jika cocok, `move_uploaded_file($_FILES['f']['tmp_name'], $_FILES['f']['name'])` → **upload file arbitrer** (jalur langsung menuju RCE), jika tidak cocok menampilkan form upload.
- **Risiko:** **Kritis.** **Keyakinan:** Tinggi.

### F-05 — Cluster webshell terobfuskasi (sragen)
- **On-disk:** `goodsdetails.php` (177.474 B) — SHA-256 `3d6b373e6489b163a9d7d40c8819517bc720f1c4973f497c81e69be7ec57fc0f`
  - Rantai terverifikasi: `eval(base64_decode('<177.444 char>'))` → `eval(base64_decode("<33.788 char>"))` → `eval(gzinflate(base64_decode("…")))` → payload.
- `shopsdetails.php` (15.089 B) — SHA-256 `ea1e3e379045730c422db5f065808dcf43f13c8febb2d44450f1d453cc3bcb57`
  - Pola `goto`-spaghetti + `base64_decode` + `gzinflate` + pergeseran karakter `-4` + key literal.
- **Arsip pembanding di root sragen** (isi `.zip` merekam **versi asli/lebih awal** dari dropper ini):

| Arsip | Member & ukuran | Timestamp internal |
|---|---|---|
| `item.zip` | `item.php` 26.227 B | 2026-08-31 15:36 |
| `hiroshi.zip` | `hiroshi.php` 106.425 B | 2026-09-07 11:45 |
| `wp-blog-header.zip` | `wp-blog-header.php` 12.207 B | 2026-09-07 11:40 |
| `google006f33b90c2ea6f4.zip` | html verifikasi Google 53 B | 2026-09-15 17:28 |
| `shopsdetails.zip` | `shopsdetails.php` 15.089 B | 2026-09-22 14:44 |
| `goodsdetails.zip` | `goodsdetails.php` 31.646 B | 2026-09-22 15:00 |
| `index.zip` | `index.php` 115.859 B | 2026-09-22 15:05 |

  → `goodsdetails.php` on-disk (177.474 B) jauh lebih besar dari versi arsip (31.646 B): berkas tumbuh setelah pengarsipan.
  → `hiroshi.php` dan `item.php` **hanya ada di dalam arsip**, tidak ada di filesystem.
  → `wp-blog-header.php` di arsip berbeda total dari `wp-blog-header.php` WordPress asli (351 B); isinya dropper `PHP Encode v1.0 by zeura.com` + `file(__FILE__)` (membaca dirinya sendiri untuk menyusun payload).

- **Koroborasi eksternal (AV):** Windows Defender mendeteksi & mengarantina `wp-blog-header.php` yang saya ekstrak ke folder laporan.
  **ThreatID `2147761377`**, waktu deteksi `13/10/2026 12:53:39` — bukti independen bahwa berkas ini benar-benar malware, bukan sekadar pola aneh.
- **Risiko:** **Kritis.** **Keyakinan:** Tinggi (untuk keluarga/rantai); payload terdalam masih perlu dituntaskan (lihat §5).

### F-06 — Malware SEO-cloaking `wp-slgnup.gz`
- **Path:** `wp-slgnup.gz` (2.640 B) — SHA-256 `9dc33afe68232584b0c00fe5e4feb1211da77b7aff5183bb24c3441e0b74e98d`
- **Isi dekompresi:** 8.121 B PHP berobfuskasi `goto` + escape oktal/hex + `str_rot13`.
- **Fungsi teridentifikasi:** `doutdo()` mengambil konten dari daftar domain C2 via `curl`/`file_get_contents`, menyuntikkannya ke respons, dengan penanganan khusus per-tipe (`okhtmlgetimg`, `okhtmlgetcss`, `okhtmlgetcontent`, `okxmlgetcontent`, …), plus penulisan ulang `robots.txt`.
- **Domain C2 (rot13/urldecode statis, tanpa koneksi jaringan):**
  - `3502-rank213.expansor.xyz`
  - `3502-rank213.clarivum.top`
  - `3502-rank213.zephyrous.xyz`
  - `3502-rank213.quiddity.top`
- **Nama file meniru `wp-signup.php`** (typo "slgnup") — teknik penyamaran.
- **Risiko:** Tinggi (SEO/hijack trafik + cloaking). **Keyakinan:** Tinggi.

### F-07 — Cluster SEO doorway/spam (kebumen)
- **Lokasi:** `smk10novemberkebumen.sch.id/prestasi/`
- **File:** `index.php` (187.018 B, `46977f41…`), `bb.php` (3.898 B, `6bd45e8b…`), `home.php` (141.813 B), `lol.txt` (371.778 B, `1b18cc07…`), `urls.txt` (1.841.790 B, `94fb5b61…`), `site1–4.xml` (~1,2 MB masing-masing), `robots.txt`, `sitemap.xml`, html verifikasi Google.
- **Bukti perilaku:**
  - `index.php`: `error_reporting(0); set_time_limit(0);` + `function denied()` yang `http_response_code(404)` lalu menampilkan `home.php` → **cloaking 404**; membaca `lol.txt` dan menyajikan halaman berdasarkan `$_GET['id']`.
  - `bb.php`: menulis ulang `robots.txt` (`file_put_contents`) menunjuk ke `sitemap.xml`, membaca `lol.txt` dengan limit 9.900 → generator massal.
- **Risiko:** Tinggi (spam SEO / penyalahgunaan reputasi domain). **Keyakinan:** Tinggi untuk perilaku; atribusi ke aktor yang sama masih dugaan.

### F-08 — Jejak eksekusi & kompromi pada `error_log`
- **Path:** `error_log` (1.771.807 B) — SHA-256 `e93ec877e06612966cffd7c4bfe85de01c342805da4068688c20dccb00f5face`
- **Rentang:** `27-Mar-2026 14:10:50 UTC` → `12-Oct-2026 00:14:17` (zona waktu **tercampur** UTC & Asia/Jakarta dalam satu log).
- **9.263 baris:** 7.755 `PHP Warning`, 41 `PHP Fatal error`.
- **Temuan konkret:**
  1. `PHP Fatal error: Call to undefined function shell_exec() in /home/smkq8953/public_html/anjv2.php:98` — **4 kali, 09-Aug-2026 23:31:56–23:32:05** (percobaan eksekusi perintah berulang; gagal karena `shell_exec` dinonaktifkan). **`anjv2.php` tidak ada di salinan ini.**
  2. **282×** `Constant DB_NAME/DB_USER/DB_PASSWORD/… already defined` pada `wp-config.php` baris 23–94 → `wp-config.php` **di-include lebih dari sekali dalam satu request** (pola khas proses PHP/webroot yang memuat WP langsung, bukan alur normal WP).
  3. 1.875× / 926× warning di `wp-includes/rest-api/class-wp-rest-server.php` (baris 1841/1853) — kemungkinan rest-api diakses langsung/di luar konteks normal; belum dipastikan terkait kompromi.
  4. 18 referensi `/tmp/phpXXXXXX` (file temporary upload PHP) → indikasi aktivitas upload script pada periode log.
  5. Referensi plugin `wp-compat.php` yang **tidak ada** di root install (hanya `wp-compat` kosong di sragen).
- **Catatan metodologis:** timestamp **tidak** dianggap bukti pasti waktu kompromi.

### F-09 — `mu-plugins/sso-loader.php`: endpoint login tanpa password
- **Path (2 lokasi, hash identik):** `wp-content/mu-plugins/sso-loader.php`, `smk10novembersragen.sch.id/wp-content/mu-plugins/sso-loader.php`
- **SHA-256:** `337fee21376d07b27f6bcf7ef7c85e74c3273aadff491137f524ab1cb1aa111d` (2.861 B)
- **Fungsi:** hook `wp_ajax_nopriv_sso-check`; mencocokkan `base64(sha256(nonce.salt))` dengan opsi `sso_token`; bila cocok → `wp_set_auth_cookie()` + `do_action('wp_login')` untuk user yang dipilih (default: **administrator pertama**) → **login admin tanpa password**.
- **Penilaian:** kode ini **menyerupai plugin "SSO" pihak ketiga (kredit "Garth Mortensen, Mike Hansen")**, jadi **belum dapat disebut malware hanya dari kode**. Namun posisinya di `mu-plugins` dan keberadaan endpoint login tanpa password menjadikannya kandidat backdoor berisiko tinggi bila nilai `sso_token` lemah/diketahui.
- **Risiko:** Tinggi (potensial). **Keyakinan:** **Sedang — perlu verifikasi** (nilai `sso_token` ada di DB, **tidak diperiksa** sesuai aturan 9).

### F-10 — Bukti akses langsung ke `sso-loader.php`
- `wp-content/mu-plugins/error_log` (580 B, `bd16c68ce6d93b73…`): 2× `Call to undefined function add_action()` di `sso-loader.php:62` (**28-Sep-2026 13:39:50**, **09-Oct-2026 22:07:53**).
- `smk10novembersragen.sch.id/wp-content/mu-plugins/error_log`: 5 kejadian serupa (**28-Sep**, 29-Sep ×2, 03-Oct, 09-Oct 2026).
- **Interpretasi:** file di `mu-plugins` dapat diakses langsung via web; fatal `add_action()` membuktikan file **dipanggil langsung** (di luar bootstrap WordPress) pada tanggal-tanggal tersebut.

---

## 3. Temuan yang PERLU VERIFIKASI (belum disimpulkan)

| ID | Path | Fakta | Mengapa belum disimpulkan |
|---|---|---|---|
| P-01 | `smk10novembercibitung.sch.id/wp-loadss.php` (3.937 B) | byte-identik dengan `wp-load.php` kecuali newline akhir | Duplikat nama mirip. Bisa artefak copy, bisa entry point alternatif. **Tidak ada bukti perilaku jahat.** |
| P-02 | `kalender-akademik/index.php` (2.560.534 B) | HTML murni (`<?php` = 0) | Berukuran ekstrem tetapi tidak mengandung PHP. Kemungkinan halaman hasil ekspor yang sah. |
| P-03 | `wp-content/wp-content.txt` (200.789 B, mtime 23:47) | output `tree` direktori volume `Data` | **Kemungkinan besar artefak persiapan salinan**, bukan serangan. Perlu konfirmasi siapa yang membuat. |
| P-04 | `sso-loader.php` (F-09) | endpoint login tanpa password | Perlu cek nilai `sso_token` di DB / apakah plugin SSO resmi memang dipasang. |
| P-05 | Plugin `ioyntkq` (cibitung) | direktori bernama acak, isinya plugin sah "Protect Uploads" | Nama direktori diacak = mencurigakan; isi kode tampak asli. Perlu bandingkan dengan rilis resmi. |
| P-06 | Plugin `lvcibfqxpseo` (cibitung) | **direktori kosong** kecuali `.htaccess` 32 B | Sisa plugin SEO spam yang sudah dihapus — sisa nama acak. |
| P-07 | Plugin `all-in-one-wp-migration-unlimited-6.77.1` (cibitung & kebumen) | versi "unlimited" (premium) | Indikasi plugin nulled/cracked — vektor infeksi klasik. Perlu verifikasi lisensi/asaldimana. |
| P-08 | `wp-content/themes/wp_edvl`, `wp_zqyq`, `uvisyfyj` | direktori tema dengan `style.css` 25–39 B + `index.php` | Tema "cangkang" palsu. Sudah dikaitkan ke webshell (F-03/F-04). |
| P-09 | File `uploads/**/<angka>.php` **0 byte** (30+ file, semua instalasi) | placeholder 0 byte di folder tahun | Tidak berisi kode. Kemungkinan artefak plugin/tool. Rendah. |
| P-10 | `error_log` root: 1.875×/926× warning di `class-wp-rest-server.php` | akses REST API di luar konteks normal | Belum dapat dipastikan terkait kompromi. |
| P-11 | `.zip` (nama file tersembunyi, 280 B) di root sragen | arsip berisi `.htaccess` 204 B | Arsip sendiri; perlu dikaitkan dengan backup/eksfiltrasi. |
| P-12 | `.tmb`, `.well-known` (0 B) di root beberapa instalasi | file kosong | Perlu konfirmasi apakah normal. |

---

## 4. Verifikasi file inti vs paket resmi WordPress 7.1.3 — **BELUM DAPAT DILAKUKAN**

- Pencarian lokal (tanpa unduh apa pun) di `D:\` (depth 2–4), `C:\Users\sakha\Downloads`, `C:\Users\sakha\Documents` untuk `wordpress*.zip`, `wordpress*.tar.gz`, `wordpress*.7z/.rar`, direktori `wordpress-7.1.3*`, folder `wordpress`, serta berkas checksum (`.md5`, `.sha1`): **tidak ditemukan**.
- **Status: BLOCKED.** Perbandingan hash file inti (`wp-admin/`, `wp-includes/`) terhadap paket resmi **belum dapat dilakukan** sampai paket WordPress 7.1.3 resmi (dan/atau daftar checksum resmi) disediakan secara lokal.
- Sementara itu, manifest SHA-256 sudah siap dipakai sebagai basis perbandingan begitu paket tersedia.

---

## 5. Risiko integritas bukti (perlu keputusan Saka)

Windows Defender **aktif dan bereaksi** terhadap isi folder ini:
- Terdeteksi: `wp-blog-header.php` hasil ekstraksi (ThreatID `2147761377`) pada `13/10/2026 12:53:07` dan `12:53:39`.
- Konsekuensi: (a) berkas itu kini terarantina/tidak dapat dibaca; (b) **risiko nyata** bahwa file malware di `D:\smk10november` juga dikarantina oleh Defender kapan saja → **bukti berubah/berkurang**.
- Mitigasi yang disarankan (butuh persetujuan terpisah): selesaikan manifest hash lebih dulu, lalu masukkan `D:\smk10november` ke daftar **exclusion** sementara **atau** gandakan bukti ke media terpisah sebelum analisis lebih lanjut. Saya **tidak** mengubah konfigurasi Defender.

---

## 6. Next verification steps (usulan, belum dijalankan)

1. **Selesaikan manifest penuh** 38.906 file + arsipkan bersama laporan (sebagian sudah ada).
2. Sedlakan **paket WordPress 7.1.3 resmi** secara lokal → jalankan diff hash `wp-admin/` + `wp-includes/` untuk menemukan file inti yang dimodifikasi.
3. **Tuntaskan decoding** payload terdalam `goodsdetails.php` / `hiroshi.php` / `item.php` / `index.php` (rantai bersarang >3 lapis; sudah terbukti `eval(base64_decode)`/`gzinflate`). Tempelkan keluarga malware.
4. **Lacak `anjv2.php`** (dirujuk `error_log`) — cari di backup, arsip AI1WM, atau log akses web server; minta log akses HTTP dari hosting untuk 09-Aug-2026 23:31 (WIB/UTC).
5. **Periksa DB (read-only)**: nilai opsi `sso_token`, daftar user admin + tanggal registrasi, opsi/`active_plugins` mencurigakan, dan post/option bertema SEO doorway. **Butuh kredensial terpisah dari Saka.**
6. **Korelasikan artefak arsip** (`item.zip`, `hiroshi.zip`, `wp-blog-header.zip` dll.) dengan log akses untuk menentukan urutan drop.
7. Konfirmasi apakah `wp-content.txt`, `wp-loadss.php`, `kalender-akademik/`, dan `.tmb`/`.well-known` berasal dari proses penyalinan (bukan serangan).
8. Bandingkan plugin `ioyntkq`, `custom-codes`, `super-easy-maintenance-mode`, `all-in-one-wp-migration-unlimited-6.77.1` dengan rilis resmi (hash).

---

## 7. Ringkasan tingkat risiko

| Tingkat | Item |
|---|---|
| **Kritis** | F-02 (RCE plugin palsu Wordfence), F-03 (RCE tema wp_edvl/wp_zqyq), F-04 (upload shell uvisyfyj), F-05 (keluarga webshell terobfuskasi + dropper terarsip) |
| **Tinggi** | F-01 (defacement), F-06 (SEO-cloaking + C2), F-07 (cluster SEO doorway), F-08 (jejak `shell_exec` gagal), F-09/F-10 (sso-loader + akses langsung) |
| **Sedang/Rendah** | P-01, P-03, P-05…P-12 |

---

## 8. Kepatuhan protokol

| Aturan | Status |
|---|---|
| 1. Tidak menulis/mengubah/memindah/menghapus di `D:\smk10november` | **Dipatuhi** — semua tulis hanya di `C:\Users\sakha\forensik-smk10november\` |
| 2. Tidak menjalankan file dari folder bukti; tanpa jaringan/hosting/DB/kredensial | **Dipatuhi** — decoding dilakukan statis (Python), tanpa eksekusi PHP |
| 3. Tidak membuat salinan/laporan di dalam direktori bukti | **Dipatuhi** |
| 4. SHA-256 read-only + manifest di luar direktori bukti | **Dipatuhi** (manifest prioritas + manifest penuh inkremental) |
| 5. Inventarisasi file & metadata lebih dulu | **Dipatuhi** |
| 6. `error_log` bertahap; timestamp bukan bukti pasti | **Dipatuhi** |
| 7. Periksa artefak bernama mencurigakan tanpa menyimpulkan dari nama saja | **Dipatuhi** (mis. `wp-loadss.php` → P-01, `sso-loader.php` → P-04) |
| 8. Verifikasi inti vs WP 7.1.3 lokal; tanpa unduh | **Dipatuhi** — hasil: paket tidak tersedia → dilaporkan BLOCKED |
| 9. Tidak membaca/mencetak rahasia `wp-config.php`/kredensial | **Dipatuhi** — hanya hash & metadata; pesan log di-mask |
| 10. Laporkan path, hash, bukti, risiko, keyakinan, langkah berikutnya | **Dipatuhi** |

**Tidak ada remediasi yang dilakukan. Berhenti sebelum perubahan apa pun, menunggu persetujuan terpisah.**
