# FASE 4C.3 — Audit Lanjutan Seluruh Tabel Database WordPress

- **Sumber:** `D:\backup-database-smk10november\` (4 dump SQL — **tidak diubah**; diperiksa ulang di akhir fase)
- **Laporan/artefak:** `C:\Users\sakha\forensik-smk10november\audit-backup\`
- **Tanggal:** 2026-10-14
- **Sifat:** READ-ONLY statis. Tanpa impor SQL, tanpa eksekusi, tanpa jaringan, tanpa perubahan akun/opsi/tabel.
- **Rujukan:** `FASE-4C-1-INVENTARIS-DATABASE.md`, `FASE-4C-2-AUDIT-AKUN-KONTEN.md` (dibaca dari **salinan lokal**; tidak ada URL yang dibuka).

---

## 1. Koreksi rekap akun Fase 4C.2

Penghitungan ulang dilakukan dari data sumber (`_4c6-users.tsv`, 163 baris) dengan skrip `rekap_koreksi.py`.

### 1.1 Klasifikasi DASAR — cocok dengan Lampiran A (tanpa perubahan)

| Situs | KUAT | VERIFIKASI | BELUM_CUKUP | Total |
|---|---|---|---|---|
| Tambun | 34 | 6 | 1 | **41** |
| Cibitung | 83 | 6 | 1 | **90** |
| Kebumen | 0 | 1 | 1 | **2** |
| Sragen | 22 | 7 | 1 | **30** |
| **Total** | **139** | **20** | **4** | **163** |

**Uji konsistensi: 139 + 20 + 4 = 163 = jumlah baris akun sumber ⇒ COCOK.** Subtotal per situs di Lampiran A juga cocok (34+6+1=41; 83+6+1=90; 0+1+1=2; 22+7+1=30).

### 1.2 Klasifikasi EFEKTIF setelah aturan klaster lintas-situs (§7.1 Fase 4C.2)

| Situs | Indikasi kuat | Perlu verifikasi | Belum cukup bukti | Total |
|---|---|---|---|---|
| Tambun | 39 | 1 | 1 | **41** |
| Cibitung | 84 | 5 | 1 | **90** |
| Kebumen | 1 | 0 | 1 | **2** |
| Sragen | 27 | 2 | 1 | **30** |
| **Total** | **151** | **8** | **4** | **163** |

**Uji konsistensi: 151 + 8 + 4 = 163 ⇒ COCOK.** 12 akun berpindah VERIFIKASI → indikasi kuat; 68 akun berada di klaster lintas-situs.

**8 akun yang masih perlu verifikasi manual:** Tambun 38; Cibitung 68, 72, 84, 88, 89; Sragen 29, 30.

### 1.3 Klarifikasi angka "122" (bukan kontradiksi)

Pada tabel ringkasan Fase 4C.2 muncul kolom **"Login berpola tool = 122"**. Itu **metrik berbeda**: hanya menghitung akun yang **login-nya** cocok pola tool (32/72/0/18). Klasifikasi **139** mencakup tambahan **17 akun** yang indikasinya dari **domain email IoC** atau **klaster lintas-situs**, bukan dari pola login. Keduanya benar; jangan disamakan.

> **Tidak ada perubahan pada berkas laporan Fase 4C.2.** Koreksi ini hanya dicatat di sini, sesuai permintaan.

---

## 2. Inventarisasi seluruh tabel per database

Nama tabel diambil dari **`CREATE TABLE` di masing-masing dump** (bukan diasumsikan), karena tabel tanpa baris tidak muncul sebagai `INSERT`.

| Situs | `CREATE TABLE` | Berisi data | Ada tapi **0 baris** | Baris diparse | Gagal |
|---|---|---|---|---|---|
| Tambun (`smk_`) | 132 | 39 | 93 | 14.362 | **0** |
| Cibitung (`cbt_`) | 131 | 38 | 93 | 16.455 | **0** |
| Kebumen (`kbm_`) | 138 | 39 | 99 | 15.030 | **0** |
| Sragen (`srg_`) | 132 | 39 | 93 | 14.052 | **0** |
| **Total** | **533** | **155** | **378** | **59.899** | **0** |

**633 tabel × situs diperiksa strukturnya; 59.899 baris berhasil diparse; 0 baris gagal.**

### 2.1 Tabel inti WordPress

| Tabel | Tambun | Cibitung | Kebumen | Sragen | Catatan |
|---|---|---|---|---|---|
| `users` | 41 | 90 | 2 | 30 | diaudit Fase 4C.2 |
| `usermeta` | 573 | 1.202 | 50 | 438 | diaudit Fase 4C.2 |
| `posts` | 378 | 681 | 642 | 284 | diaudit Fase 4C.2 |
| `postmeta` | 8.213 | 9.361 | 9.160 | 8.167 | diaudit Fase 4C.2 |
| `options` | **346** | **307** | **329** | **322** | §3 |
| `terms` | 9 | 9 | 16 | 9 | §4 |
| `term_taxonomy` | 9 | 9 | 16 | 9 | §4 |
| `term_relationships` | 27 | 31 | 49 | 27 | §4 |
| `termmeta` | ada, **0 baris** | ada, **0 baris** | **4** | ada, **0 baris** | — |
| `comments` | ada, **0 baris** | ada, **0 baris** | ada, **0 baris** | ada, **0 baris** | §4 |
| `commentmeta` | ada, **0 baris** | ada, **0 baris** | ada, **0 baris** | ada, **0 baris** | §4 |
| `links` | ada, **0 baris** | ada, **0 baris** | ada, **0 baris** | ada, **0 baris** | §4 |

### 2.2 Keluarga tabel plugin yang berisi data (identik di keempat situs kecuali yang ditandai)

| Keluarga (prefiks) | Perkiraan pemilik | Berisi data | Catatan risiko |
|---|---|---|---|
| `mailpoet_*` | MailPoet (newsletter) | 10 tabel | **berisi data pelanggan/email** → jangan migrasikan mentah |
| `wc_*`, `woocommerce_*` | WooCommerce | 12 tabel | termasuk **`woocommerce_sessions`** (sesi) → jangan dimigrasikan |
| `actionscheduler_*` | Action Scheduler | 3 tabel | antrean pekerjaan → dibuat ulang |
| `e_*`, `elementor_library*` | Elementor | 2 tabel | konfigurasi builder |
| `jet_*` | JetEngine/JetSmartFilters | 1 tabel data | `jet_taxonomies` |
| `gla_*` | Google Listings & Ads | 1 tabel (**4.230 baris**!) | data referensi, bukan IoC |
| `jetpack_sync_queue` | Jetpack | 1 | antrean sinkronisasi |
| `cartflows_ca_cart_abandonment` | CartFlows | 1 | berisi **data checkout pelanggan** |
| `frmt_*`, `wpforms_*`, `wpf_*` | Forminator / WPForms / WP All Import | 3+ | data formulir |
| `hostinger_*` | Hostinger | 0 berisi | — |
| **`ahm_*`** | plugin **hanya ada di Kebumen** (8 tabel, semua 0 baris) | 0 | perlu identifikasi pemilik |
| **`snippets`** | **Code Snippets** (menyimpan kode PHP) | **Tambun 4, Sragen 4** | diperiksa khusus → §5.2 |
| `woof_*`, `wpfm_*` | WooCommerce filter / WP File Manager | 0 berisi | — |

> Catatan metodologi: heuristik penamaan saya menandai 429 entri tabel sebagai "tidak dikenal", tetapi seluruhnya **dapat dikenali dari prefiks plugin** di atas. Yang benar-benar belum jelas pemiliknya hanya **`ahm_*`** (Kebumen) dan 2 tabel `frmt_/wpf_` tertentu.

---

## 3. Temuan tabel `options`

Total **1.304 baris** opsi (346/307/329/322).

### 3.1 Bentuk nilai & validasi serialisasi

| Bentuk | Tambun | Cibitung | Kebumen | Sragen |
|---|---|---|---|---|
| angka | 98 | 93 | 99 | 96 |
| teks | 96 | 78 | 80 | 83 |
| **serialized (valid)** | **78** | **73** | **89** | **78** |
| kosong | 62 | 52 | 49 | 53 |
| token_mirip | 8 | 5 | 5 | 8 |
| json_objek | 3 | 3 | 3 | 3 |
| **serialized!INVALID** | **1** | **3** | **4** | **1** |

**Validasi serialisasi dilakukan dengan parser TEKS murni** (`val_ser`) yang **tidak pernah memanggil `unserialize()`** dan **tidak menginstansiasi objek apa pun** — hanya memeriksa struktur, tipe, dan **kecocokan panjang string yang dideklarasikan**. JSON diperiksa dengan `json.loads` (aman).

**9 nilai serialized tidak lolos validasi** — semuanya berjenis **cache/transient/plugin-cache**:
`elementskit-lite__banner_data` (4 situs), `_site_transient_wp_plugin_dependencies_plugin_data` (Cibitung, Kebumen), `_elementor_home_screen_data` (Cibitung, Kebumen), `astra_partials_config_cache` (Kebumen).

**Penilaian (penting, jangan dilebih-lebihkan):** pola kerusakannya adalah **escape ganda** (tanda kutip dalam data tersimpan sebagai `\"` sehingga panjang yang dideklarasikan tidak lagi cocok). Karena **pola yang sama muncul pada transien yang ditulis WordPress inti sendiri** (bukan opsi milik penyerang), saya menilai ini **artefak escaping pada sisi plugin/proses, bukan sabotase**. Bukti byte-level: dump menulis `\"` (`od -c`), dan nilai yang bersih (mis. `_site_transient_theme_roots`) **lolos validasi** — jadi parser saya tidak salah menuduh. **Dampak rendah** (cache akan dibuat ulang), tetapi **tidak boleh dimigrasikan apa adanya**.

