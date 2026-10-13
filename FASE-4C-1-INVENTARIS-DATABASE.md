# FASE 4C.1 — Inventarisasi & Audit Statis Database WordPress

- **Lokasi sumber:** `D:\backup-database-smk10november`
- **Tanggal pemeriksaan:** 2026-10-13 (≈23:30–24:00 WIB)
- **Tanggal berkas (mtime):** 2026-10-13 23:26, 23:27, 23:27, dan 19:40
- **Sifat kerja:** READ-ONLY. Tidak ada berkas sumber yang diubah/dipindah/diganti nama/dihapus.
- **Tidak dilakukan:** impor SQL, eksekusi query (lokal/remote), koneksi jaringan, menjalankan skrip di dalam dump.
- **Tidak ditampilkan (aturan 6/8):** `user_pass` (hash), token, cookie, salt, session token, API key. Nilai opsi hanya dilaporkan sebagai **panjang** + **autoload**. Login pengguna **TERMASK**; email hanya **domain**-nya.
- **Laporan disimpan di** `C:\Users\sakha\forensik-smk10november\audit-backup\` (bukan di folder sumber).
- **Aturan 10 dipatuhi:** tidak ada database yang dinyatakan bersih; seluruh pernyataan di bawah dibatasi pada apa yang benar-benar diperiksa.

---

## 1. Ringkasan jumlah database dan arsip

| Jenis | Jumlah | Catatan |
|---|---|---|
| **Dump SQL (teks/plain)** | **4** | Semua **tidak** dikompresi, **tanpa arsip** |
| Arsip ZIP/GZ/TAR | **0** | Tidak ada arsip di direktori ini |
| Subdirektori | 0 | Semua berkas berada di level-1 |
| Total ukuran | **342.655.406 B (±342,7 MB)** | — |

Keempat dump dibaca **100%** (bukan sampel) untuk pencarian pola opsi/user/tabel; namun lihat §7 untuk batas atribusi per-baris pada tabel besar.

---

## 2. Daftar file dan SHA-256

| # | Nama file | Ukuran (B) | mtime | SHA-256 | Jenis (dari isi) | Sandi |
|---|---|---|---|---|---|---|
| 1 | `smkq8953_utama1.sql` | 80.550.250 | 2026-10-13 19:40 | `04a7b303ac978e4cbd1dacac6a69a56b9742bbf01dad73d588d014ed82309506` | teks/plain (bukan gzip) | tidak ada |
| 2 | `smkq8953_cbitung.sql` | 85.372.896 | 2026-10-13 23:26 | `85d9820370f9044bace33d0c81105c8ed0932d5082a1427a4245ecf2e031ffc3` | teks/plain | tidak ada |
| 3 | `smkq8953_kebumen.sql` | 95.834.894 | 2026-10-13 23:27 | `d91e42bfa1e480995f8c0ea2af869a6d028f85ab05fffb9fd0ca85f02fa1eedc` | teks/plain | tidak ada |
| 4 | `smkq8953_sragen10.sql` | 80.897.366 | 2026-10-13 23:27 | `0448259bb0a117636e2e40bcdd11b15ea2f66d1f7615a84769dac0680b46f3af` | teks/plain | tidak ada |

**Catatan silang:** `smkq8953_utama1.sql` **identik** (SHA-256 sama) dengan berkas bernama sama di `C:\Users\sakha\Downloads\` (laporan Fase 4A) — salinan dari berkas yang sama. Berkas `index.htm` (Fase 4A) menunjukkan dump ini berasal dari phpMyAdmin di host `brantas.iixcp.rumahweb.net`, database `smkq8953_utama1`.

**Pola format dump (konsisten keempatnya):** `INSERT INTO \`tabel\` (\`kolom\`, ...) VALUES` dengan **spasi setelah koma**; string di-escape dengan `\'` dan `\"`. Format ini yang membuat beberapa pola awal saya gagal cocok (dikoreksi; lihat §7).

---

## 3. Pemetaan kandidat database ke domain

Pemetaan **tidak** memakai nama berkas saja, tetapi dari **isi**: opsi `siteurl`, `home`, `blogname`, dan domain pada URL di seluruh dump.