### 3.2 Opsi konfigurasi autentikasi (uji khusus)

| Opsi | Tambun | Cibitung | Kebumen | Sragen | Temuan |
|---|---|---|---|---|---|
| `users_can_register` | `0` | `0` | `0` | `0` | **Registrasi terbuka TIDAK aktif** di seluruh situs (temuan negatif) |
| `default_role` | `subscriber` | `subscriber` | `subscriber` | `subscriber` | **Peran bawaan bukan administrator** (temuan negatif penting) |
| `wp_user_roles` | serialized valid | valid | valid | valid | tidak ada peran tambahan |

⇒ Penyerang **tidak** menyalakan pendaftaran publik dan **tidak** mengubah peran bawaan; akun admin dibuat lewat jalur lain (tool `wp2shell`), konsisten dengan Fase 4C.2.

### 3.3 Indikator kompromi di `options`

| Situs | Opsi | Bukti | Kelas |
|---|---|---|---|
| **Tambun** | **`_codehidden`** | 10.831 B; **base64 → 8.120 B kode PHP**, terobfuskasi (`<?php goto …`); profil fungsi: **`file_put_contents`×2, `curl_exec`, `exec`, `header(`×9, `$_SERVER`×21, `str_rot13`, `md5`** | **IoC — TINGGI** |
| Tambun, Sragen | `sso_token` (67 B) | token endpoint login tanpa password | **IoC — TINGGI** |
| Tambun, Sragen | `_site_transient_update_themes`, `_site_transient_theme_roots` | memuat nama tema **`wp_edvl`** / **`wp_zqyq`** / `uvisyfyj` **hanya sebagai cache pemindaian tema** | **IoC — SEDANG** (bukan tema aktif) |
| Cibitung | `__mnx_versions` | mendaftarkan plugin palsu **`wp-wordfence-security`** | **IoC — TINGGI** |
| Tambun, Cibitung, Sragen | `fm_key` (25–28 B) | token manajer berkas (`wp-file-manager`) | **perlu tinjau — TINGGI** |
| Semua situs | `mnx_wp_auth` (JSON 71–139 B) | kunci otentikasi plugin "mnx" (pemilik belum jelas) | perlu tinjau |
| Kebumen | `__wpdm_enc_key` (259 B) | kunci enkripsi Download Manager | perlu tinjau (**regenerasi**) |
| Semua situs | `mailserver_pass` | kosong (nilai tidak ditampilkan) | perlu tinjau (rendah) |

**Konfirmasi penting:** nama tema jahat muncul **hanya di cache transient**, **bukan** di `template`/`stylesheet`/`current_theme` ⇒ tema webshell **ada di disk tetapi tidak pernah dipakai sebagai tema aktif**. (Sesuai temuan berkas Fase 3A.)

### 3.4 URL eksternal pada opsi — tidak ada yang mencurigakan

Host yang muncul: domain sekolah sendiri (siteurl/home), `downloads.wordpress.org`, `ps.w.org`, `s.w.org`, `library.elementor.com`, `api.wpmet.com`, `rpc.pingomatic.com`, `hostinger.com`, `wp-update.hostinger.io`, `hostingersite.com` (log Elementor). **Tidak ada** domain C2 Fase 3A (`*.expansor.xyz`, `*.clarivum.top`, `*.zephyrous.xyz`, `*.quiddity.top`), tidak ada shortener, tidak ada parameter mencurigakan. Opsi bernuansa login/redirect (`_elementor_pro_api_requests_lock`, `ai1wm_status`, dll.) tidak menemukan konfigurasi redirect berbahaya.

### 3.5 Pemeriksaan sistematis kode tersisip (`scan_hidden.py`)

Seluruh nilai opsi yang berbentuk base64/hex di keempat DB **didekode hanya di memori** (tidak pernah ditulis ke disk, tidak dieksekusi) dan diklasifikasi:

| Situs | Nilai terenkode berisi kode |
|---|---|
| Tambun | **1 → `_codehidden`: PHP** |
| Cibitung | 0 |
| Kebumen | 0 |
| Sragen | 0 |

⇒ **Hanya satu kode tersisip di seluruh tabel opsi keempat database**, dan itu di Tambun.

---

## 4. Temuan komentar dan taxonomy

Nama tabel aktual: `comments`, `commentmeta`, `links` **ada di keempat database** (dari `CREATE TABLE`) tetapi **0 baris**; `terms`, `term_taxonomy`, `term_relationships` berisi data; `termmeta` hanya berisi data di Kebumen (4 baris).

| Pemeriksaan | Hasil |
|---|---|
| Spam komentar | **Tidak dapat ada** — tabel `comments` kosong (0 baris) di keempat situs; artefak `_4c3-komentar.tsv` kosong |
| `commentmeta` | 0 baris di keempat situs |
| `links` (blogroll lama) | 0 baris di keempat situs |
| Istilah taxonomy mencurigakan (spam judi / URL di nama/slug) | **Tidak ditemukan** (0 temuan dari 33 istilah: 9/9/16/9) |
| `termmeta` ber-URL atau berkunci sensitif | **Tidak ditemukan** |
| Taksonomi yang ada | `category`, `nav_menu`, `elementor_library_type` (Kebumen menambah `post_tag`/lainnya) |
| Relasi tidak konsisten | Tidak diperiksa silang penuh terhadap `term_taxonomy.count`; jumlah relasi 27/31/49/27 wajar untuk menu + kategori |

**Keyakinan: TINGGI bahwa tidak ada IoC di jalur komentar/tautan** — karena tabelnya **kosong**, bukan karena pencarian pola tidak menemukan apa pun. (Perbedaan ini penting: kosong ≠ bersih karena tersaring.)

---