| Berkas | `siteurl` / `home` | `blogname` | Domain | Keyakinan |
|---|---|---|---|---|
| `smkq8953_utama1.sql` | `https://smk10novembertambun.sch.id` | SMK 10 November Tambun Selatan | **tambun** | **Tinggi** |
| `smkq8953_cbitung.sql` | `https://smk10novembercibitung.sch.id` | SMK 10 November Cibitung | **cibitung** | **Tinggi** |
| `smkq8953_kebumen.sql` | `https://smk10novemberkebumen.sch.id` | SMK 10 November Kebumen | **kebumen** | **Tinggi** |
| `smkq8953_sragen10.sql` | `https://smk10novembersragen.sch.id` | SMK 10 November Sragen | **sragen** | **Tinggi** |

**Kesimpulan pemetaan: 4/4 terpetakan, 0 UNKNOWN.** `siteurl` dan `home` konsisten satu sama lain (tidak ada indikasi pengalihan URL).

---

## 4. Nama database dan prefiks tabel yang ditemukan

| Domain | Prefiks tabel | Jumlah tabel berisi data | Blok INSERT | Jumlah nama opsi unik | Nama DB (indikasi) |
|---|---|---|---|---|---|
| tambun | **`smk_`** | 39 | 1.455 | 315 | `smkq8953_utama1` (dari `index.htm`, Fase 4A) |
| cibitung | **`cbt_`** | 38 | 1.567 | 275 | **`smkq8953_cbitung`** (dari nama berkas) |
| kebumen | **`kbm_`** | 39 | 1.710 | 295 | **`smkq8953_kebumen`** (dari nama berkas) |
| sragen | **`srg_`** | 39 | 1.465 | 292 | **`smkq8953_sragen10`** (dari nama berkas) |

Catatan: dump **tidak** memuat pernyataan `CREATE DATABASE` maupun `USE`, sehingga nama database hanya dapat disimpulkan dari nama berkas dan (untuk tambun) dari sumber phpMyAdmin. Prefiks tabel **berbeda untuk setiap situs** (bukan `wp_`), yang wajar untuk pemasangan terpisah.

**Opsi inti yang terverifikasi ada di keempat DB:** `siteurl`, `home`, `active_plugins`, `cron`, `hack_file` (opsi bawaan WordPress — **bukan** indikator).

---

## 5. Temuan mencurigakan

### 5.1 Akun administrator — TEMUAN UTAMA (keyakinan **TINGGI**)

| Domain | Total user | Total **administrator** | Administrator dibuat **≥ 2026-07-25** | User lama (< 2026-01-01) |
|---|---|---|---|---|
| **tambun** | 41 | **41** | **40** | 1 |
| **cibitung** | 90 | **90** | **89** | 1 |
| **kebumen** | 2 | **2** | **1** | 1 |
| **sragen** | 30 | **30** | **29** | 1 |
| **TOTAL** | 163 | **163** | **159** | 4 |

**Bukti karakteristik akun baru (bukan pola normal sekolah):**
1. **Semua akun baru berperan `administrator`** (bukan subscriber/editor).
2. **Registrasi terkumpul rapat** pada 25-Jul-2026 s/d 11-Sep-2026 — termasuk beberapa akun dalam hitungan **detik** yang sama (mis. cibitung id=9,10,11,12,13 pada 04:04:53–04:05:24) ⇒ pola **otomasi**.
3. **Email pada domain yang tidak dapat diresolusi**: `*.invalid`, `*.local`, `*.localhost`, `*.research` — ditambah `wordpress.com`, `goole.com` (salah ketik `google`), serta sebagian `gmail.com`/`outlook.com`.
4. **Login berpola nama tool**: akun dengan awalan `wp2***` (cocok dengan `wp2shell`), `w2s***`, `rce***`, `Nx_***`, `Bun***`, `HI_***`, `bdk***`, `nul***`, `upg***`.
5. **Hanya 1 akun lama per situs**: `adm***` dengan domain email resmi sekolah, terdaftar **2025-10-30 09:08:29** — **timestamp identik di keempat DB** ⇒ keempat situs diprovisi dari templat yang sama pada waktu itu.