## 5. Temuan tabel plugin dan tabel lainnya

### 5.1 Antrean, sesi, log (prioritas tinggi untuk migrasi)

| Tabel | Tambun | Cibitung | Kebumen | Sragen | Penilaian |
|---|---|---|---|---|---|
| `woocommerce_sessions` | ada | ada | ada | ada | **sesi** → JANGAN dimigrasikan |
| `cartflows_ca_cart_abandonment` | 1 | 1 | 1 | 1 | berisi email/checkout → **jangan** dimigrasikan mentah |
| `actionscheduler_actions/logs/groups` | 3/5/2 | — | — | — | antrean pekerjaan → dibuat ulang |
| `jetpack_sync_queue` | ada | ada | ada | ada | antrean → regenerasi |
| `mailpoet_scheduled_tasks`, `mailpoet_migrations` | ada | ada | ada | ada | antrean plugin |
| `wc_rate_limits` | 0 baris | 0 baris | 0 baris | 0 baris | — |

### 5.2 `snippets` (Code Snippets — menyimpan KODE PHP) — diperiksa khusus

| Situs | Baris | Isi | Status | Penilaian |
|---|---|---|---|---|
| Tambun | 4 | "Make upload filenames lowercase", "Disable admin bar", "Allow smilies", "Current year" | **semua `active=0`** | **snippet CONTOH bawaan plugin** — bukan IoC |
| Sragen | 4 | sama | semua `active=0` | idem |
| Cibitung | tabel ada, **0 baris** | — | — | — |

Profil fungsi berbahaya pada kolom `code`: **tidak ada** (`eval`, `base64_decode`, `exec`, `curl_exec`, `$_POST` — nol). **False positive yang saya catat:** heuristik nama saya menandai "**Make upload** filenames lowercase" dan "Disable **admin** bar" sebagai mencurigakan hanya karena kata "upload"/"admin" — **itu FP**, dinyatakan di sini agar tidak dikutip sebagai temuan.

### 5.3 Tabel lain

- `gla_budget_recommendations` (4.230 baris × 4 situs = **16.920 baris**, 28 % dari seluruh data non-WP-inti) = data referensi Google Listings & Ads → aman, tetapi **tidak perlu dimigrasikan**.
- `ahm_*` (8 tabel, **hanya Kebumen**, semua 0 baris) → **pemilik belum jelas** (perlu verifikasi sebelum dianggap aman/hapus).
- `frmt_form_reports` (1 baris/situs), `wpf_*`, `wpforms_tasks_meta` → data plugin formulir/impor.
- `jet_taxonomies` (2 baris/situs) → definisi taxonomy JetEngine.
- `mailpoet_*` (10 tabel berisi) → **berisi data pelanggan**: perlakuan data pribadi.

---

## 6. Indikator kompromi — bukti dan tingkat keyakinan

| # | Indikator | Situs | Bukti | Keyakinan |
|---|---|---|---|---|
| **I-01** | **`_codehidden`**: payload PHP tersimpan di `wp_options` (base64 dari 8.120 B PHP terobfuskasi `goto`; kapabilitas `file_put_contents`, `curl_exec`, `exec`, `header`, `$_SERVER`) | **Tambun** | `_4c3-opsi-terenkode.tsv` + `probe_codehidden*.py` | **TINGGI** (kode PHP di opsi; tidak ada penjelasan sah) |
| **I-02** | 159 akun administrator dibuat 25-Jul→11-Sep-2026, pola login/email tool | semua | Fase 4C.2 (Lampiran A) | **TINGGI** |
| **I-03** | Opsi `sso_token` + `sso-loader.php` (endpoint login tanpa password) | Tambun, Sragen | Fase 4C.2 §5.3 + Fase 3A F-09 | **TINGGI** |
| **I-04** | Opsi `wp2shell_*` + item menu `x`/`proof` → `wp2shell.research/.com` | Tambun, Cibitung, Sragen (+ menu di Kebumen) | Fase 4C.2 §5.2/§5.1 | **TINGGI** |
| **I-05** | Plugin palsu `wp-wordfence-security` terdaftar di `__mnx_versions` | Cibitung | §3.3 | **TINGGI** |
| **I-06** | Nama tema webshell (`wp_edvl`, `wp_zqyq`, `uvisyfyj`) terekam di cache transient tema | Tambun, Sragen | §3.3 | **SEDANG** (tidak aktif) |
| **I-07** | `fm_key` (token manajer berkas) | Tambun, Cibitung, Sragen | §3.3 | **SEDANG** |
| **I-08** | 9 nilai serialized ter-escape ganda | semua | §3.1 | **RENDAH** (dinilai artefak escaping, bukan sabotase) |
| **I-09** | `ahm_*` (Kebumen) pemilik tak dikenal | Kebumen | §5.3 | **RENDAH** (belum cukup bukti) |

**Temuan negatif yang juga penting** (memperkecil cakupan tebakan): registrasi tertutup & `default_role=subscriber` (semua situs); tidak ada IoC di `comments`/`commentmeta`/`links` (kosong); tidak ada spam/judi di taksonomi; tidak ada URL C2 di opsi; tidak ada kode berbahaya di `snippets`; deteksi iframe/spam di konten = 0 (Fase 4C.2).

---

## 7. Keterbatasan parser dan data yang belum diperiksa

1. **Parser** Fase 4C.2 (L1 regex → L1.5 `csv` → L2/L3 state-machine) dipakai ulang; **0 baris gagal** dari 59.899 baris. Namun:
   - **Tabel tanpa baris tidak terlihat lewat `INSERT`** — itu sebabnya inventaris diambil dari `CREATE TABLE`; baris "0" berarti **tidak ada `INSERT`**, yang untuk dump phpMyAdmin berarti tabel kosong (bukan pasti; dump bisa saja mengecualikan tabel kosong).
   - **Dua bug saya perbaiki di fase ini**: (a) deteksi serialisasi/JSON gagal karena kutip luar & escape belum dilepas (`cron` 3.136 B terbaca "teks") — setelah diperbaiki `cron`/`active_plugins` terdeteksi **serialized**; (b) probe awal hanya memeriksa blok `options` pertama dan salah asumsi jumlah kolom. Semua angka di laporan ini berasal dari versi terkoreksi.
2. **Validasi serialisasi tidak dapat membedakan** "rusak karena escaping" dari "diubah dengan sengaja"; saya menyimpulkan artefak karena pola sama muncul pada transien inti WordPress.
3. **Unserialize penuh tidak dilakukan** (sesuai aturan) → isi `_elementor_data`, `woocommerce_sessions`, `mailpoet_*` hanya diperiksa pola/struktur, tidak dibaca semantik.
4. **`_codehidden` belum dianalisis semantik penuh.** Yang dilaporkan hanya profil kapabilitas; payload tidak ditampilkan/diekstrak. URL yang dibangun **tidak statis** (tidak ada base64/domain di dalamnya) ⇒ C2-nya dibentuk saat runtime (kemungkinan dari `$_SERVER`/konfigurasi lain). **Tidak dieksekusi.**
5. **Tidak ada korelasi dengan log akses HTTP** (di luar izin fase ini) ⇒ urutan waktu serangan tidak dipastikan.
6. **Tabel yang belum selesai diperiksa secara semantik:** `mailpoet_*` (10 tabel), `woocommerce_*` (12), `gla_*`, `frmt_*`, `wpf_*`, `jet_*`, `ahm_*` — baru pada tingkat **struktur, jumlah baris, dan pencarian pola IoC**, **belum** pembacaan isi baris.
7. **Pencarian IoC per-tabel** memakai pencocokan string pada byte region (bukan per-kolom); atribusi baris tersedia untuk tabel non-`posts`/`postmeta` (`_4c3-ioc.tsv`, 91 baris).
8. **Database Tidak dinyatakan bersih** — kesimpulan dibatasi pada tabel dan pola yang benar-benar diperiksa.