> **Login ditampilkan TERMASK** (3 karakter pertama + `***`) sesuai aturan 8. Daftar lengkap dapat disiapkan terpisah bila diperlukan untuk pembersihan.

### 5.2 Opsi jejak tool: `wp2shell_<hex>` (keyakinan **TINGGI**)

| Domain | Nama opsi | Panjang nilai | Catatan |
|---|---|---|---|
| tambun | `wp2shell_456ffcd40df5` | 63 | di **3** situs yang sama dengan ledakan akun admin |
| cibitung | `wp2shell_e0f4a8f8952f` | 63 | — |
| sragen | `wp2shell_aa673039837f` | 63 | — |
| kebumen | **(tidak ada)** | — | konsisten dengan kebumen yang hanya punya 2 admin |

Nilai opsi **tidak ditampilkan** (63 karakter, terindikasi data non-publik). Keberadaan opsi bernama tool webshell ini, yang **hanya ada di situs dengan ledakan akun admin**, merupakan korelasi kuat.

### 5.3 Opsi `sso_token` — silang-bukti dengan Fase 3A (keyakinan **TINGGI**)

| Domain | `sso_token` | Panjang | `sso-loader.php` di filesystem (Fase 3A) |
|---|---|---|---|
| **tambun** | **ADA** | 64 char | **ADA** (`wp-content/mu-plugins/`) |
| **cibitung** | tidak ada | — | tidak ada |
| **kebumen** | tidak ada | — | tidak ada |
| **sragen** | **ADA** | 64 char | **ADA** (`wp-content/mu-plugins/`) |

Korelasi **sempurna**: berkas `sso-loader.php` ada tepat di dua situs yang opsinya ada. Panjang 64 karakter konsisten dengan skema plugin (`base64_encode(hash('sha256', …, false))` lalu `substr(…, 0, 64)`). **Nilai token tidak dibaca/ditampilkan.** ⇒ Endpoint login tanpa password (temuan A10/F-09 Fase 3A) **aktif pada tambun dan sragen**.

### 5.4 Plugin aktif — korelasi dengan filesystem (keyakinan **TINGGI**)

| Domain | Plugin aktif (dari `active_plugins`) |
|---|---|
| tambun | `super-easy-maintenance-mode`, **`wp-file-manager`** |
| cibitung | `classic-editor`, `disable-wordpress-updates`, `elementor`, `elementor-pro`, `elementskit-lite`, `skyboot-…`, **`wp-file-manager`** |
| kebumen | `classic-editor`, `disable-wordpress-updates`, `download-manager`, `elementor`, `elementor-pro`, `elementskit-lite`, `skyboot-…` |
| sragen | `custom-codes`, **`wp-file-manager`** |

- **`wp-file-manager` (plugin manajer berkas) aktif di tambun, cibitung, dan sragen — tiga situs yang sama** dengan ledakan akun admin. Kebumen **tidak** memakainya.
- **Penting:** plugin berbahaya `wp-wordfence-security` (webshell RCE, Fase 3A F-02) **TIDAK terdaftar di `active_plugins`** — konsisten dengan bukti `error_log` Fase 2/3A bahwa berkasnya diesksekusi **langsung via HTTP**, tanpa perlu aktivasi.
- `kebumen` = satu-satunya situs tanpa `wp-file-manager`, tanpa `wp2shell_*`, tanpa `sso_token`, dan hanya 2 akun admin ⇒ **kebumen berbeda jalur/tingkat dampak** (konsisten dengan temuan filesystem: hanya cluster SEO doorway).

### 5.5 Konten (posts) — indikator (keyakinan **SEDANG**)

| Domain | `<script` | `<iframe` | `<?php` | `eval(` | `base64_decode` |
|---|---|---|---|---|---|
| tambun | 0 | 48 | **1** | 0 | 0 |
| cibitung | 0 | 176 | 0 | 0 | 0 |
| kebumen | 0 | 192 | 0 | 0 | 0 |
| sragen | 0 | 34 | **1** | 0 | 0 |

- **Tidak ditemukan** `eval(`, `base64_decode`, atau `<script` di dalam konten ⇒ tidak ada bukti injeksi JavaScript/PHP di dalam basis data pada pola yang diuji.
- Kemunculan `<?php` (1 di tambun, 1 di sragen) sudah **diperiksa konteksnya**: merupakan **contoh kode bawaan** (`'<?php echo date( \'Y\' ); ?>'`, tag `'sample, dates'`) — **bukan** malware.
- `<iframe` 34–192 kemunculan berasal dari embed (YouTube/Elementor) — **belum** diklasifikasikan satu per satu; tidak ada domain C2 yang muncul pada daftar host eksternal dump (hanya host sah: `library.elementor.com`, `www.w3.org`, `downloads.wordpress.org`, `wa.me`, `maps.google.com`, dsb.).

### 5.6 Cron (keyakinan **TINGGI — negatif**)

Tidak ada hook cron mencurigakan. Hook yang ada adalah hook bawaan WordPress/plugin (`wp_scheduled_delete`, `delete_expired_transients`, `wp_update_user_counts`, `actionscheduler_*`, `bbp_license_sync`, `__wpdm_cron`). Beberapa nama seperti `args`, `schedule`, `interval`, `daily`, `weekly`, `version` adalah **kunci struktur serialisasi**, bukan nama hook.

### 5.7 Opsi lain yang mirip-mencurigakan tetapi **wajar** (bukan IoC)

`hack_file` (opsi bawaan WordPress, panjang 1), `ai1wm_secret_key`, `session_tokens`, `subscriber_link_tokens`, `unsubscribe_tokens`, `aioseo_activation_redirect`, dan sederet `elementor_pro_*_api_key` (**semuanya panjang 0** kecuali `ai1wm_secret_key`=12). Kata "secret/token/key" memicu filter saya, tetapi **tidak ada indikasi penyalahgunaan** pada nilai/panjangnya. **Nilai tidak ditampilkan.**

---

## 6. Database yang belum dapat dipetakan

**Tidak ada.** Keempat berkas berhasil dipetakan ke domain dengan keyakinan tinggi (§3).

Yang belum dapat dipastikan: **nama database resmi di sisi server** untuk cibitung/kebumen/sragen (dump tidak memuat `CREATE DATABASE`/`USE`). Indikasi kuat dari nama berkas: `smkq8953_cbitung`, `smkq8953_kebumen`, `smkq8953_sragen10` — perlu konfirmasi dari cPanel/hosting.

---

## 7. Keterbatasan pemeriksaan

1. **Tidak ada impor/eksekusi SQL.** Seluruh analisis berbasis pembacaan teks + regex. Tidak ada query dijalankan pada MySQL.
2. **Atribusi per-baris tidak lengkap untuk tabel besar.** Untuk `posts`/`postmeta` (ribuan blok INSERT, nilai sangat besar), indikator dihitung **global per dump** (jumlah kemunculan pola di seluruh berkas) — **bukan** per-post. Karena itu: **saya tidak dapat menyatakan post ID tertentu bersih/kotor**, dan tidak dapat menyatakan seluruh konten sudah diaudit baris demi baris.
3. **Koreksi metodologi yang saya lakukan sendiri:** tiga iterasi skrip parser saya gagal lebih dulu karena asumsi format yang salah — (a) mengabaikan **spasi setelah koma** pada dump ini, (b) pola pengambilan nilai melewatkan **kutip pembuka**, (c) deteksi role memakai *substring* `administrator` yang menghasilkan false positive terhadap entri `administrator";b:0`. Semua sudah dikoreksi (v5) dan hasil final diverifikasi dengan sampel baris mentah. Angka pada §5 berasal dari versi yang sudah dikoreksi.
4. **`last login` tidak tersedia.** `wp_users` tidak menyimpan waktu login terakhir, sehingga akun tidak dapat diurutkan berdasarkan aktivitas; opsi `session_tokens` **sengaja tidak dibaca** (rahasia).
5. **Deteksi "akun baru" memakai cutoff 2026-01-01.** Kompromi yang membuat akun **sebelum** tanggal itu tidak akan tertandai hanya dari tanggal registrasi.
6. **Nilai sensitif tidak dibaca.** Pada dump tambun terdapat **2.734** kemunculan kata sensitif (`pass/token/secret/salt/key`); semuanya **tidak** ditampilkan.
7. **`active_plugins` hanya mencerminkan plugin terdaftar.** Webshell berbasis berkas (mis. `wp-wordfence-security`, `themes/archive.php`, injeksi di `wp-signup.php`) **tidak** muncul di daftar ini — jadi ketiadaan plugin mencurigakan di `active_plugins` **bukan** bukti situs bersih.
8. **Satu DB belum dibandingkan dengan dump lain dari sumber berbeda** (mis. dump DB dari hosting), sehingga tingkat kelengkapan/kesegaran setiap dump tidak diverifikasi.
9. File dump besar (80–96 MB) dibaca utuh ke memori; cap 6 MB diterapkan hanya pada pembacaan nilai opsi tunggal (`value_of`).