---

## 8. Klasifikasi data untuk migrasi

### (a) Kandidat migrasi — tampak diperlukan, tidak ditemukan indikator dalam pemeriksaan ini
`posts` (kecuali tipe/status terlarang), `postmeta` (kecuali pasangan menuju item menu `wp2shell.*`), `terms`, `term_taxonomy`, `term_relationships`, `termmeta`, dan `options` **daftar-putih** (siteurl, home, blogname, timezone, format tanggal, konfigurasi Elementor/tema yang sah). **Bukan jaminan bersih** — cakupan pemeriksaan seperti §7.

### (b) Perlu verifikasi sebelum dipakai
`_codehidden` (Tambun — **jangan** dipindahkan sebelum dipahami), `fm_key`, `mnx_wp_auth`, `__mnx_versions`, `__wpdm_enc_key` (Kebumen), `mailserver_pass`, tabel `ahm_*` (Kebumen), `snippets` (meski tampak contoh), 8 akun yang belum terverifikasi (§1.2), 1 post `trash` (Kebumen).

### (c) JANGAN dimigrasikan sebelum dibersihkan (mengandung IoC atau rahasia autentikasi lama)
- **`options`:** `_codehidden`; `sso_token` (Tambun, Sragen); `wp2shell_*` (3 situs); 9 nilai serialized INVALID; `active_plugins` & `__mnx_versions` (memuat plugin palsu/tak sah); seluruh SALT/kunci (`wp_user_roles` boleh, tetapi kunci `wp-config.php` lama **harus** diganti).
- **`users`/`usermeta`:** 151–159 akun admin tak sah + `session_tokens`/`capabilities` terkait. **Migrasikan akun hanya dari daftar bersih yang dibuat ulang.**
- **`posts`/`postmeta`:** item menu & changeset `x`/`proof` → `wp2shell.*`.
- **`usermeta`:** baris dengan jejak `wp2shell`.

### (d) Tidak perlu dimigrasikan (akan dibuat ulang / data sementara)
`woocommerce_sessions`, `actionscheduler_*` (actions/logs/groups), `jetpack_sync_queue`, `mailpoet_scheduled_tasks`, `mailpoet_migrations`, seluruh `_transient*`/`_site_transient*`, `_elementor_*cache*`, `gla_budget_recommendations` (4.230/situs), `wc_admin_notes`/`wc_admin_note_actions`, `wc_rate_limits`, `wc_category_lookup`/`wc_product_meta_lookup`/`wc_tax_rate_classes` (dibangun ulang), `frmt_form_reports`, 378 tabel kosong.

---

## 9. Rekomendasi tahap persiapan migrasi