---

## 8. Rekomendasi tahap berikutnya

| # | Tindakan | Prasyarat |
|---|---|---|
| 1 | **Pembersihan akun administrator**: identifikasi & hapus **159 akun admin** yang dibuat ≥ 2026-07-25 (tambun 40, cibitung 89, sragen 29, kebumen 1), pertahankan akun `adm***` (2025-10-30) setelah verifikasi | Persetujuan; **jangan** jalankan DML pada produksi sebelum backup terverifikasi |
| 2 | **Nonaktifkan/cabut `sso_token`** di tambun & sragen dan hapus `sso-loader.php` — endpoint login tanpa password | Persetujuan (remediasi, di luar fase ini) |
| 3 | **Audit plugin `wp-file-manager`** (aktif di 3 situs) beserta berkas webshell yang **tidak** terdaftar sebagai plugin (`wp-wordfence-security`, `themes/archive.php`, injeksi `wp-signup.php`/`class-wp-query.php`) | Persetujuan remediasi |
| 4 | **Analisis mendalam `wp2shell_*`**: identifikasi tool/keluarga malware dari nama opsi + pola akun, lalu bandingkan dengan indeks AV/threat intel | Tidak ada (bisa statis); butuh sumber intel eksternal untuk pencocokan |
| 5 | **Atribusi per-post** untuk 34–192 `<iframe` dan semua `postmeta` bertipe Elementor, memakai parser yang sudah dikoreksi | Tidak ada |
| 6 | **Verifikasi kelengkapan dump** (apakah ada tabel yang sengaja dikosongkan; bandingkan jumlah baris dengan situs live) | Akses DB read-only |
| 7 | **Korelasi lintas-fase**: silangkan daftar akun admin + `wp2shell_*` + `sso_token` dengan temuan berkas Fase 3A dan log akses HTTP (bila tersedia) | Log akses dari hosting (butuh OTP — di luar izin) |
| 8 | Perluas cakupan pencarian DB/backup lain (§8 laporan Fase 4A) bila masih ada dump lain | Persetujuan Saka (aturan 8 Fase 4A) |

---

## 9. Kepatuhan aturan

| Aturan | Status |
|---|---|
| Read-only terhadap semua file sumber | **Dipatuhi** — tidak ada berkas sumber yang diubah; semua keluaran ke `audit-backup\` |
| Tidak mengubah/memindah/mengganti nama/menghapus | **Dipatuhi** |
| Tidak mengimpor atau mengeksekusi SQL | **Dipatuhi** — tidak ada koneksi DB sama sekali |
| Tidak menjalankan query ke DB lokal/remote | **Dipatuhi** |
| Tidak ada koneksi jaringan | **Dipatuhi** |
| Tidak menampilkan password/token/cookie/hash kredensial/nilai rahasia | **Dipatuhi** — login TERMASK, email hanya domain, nilai opsi hanya panjang+autoload, `user_pass` tidak pernah dibaca |
| Tidak menulis laporan ke folder database sumber | **Dipatuhi** |
| Tidak menganggap DB bersih karena pola tidak ditemukan | **Dipatuhi** — dinyatakan eksplisit di §7.7 dan §7.2 |