1. **Blokir lebih dulu, migrasi kemudian** — sebelum migrasi, pastikan: (a) `_codehidden` dipahami/dinetralkan, (b) `sso_token` & `sso-loader.php` dicabut, (c) seluruh kredensial/SALT diganti, (d) kredensial DB lama dianggap bocor.
2. **Bersihkan I-01 (`_codehidden`)** sebagai prioritas **tertinggi** — ini backdoor **di dalam database**; menghapus berkas webshell saja tidak cukup.
3. **Audit menyeluruh tabel `snippets`** pada instalasi lama sebelum ekspor: ekspor hanya snippet yang `active=1` dan terverifikasi.
4. **Ekspor data selektif per tabel** dengan daftar-putih kolom; **jangan** `mysqldump` seluruh DB lalu impor (itu akan membawa IoC + kredensial lama).
5. **Data pribadi** (`mailpoet_subscribers`, `cartflows_ca_cart_abandonment`, `woocommerce_*`) → tinjau kewajiban privasi sebelum memindahkan.
6. **Bangun ulang** tabel sesi/antrean/cache di instalasi baru (jangan impor).
7. **Verifikasi pasca-migrasi** (wajib): cari string `wp2shell`, `_codehidden`, `sso_token`, `fm_key`, `wp_edvl`, `wp_zqyq`, `uvisyfyj`, `wp-wordfence` di seluruh DB baru — hasil harus **0**.
8. **Perbaiki akar masalah escaping serialisasi** (plugin/proses yang membuat `\")` agar tidak terulang di instalasi baru.
9. Setelah migrasi: pantau integritas berkas, aktifkan 2FA, batasi login, dan periksa ulang akun secara berkala.

---

## 10. Kepatuhan aturan

| Aturan | Status |
|---|---|
| Tidak mengimpor SQL ke DB lokal/produksi | **Dipatuhi** — tidak ada koneksi DB sama sekali |
| Tidak menjalankan SQL/PHP/JS/plugin/tema/payload dari backup | **Dipatuhi** — payload `_codehidden` **didekode di memori**, tidak ditulis/dieksekusi |
| Tidak mengakses situs produksi / jaringan | **Dipatuhi** |
| Tidak mengubah/menghapus/memindahkan/menimpa file sumber | **Dipatuhi** — 4 berkas, mtime asli, 0 perubahan (diperiksa ulang di akhir) |
| Tidak mengubah akun/konten/opsi/tabel | **Dipatuhi** |
| Tidak menampilkan password/hash/token/cookie/salt/API key | **Dipatuhi** — nilai opsi/token hanya dilaporkan sebagai **panjang + bentuk**; email hanya domain; IP disamarkan; payload tidak ditampilkan |
| Melaporkan kegagalan parsing (jumlah & sebab) | **Dipatuhi** — 0 baris gagal dari 59.899; cacat parser saya sendiri dilaporkan di §7 |
| Tidak menyebut database bersih menyeluruh | **Dipatuhi** — §7.8 |

---

## Ringkasan akhir fase

**Diperiksa:**

| Ukuran | Jumlah |
|---|---|
| Berkas dump | **4/4**, 100 % dibaca (342.655.406 B) |
| Tabel (dari `CREATE TABLE`) | **533 entri tabel × situs** (132/131/138/132 per DB) |
| Tabel berisi data | **155** |
| Tabel ada-tanpa-baris | **378** |
| **Baris berhasil diparse** | **59.899** |
| **Baris GAGAL diparse** | **0** |
| Nilai opsi diperiksa | **1.304** (nilai serialized divalidasi: 318 valid, **9 tidak valid**) |
| Nilai terenkode didekode (memori) | seluruh opsi berbentuk base64/hex → **1 berisi kode (PHP)** |
| Snippet diperiksa | **8** (4 Tambun + 4 Sragen), semua `active=0`, tanpa fungsi berbahaya |

**Tidak selesai / belum diperiksa sampai tingkat isi baris:** `mailpoet_*` (10 tabel berisi), `woocommerce_*` (12), `gla_*` (1), `frmt_*` (1), `wpf_*`/`wpforms_*` (3), `jet_*` (1), `cartflows_*` (1), `jetpack_sync_queue` (1), `ahm_*` (Kebumen, 0 baris) — tabel-tabel ini baru diperiksa **struktur, jumlah baris, dan pola IoC**, belum pembacaan isi. **Laporan ini tidak menyatakan database bersih.**

**Tidak ada remediasi atau migrasi yang dijalankan pada fase ini.**

**Laporan:** `C:\Users\sakha\forensik-smk10november\audit-backup\FASE-4C-3-AUDIT-TABEL-LANJUTAN.md`
**Artefak pendukung:** `_4c3-tabel.tsv`, `_4c3-tabel-lengkap.tsv`, `_4c3-options.tsv`, `_4c3-serial.tsv`, `_4c3-opsi-terenkode.tsv`, `_4c3-komentar.tsv`, `_4c3-taxonomy.tsv`, `_4c3-ioc.tsv`, `_4c3-snippets.tsv`, `_4c3-run.txt`
