# FASE 4C.2 — Audit Akun Administrator dan Konten WordPress

- **Lokasi sumber:** `D:\backup-database-smk10november\` (4 dump SQL — **tidak diubah**, terverifikasi)
- **Folder laporan:** `C:\Users\sakha\forensik-smk10november\audit-backup\`
- **Tanggal kerja:** 2026-10-14
- **Sifat:** READ-ONLY statis. **Tidak** ada impor SQL, eksekusi, koneksi jaringan/DB, atau perubahan akun/konten/opsi.
- **Catatan konteks:** laporan Fase 4C.1 dirujuk dari **salinan lokal** (`FASE-4C-1-INVENTARIS-DATABASE.md`). URL GitHub pada brief **tidak** dibuka — aturan melarang koneksi jaringan. Isi yang dipakai identik dengan yang sudah saya serahkan.

---

## 1. Ringkasan eksekutif

| Situs | Total user | **Administrator** | Dibuat ≥ 2026-07-25 | Login berpola tool | Jejak tool `wp2shell` di konten | Status |
|---|---|---|---|---|---|---|
| **Tambun** (`smk_`) | 41 | **41 (100 %)** | **40** | 32 | **2 item menu** (`x`, `proof`) | **Terkompromi — dampak luas** |
| **Cibitung** (`cbt_`) | 90 | **90 (100 %)** | **89** | 72 | **1 item menu** (`x`) | **Terkompromi — dampak terluas** |
| **Kebumen** (`kbm_`) | 2 | **2 (100 %)** | 1 | 0 | **2 item menu** (`x`, `proof`) | **Terkompromi — dampak akun minimal, jejak tool ADA** |
| **Sragen** (`srg_`) | 30 | **30 (100 %)** | **29** | 18 | **2 item menu** (`x`, `proof`) | **Terkompromi — dampak luas** |
| **TOTAL** | **163** | **163** | **159** | **122** | **7 baris changeset** | — |

**Tiga kesimpulan utama:**

1. **Seluruh 163 akun di keempat situs berperan `administrator` — 100 %, tanpa satu pun peran lain.** Ini sendiri tidak wajar: tidak ada editor/penulis/kontributor/pelanggan sama sekali. 159 akun dibuat dalam jendela **25-Jul → 11-Sep-2026**.
2. **Ditemukan "kartu nama" tool di keempat situs**: item menu bernama **`x`** dan **`proof`** yang menunjuk ke `https://wp2shell.research` dan `https://wp2shell.com`, tersimpan di `customize_changeset` (status `publish`) dan sebagian sudah menjadi baris `wp_postmeta` (`_menu_item_url`). **Kebumen punya jejak ini walaupun tidak punya ledakan akun.**
3. **Konten publik keempat situs tidak memuat spam SEO/judi maupun tautan C2.** 0 baris kata kunci spam, 0 host mencurigakan (TLD berisiko/shortener/mirip-domain/IP langsung), semua `<iframe` menunjuk host sah. ⇒ **Spam SEO insiden ini tidak disebarkan lewat basis data**, melainkan lewat berkas (Fase 3A: `wp-slgnup.gz` SEO-cloaking, cluster doorway `prestasi/` di Kebumen).

---

## 2. Metode, cakupan, dan batasan

### 2.1 Parser (tugas D)
Dibangun parser **berlapis**, setiap lapisan diverifikasi lewat **uji jumlah kolom**:

| Lapis | Teknik | Peran |
|---|---|---|
| L1 | regex sadar-kutip-tunggal (gaya mysqldump) | mayoritas baris |
| L1.5 | modul `csv` (C-level): `quotechar='` , `escapechar=\` , `doublequote=True` | komut di dalam string, escape `\'` |
| L2 | state-machine Python per baris | baris yang gagal L1/L1.5 |
| L3 | state-machine atas seluruh region | bila L2 gagal (tidak terjadi di fase ini) |

**Cacat yang saya temukan dan perbaiki sendiri selama fase ini (dicatat agar tidak disembunyikan):**

| # | Cacat | Dampak | Perbaikan & bukti |
|---|---|---|---|
| 1 | Batas region INSERT ikut menyapu `CREATE TABLE` tabel berikutnya | Fragmen DDL (`varchar(250)`, `int(11)`) terbaca sebagai "baris" `(250)`, `(11)`; `smk_users` terbaca **53 baris**, padahal 41 | Penanda batas + `bisect` → **41 baris** (`_4c2-parser-test*.txt`) |
| 2 | Regex jalur cepat memperlakukan `"` sebagai pembatas string, padahal `post_content` penuh `"` di dalam string kutip tunggal | Salah pemisahan field (`cbt_posts`: 86 & 59 field untuk 23 kolom) **dan** lambat | Jalur L1/L1.5 sadar-kutip-tunggal + state-machine; terverifikasi **tepat 23 field** |
| 3 | `data.find("CREATE TABLE")` memindai ~80 MB sampai EOF untuk setiap blok | **135 s dari 186 s** waktu parse | Satu lintasan penanda + `bisect`: **186 s → 42 s** per situs |
| 4 | Agregator laporan memakai indeks kolom salah (judul dibaca sebagai jenis temuan) | Tabel ringkasan salah | Diperbaiki; angka pada laporan ini berasal dari versi terkoreksi |

**Bukti uji parser (record mentah)** — `_4c2-parser-test.txt`: `smk_users` 41 baris (10 kolom) ✓, `smk_usermeta` 573 baris (4 kolom) ✓, `smk_posts` 71 baris (23 kolom) ✓, `smk_postmeta` 75 baris (4 kolom) ✓ — semua **COCOK**.

### 2.2 Cakupan & statistik parser (hasil final)

| Situs | Tabel | Blok INSERT | Baris L1 | Dipulihkan L1.5/L2 | **GAGAL** | Region dipindai ulang |
|---|---|---|---|---|---|---|
| Tambun | `smk_users` / `smk_usermeta` / `smk_posts` / `smk_postmeta` | 1 / 1 / 18 / 1360 | 41 / 568 / 266 / 7213 | 0 / 5 / 112 / 1000 | **0** | 0 |
| Cibitung | `cbt_*` | 1 / 1 / 37 / 1451 | 90 / 1197 / 437 / 8267 | 0 / 5 / 244 / 1094 | **0** | 0 |
| Kebumen | `kbm_*` | 1 / 1 / 66 / 1562 | 2 / 48 / 371 / 7990 | 0 / 2 / 271 / 1170 | **0** | 0 |
| Sragen | `srg_*` | 1 / 1 / 11 / 1377 | 30 / 431 / 206 / 7160 | 0 / 7 / 78 / 1007 | **0** | 0 |
| **Total** | | **5.890 blok** | **37.311** | **3.761** | **0** | **0** |

**Baris berhasil diparse: 39.312 baris, 0 gagal.** Sesuai aturan D.4, jumlah blok INSERT (5.890) **tidak** disamakan dengan jumlah baris (39.312) maupun jumlah post.

### 2.3 Batasan (dinyatakan tegas)
1. **Hanya 4 tabel** yang diaudit: `users`, `usermeta`, `posts`, `postmeta`. Setiap DB memiliki ±39 tabel berisi data; `options`, `comments`, `commentmeta`, `terms`, `termmeta`, `links` **tidak** diparse pada fase ini (sebagian `options` sudah dicakup Fase 4C.1). **Audit ini TIDAK menyeluruh dan tidak boleh disebut begitu.**
2. **Atribusi per-baris `postmeta` ke post induk tidak selalu berhasil**: sebagian `_menu_item_url` merujuk `post_id` yang tidak ada di tabel `posts` (mis. `1707223188`) — kemungkinan item menu yang hanya hidup di changeset Customizer lalu di-*orphan*. Dilaporkan apa adanya, **belum terkonfirmasi**.
3. **Tidak ada impor/eksekusi.** Semua temuan berasal dari pembacaan teks; serialisasi PHP tidak di-*unserialize* penuh (hanya pola yang dibaca).
4. **Tidak ada log akses HTTP.** Urutan/kronologi serangan tidak dapat dipastikan.
5. **Detektor saya sendiri punya FP yang terbukti** (lihat §8): `SHORTCODE` dan `KONTEN_TERSEMBUNYI` **tidak boleh** dipakai sebagai temuan tanpa klasifikasi manual — sudah saya klasifikasi di §5.
6. **Nilai sensitif tidak dibaca/ditampilkan**: `user_pass` tidak pernah disimpan, login **TERMASK**, email hanya **domain**, URL **tanpa query-string**.

---

## 3. Tabel temuan akun administrator per situs

Semua akun di bawah **berperan `administrator`** (tidak ada peran lain di keempat situs).

| Situs | Total | Dibuat 2025-10 (pra-jendela) | 2026-07 | 2026-08 | 2026-09 | ≥ 2026-07-25 |
|---|---|---|---|---|---|---|
| Tambun | 41 | 1 | 19 | 19 | 2 | **40** |
| Cibitung | 90 | 1 | 54 | 35 | — | **89** |
| Kebumen | 2 | 1 | — | 1 | — | **1** |
| Sragen | 30 | 1 | 7 | 18 | 4 | **29** |
| **Total** | **163** | **4** | **80** | **73** | **6** | **159** |

**Registrasi berkelompok dalam hitungan detik** (contoh Cibitung: id 9–13 pada 04:04:53–04:05:24) ⇒ **pembuatan akun otomatis**, bukan kerja manual.

**Akun pra-jendela (2025-10-30 09:08:29, 1 per situs)**: timestamp **identik di keempat DB** ⇒ keempat situs diprovisi dari templat yang sama pada tanggal itu. **Tidak otomatis sah** (aturan A.6) — tetap masuk daftar verifikasi.

### 3.1 Domain email akun baru (indikator kuat)

| Domain email | Tambun | Cibitung | Kebumen | Sragen | Sifat |
|---|---|---|---|---|---|
| `wp2shell.invalid` | 17 | 45 | — | 8 | **IoС** (nama tool) |
| `wp2shell.local` / `wp2shell.research` | 1 | 5 | — | 1 | **IoС** |
| `Bunk.invalid` / `bunk.invalid` | 10 | 6 | — | 6 | **IoС** (keluarga tool) |
| `nx.invalid` | 3 | 8 | — | 2 | **IoС** |
| `Fxshell.invalid` | 1 | — | — | 1 | **IoС** |
| **`shellcode.lol`** | — | **2** | — | — | **IoС sangat kuat** |
| `rce.invalid`, `JF.invalid` | — | 2 | — | — | **IoС** |
| `svc.localhost` / `service.localhost` / `local.host` | 1 | 11 | — | 2 | **IoС** (non-routable) |
| **`goole.com`** (salah ketik `google`) | 1 | 1 | **1** | 1 | **IoС penghubung — ada di KEEMPAT situs** |
| `example.com` | — | 2 | — | 1 | perlu verifikasi |
| `wordpress.org` / `wordpress.com` | 1 | 3 | — | 2 | perlu verifikasi |
| `gmail.com` / `outlook.com` | 4 | 2 | — | 4 | perlu verifikasi (bisa staf asli) |
| domain sekolah resmi | 1 | **3** | 1 | 1 | **perlu verifikasi** (aturan A.6) |
| *(bukan email / kosong)* | 1 | — | — | 1 | **IoС** |

### 3.2 Prefiks login akun (TERMASK 3 karakter)

`wp2***` (72 akun), `wp_***` (14), `Nx_***` (13), `w2s***` (6), `Bun***`/`bun***` (16), `HI_***`, `rce***`, `bdk***`, `upg***`, `nul***`, `Fx_***`, `bbp***`, `ngx***`, `wpa***` — seluruhnya cocok dengan penamaan tool/webshell yang ditemukan di Fase 3A.
Prefiks `Suk***`, `aws***`, `haq***`, `sik***` muncul **dengan himpunan identik** di Tambun dan Sragen ⇒ kumpulan akun yang sama ditanam di dua situs.

---

## 4. Kandidat akun yang harus diverifikasi (identitas dimasking)

Klasifikasi sesuai tugas A.5. **Lampiran A** memuat seluruh **163 akun** dengan ID, login TERMASK, nama tampilan TERMASK, domain email, peran, dan tanggal registrasi.

### 4.1 Indikasi kompromi kuat — **139 akun**
Kriteria objektif (dapat dihitung ulang dari Lampiran A / `_4c6-users.tsv`): **domain email IoC** (§3.1) → **134 akun**, **atau** prefiks login berpola tool dengan domain email biasa → **5 akun** (`upg***` dan `nul***` pada `wordpress.com`; `wp2***` ×3 pada `example.com` — Cibitung 43/44/56/57, Sragen 9). Contoh bentuk:
| Situs | ID | Login | Domain email | Registrasi | Alasan |
|---|---|---|---|---|---|
| Cibitung | 9–13 | `wp2***` | `wp2shell.invalid` | 2026-07-25 | nama tool + email tool + registrasi beruntun detik |
| Cibitung | * | `wp_***` | `shellcode.lol` | 2026-07 | **domain `shellcode.lol`** |
| Tambun / Cibitung / Kebumen / Sragen | 39 / 90 / 2 / 26 | `adm***` | **`goole.com`** | **2026-08-13 14:40:21–14:40:23** | **keempat situs dalam 2 detik** — lihat §7.1 |
| Tambun | * | `Bun***` | `Bunk.invalid` | 2026-07/08 | keluarga tool |
| Sragen | * | `Nx_***` | `nx.invalid` | 2026-08 | keluarga tool |
| Cibitung | * | `Nx_***` | `svc.localhost` | 2026-08 | domain non-routable |
| Cibitung | 43, 44 | `upg***`, `nul***` | `wordpress.com` | 2026-07-28 | nama tool + registrasi 2 jam berurutan |
> Rincian per-akun ada di **Lampiran A** (kelas `KUAT`).

### 4.2 Perlu verifikasi manual — **20 akun**
Email pada domain yang **mungkin** sah (`gmail.com`, `outlook.com`, `wordpress.org/com`, `example.com`) **atau** domain sekolah resmi (termasuk **2 akun tambahan di Cibitung** di luar akun 2025-10-30), tetapi registrasinya **di dalam jendela serangan**. Tidak boleh dihapus tanpa konfirmasi pemilik situs.

**Pembaruan setelah analisis klaster (§7.1):** **12 dari 20** akun di kelas ini terbukti ikut dalam **klaster pembuatan akun serentak lintas-situs** (utama1 id 35/37/39/40/41; Cibitung id 90; Kebumen id 2; Sragen id 24/25/26/27/28) ⇒ statusnya **naik menjadi indikasi kuat**. Yang benar-benar masih perlu verifikasi manual tinggal **8 akun**: utama1 **38**; Cibitung **68, 72, 84, 88, 89**; Sragen **29, 30**.

### 4.3 Belum cukup bukti — 4 akun
Akun `adm***` tanggal **2025-10-30 09:08:29** (satu per situs, domain sekolah). Ciri wajar untuk akun awal pemasangan, **tetapi** wajib diverifikasi karena (a) keempatnya bertimestamp identik, (b) hanya melihat tanggal tidak cukup untuk membuktikan keaslian (aturan A.6).
> **Catatan penting tentang Kebumen:** akun `adm***` kedua (2026-08, `goole.com`) masuk kategori **4.1**, bukan 4.3.

---

## 5. Tabel temuan konten per post/metadata

Sumber: `_4c6-findings.tsv` (per post & postmeta, dengan post_id/tipe/status) dan `_4c6-spam.tsv`.

### 5.1 Temuan bernilai tinggi — item menu `wp2shell` (keyakinan **TINGGI**)

| Situs | Tabel | post_id | tipe/status | Judul item | URL tujuan | Bukti |
|---|---|---|---|---|---|---|
| Tambun | `smk_posts` | 1380 | `customize_changeset` / publish | `x` | `https://wp2shell.research` | JSON nav_menu_item |
| Tambun | `smk_posts` | 1456 | `customize_changeset` / publish | `proof` | `https://wp2shell.com` | JSON nav_menu_item |
| Tambun | `smk_postmeta` | 1707223188 | (post induk tidak ada di `posts`) | — | `https://wp2shell.research` | `_menu_item_url` + `_menu_item_orphaned` = **1785108673** |
| Cibitung | `cbt_posts` | 1524 | `customize_changeset` / publish | `x` | `https://wp2shell.research` | JSON nav_menu_item |
| Cibitung | `cbt_postmeta` | 1765455876 | (post induk tidak ada) | — | `https://wp2shell.research` | `_menu_item_url` |
| **Kebumen** | `kbm_posts` | 1521, 1641 | `customize_changeset` / publish | `x`, `proof` | `wp2shell.research`, `wp2shell.com` | JSON nav_menu_item |
| Sragen | `srg_posts` | 1335, 1396 | `customize_changeset` / publish | `x`, `proof` | `wp2shell.research`, `wp2shell.com` | JSON nav_menu_item |
| Sragen | `srg_postmeta` | 1717430079, 1872324117 | (post induk tidak ada) | — | `wp2shell.research`, `wp2shell.com` | `_menu_item_url` |

- Nilai `_menu_item_orphaned` = `1785108673` → **2026-07-27 06:31:13 WIB**, tepat di awal ledakan akun (25-Jul-2026). Ini **jangkar waktu serangan** dari basis data.
- Item bertanda `"status":"publish"` dan `"_invalid":false` ⇒ changeset **diterapkan**, bukan sekadar draf.
- **Kebumen: jejak hanya ada di `customize_changeset`** (belum menjadi baris `postmeta`) ⇒ niat item menu tercatat, materialisasi tidak. Ini menempatkan Kebumen **di jalur tool yang sama** meski DB-nya tidak punya `wp2shell_*` maupun ledakan akun.

### 5.2 Temuan lain

| Jenis | Tambun | Cibitung | Kebumen | Sragen | Keyakinan & alasan |
|---|---|---|---|---|---|
| `IFRAME` (post memuat iframe) | 48 | 170 | 188 | 34 | **Bukan temuan.** Seluruh host sah: domain sendiri, `maps.google.com`, `wa.me`, `www.w3.org`, `instagram`, `tiktok`, `youtube` |
| `HOST_NON_LEGIT` | 33 | 73 | 65 | 20 | **Sebagian besar FP** (lihat §8); isinya `example.invalid` (template), `github.com`, `example.com`; sisanya **2–4 baris/situs = `wp2shell.*` (IoС, §5.1)** |
| `KONTEN_TERSEMBUNYI` | 41 | 148 | 172 | 24 | **FP terbukti** — semuanya pola `visibility:hidden` dari iframe oEmbed bawaan WordPress (diverifikasi konteks) |
| `SHORTCODE` | 34 | 89 | 82 | 22 | **FP terbukti** — cocok pada `nav_menu_item[<id>]` (JSON changeset) & CSS; **bukan** shortcode nyata |
| `PARAMETER_MENCURIGAKAN` | **0** | **0** | **0** | **0** | Tidak ada `?cmd=`, `?g=admin`, `cupiUploader`, dsb. di dalam basis data |
| `META_URL` (metadata memuat URL) | 36 | 87 | 11 | 23 | Wajar: `_menu_item_url`, `_elementor_*`, `envato_tk_*`, `_oembed_*`. **2+2+… di antaranya `wp2shell.*`** |
| Kata kunci spam (judi/SEO) | **0** | **0** | **0** | **0** | **Tidak ada** di judul, slug, kutipan, isi, maupun metadata |
| `eval(` / `base64_decode` / `<script>` | 0 | 0 | 0 | 0 | Tidak ditemukan di konten |

**Sebaran jenis & status post (bukti cakupan publik/draf/revisi/attachment)** — lihat `_4c6-posttypes.tsv`:

| Situs | post_type terbanyak | post_status terbanyak |
|---|---|---|
| Tambun | attachment 120, revision 101, oembed_cache 45, customize_changeset 34, request 34, nav_menu_item 22, page 11 | inherit 221, publish 119, parse 34, draft 3, auto-draft 1 |
| Cibitung | attachment 153, revision 153, oembed_cache 149, customize_changeset 89, request 89, page 15 | inherit 306, publish 280, parse 89, draft 6 |
| Kebumen | revision 214, oembed_cache 172, attachment 122, customize_changeset 78, **post 13**, page 11 | inherit 336, publish 303, **trash 1**, auto-draft 2 |
| Sragen | attachment 107, revision 55, oembed_cache 29, nav_menu_item 22, customize_changeset 22, page 11 | inherit 162, publish 96, parse 22, draft 3, auto-draft 1 |

Temuan `wp2shell` berada pada **`customize_changeset` berstatus `publish`** — bukan di draf/trash, sehingga konsisten dengan perubahan yang benar-benar diterapkan. Kebumen satu-satunya yang punya **1 post di `trash`** (perlu dilihat saat verifikasi manual).

---

## 6. Daftar URL/domain mencurigakan (tidak dikunjungi)

| Domain | Di mana ditemukan | Jumlah | Penilaian |
|---|---|---|---|
| **`wp2shell.research`** | `_menu_item_url` + `customize_changeset` (4 situs) | 7 | **IoС kuat** — kartu nama tool |
| **`wp2shell.com`** | `_menu_item_url` + `customize_changeset` (Tambun, Kebumen, Sragen) | 5 | **IoС kuat** |
| **`wp2shell.invalid` / `.local` / `.research`** | domain email akun admin | 71 | **IoС kuat** |
| **`shellcode.lol`** | domain email akun admin (Cibitung) | 2 | **IoС sangat kuat** |
| **`Bunk.invalid` / `bunk.invalid` / `Fxshell.invalid` / `nx.invalid` / `rce.invalid` / `JF.invalid`** | domain email akun admin | 35 | **IoС kuat** |
| **`goole.com`** | domain email akun admin (**keempat situs**) | 4 | **IoС penghubung** |
| `local.host`, `svc.localhost`, `service.localhost` | domain email akun admin | 17 | **IoС** (non-routable) |
| `example.invalid` | konten changeset & menu | 171 | **BUKAN IoС** — placeholder template Skoola |
| `example.com` | domain email akun admin | 3 | perlu verifikasi |
| `github.com`, `instagram.com`, `tiktok.com`, `youtube.com`, `maps.google.com`, `wa.me`, `w3.org` | konten | bervariasi | **sah** |

**Tidak ditemukan** di basis data: domain C2 hasil Fase 3A (`*.expansor.xyz`, `*.clarivum.top`, `*.zephyrous.xyz`, `*.quiddity.top`) — konsisten, karena C2 itu berada di **berkas** webshell, bukan di konten DB. **Tidak ada** URL dengan parameter mencurigakan, tidak ada shortener, tidak ada TLD berisiko, tidak ada IP langsung, tidak ada IDN/punycode, tidak ada domain mirip-domain-sendiri.

---

## 7. Korelasi dengan temuan Fase 3A dan Fase 4A

| Indikator | Bukti DB (Fase 4C.2) | Bukti berkas (Fase 3A/4A) | Kesimpulan |
|---|---|---|---|
| **`sso_token`** | Ada di **Tambun + Sragen** (64 char) | `mu-plugins/sso-loader.php` ada di **root(Tambun) + Sragen**, tidak ada di Cibitung/Kebumen | **Korelasi sempurna (4/4)** — endpoint login tanpa password aktif di dua situs itu |
| **`wp-file-manager`** | Aktif di **Tambun, Cibitung, Sragen**; **tidak** di Kebumen | Direktori plugin `wp-file-manager` ada di root/Cibitung/Sragen; **tidak ada** di Kebumen | **Korelasi sempurna (4/4)** |
| **`wp2shell_*` (opsi)** | Tambun, Cibitung, Sragen (tidak di Kebumen) | — | Tool yang sama |
| **`wp2shell.research`/`.com` (menu)** | **Keempat situs** | — | ⇒ **Kebumen tetap terpapar tool** meski tanpa ledakan akun/opsi |
| **`goole.com` (email admin)** | **Keempat situs** | — | ⇒ indikasi **satu aktor/otomasi** yang sama |
| **Plugin webshell `wp-wordfence-security`** | **Tidak** terdaftar di `active_plugins` | RCE `proc.php`/`pcntl.php` di Cibitung; injeksi `pcntl` ke `hello-elementor/header.php` | Webshell dieksekusi **langsung via HTTP** — tidak perlu aktivasi plugin |
| **`cupiUploader`** | **0** di basis data | Hanya di `cibitung/wp-signup.php` (1 berkas di seluruh tree) | Payload upload **berbasis berkas**, tidak lewat DB |
| **SEO-cloaking & doorway** | **0** kata kunci spam di DB | `wp-slgnup.gz` (root) + cluster `prestasi/` (Kebumen) + `bb.php`/`urls.txt` | SEO spam **tidak di DB** |
| **phpFileManager / file manager** | `wp-file-manager` aktif 3 situs | `wp-admin/includes/admin.php` 84 KB (root) + 2 salinan; `archive.php` 999 KB (Sragen) | Jalur **file manager** memang digunakan |
| **`wp-compat`** | **Tidak** ada di `active_plugins` (Sragen) | **Direktori KOSONG**, mtime **2026-09-11 20:18** (dalam jendela serangan) | Lihat §8 — **belum terkonfirmasi** |

**Perbedaan tingkat dampak per situs (dari kombinasi DB + berkas):**

| Situs | Ledakan akun | `wp2shell_*` opsi | `sso_token` | `wp-file-manager` | Kelas artefak berkas (Fase 3A) |
|---|---|---|---|---|---|
| Tambun | 40 | ✔ | **✔** | ✔ | file inti ditimpa + RCE XOR + SEO-cloaking + sso-loader |
| Cibitung | **89** | ✔ | — | ✔ | injeksi **berkas inti** (`wp-signup.php`, `class-wp-query.php`) + pcntl RCE |
| Kebumen | **1** | — | — | **—** | **hanya cluster SEO doorway** (tanpa webshell RCE) |
| Sragen | 29 | ✔ | **✔** | ✔ | phpFileManager (password kosong) + RCE XOR + 7 arsip dropper |

### 7.1 Bukti lintas-situs: akun dibuat BERSAMAAN di beberapa situs (keyakinan **TINGGI**)

`cluster_xsite.py` mengelompokkan registrasi akun pada jendela **180 detik**. Hasil: **26 klaster lintas-situs, mencakup 68 akun**.

| Waktu (WIB) | Sebaran | Situs terlibat | Akun |
|---|---|---|---|
| 2025-10-30 09:08:29 | **0 s** | **keempat situs** | `adm***` id=1 (provisioning awal) |
| 2026-07-26 13:10:57 | 23 s | Cibitung, Sragen, Tambun | `wp2***` |
| 2026-07-26 20:58:10 | 1 s | Cibitung, Tambun | `wp2***` |
| 2026-07-27 02:26:31 | 18 s | Cibitung, Sragen, Tambun | `wp2***` |
| 2026-07-30 14:33:57 | **2 s** | Cibitung, Sragen, Tambun | `wp2***` |
| 2026-08-01 10:01:37 | 14 s | Cibitung, Sragen, Tambun | `wp_***` (`local.host`) |
| 2026-08-05 09:11:47 | **2 s** | Cibitung, Sragen, Tambun | `wp2***` |
| 2026-08-08 13:47:34 | 1 s | Sragen, Tambun | `wpa***` |
| 2026-08-09 08:55:06 | 5 s | Sragen, Tambun | `Suk***` (`gmail.com`) |
| **2026-08-13 14:40:21** | **2 s** | **keempat situs** | `adm***` (`goole.com`) |
| 2026-09-11 03:20:57 | 7 s | Sragen, Tambun | `sik***` (`outlook.com`) |

Rekap kombinasi situs: Cibitung+Tambun **10×**, Cibitung+Sragen+Tambun **7×**, Sragen+Tambun **6×**, **keempat situs 2×**, Cibitung+Sragen 1×.

⇒ Akun dengan pola nama **sama** dibuat di situs **berbeda** dalam **1–2 detik**. Ini **tidak mungkin** hasil pekerjaan manual per situs ⇒ **satu operasi/otomasi yang sama menyerang beberapa situs sekaligus** (Tambun, Cibitung, Sragen berjalan paralel; Kebumen tersentuh pada klaster 4-situs).

**Konsekuensi penting:** 12 dari 20 akun yang di §4.2 masuk "perlu verifikasi manual" ternyata **berada di klaster lintas-situs** (utama1 id 35/37/39/40/41; Cibitung id 90; Kebumen id 2; Sragen id 24/25/26/27/28) ⇒ akun ber-email `gmail.com`/`outlook.com`/`goole.com` itu **naik status menjadi indikasi kuat**. Yang ** benar-benar** belum cukup bukti tinggal **8 akun**: utama1 id 38 (`wordpress.org`); Cibitung id 68 & 72 (domain sekolah), id 84 (`wordpress.com`), id 88 & 89 (`gmail.com`); Sragen id 29 & 30 (`wordpress.org`).

⇒ **Tetap tidak dapat disimpulkan** bahwa keempat situs masuk lewat **satu jalur** yang sama (sejalan dengan catatan Fase 3A). Namun **jejak tool yang sama (`wp2shell`) dan pola pembuatan akun yang terbukti serentak** menunjukkan **satu pelaku/otomasi yang sama**, dengan **dampak berbeda per situs**.

---

## 8. Temuan yang belum dapat dipastikan & false positive yang mungkin

**Belum terkonfirmasi:**
1. **`wp-compat` (Sragen)** — direktori plugin **kosong**, mtime **2026-09-11 20:18:28** (di dalam jendela serangan), tidak ada di `active_plugins`. Nama `wp-compat.php` pernah disebut sebagai artefak yang dicari. **Pemeriksaan Defender (read-only) menunjukkan TIDAK ada deteksi/karantina** pada direktori itu (hanya 2 deteksi lama pada `wp-blog-header.php` di folder laporan saya, ThreatID 2147761377) ⇒ pengosongan berasal dari sumber, bukan AV di mesin ini. Isi aslinya **tidak diketahui**; butuh log akses/berkas asli dari hosting.
2. **`post_id` yatim pada `_menu_item_url`** (`1707223188`, `1765455876`, `1717430079`, `1872324117` — dua di antaranya bertimestamp **tahun 2027 dan 2029** bila dibaca sebagai epoch) — kemungkinan ID internal Customizer/plugin, **bukan** ID post sekuensial. Belum terkonfirmasi.
3. **Akun domain sekolah** (4 akun; Cibitung punya **3**) — tidak dapat dipastikan sah. Aturan A.6 dipatuhi: domain sekolah **bukan** bukti keaslian.
4. **Akun `gmail.com`/`outlook.com`** (10 akun, semua dalam jendela serangan) — bisa staf asli, bisa pendaftaran penyerang.
5. **`1 post` berstatus `trash` di Kebumen** — isi belum ditujukan untuk analisis mendalam.

**False positive yang saya temukan pada detektor sendiri (dan sudah diklasifikasi):**
| Detektor | FP | Bukti |
|---|---|---|
| `KONTEN_TERSEMBUNYI` | **100 %** dari 385 baris | Konteks mentah: `style="position: absolute; visibility: hidden;"` pada `<iframe class="wp-embedded-content">` = **markup oEmbed bawaan WordPress** |
| `SHORTCODE` | Seluruh 227 baris | Cocok pada `nav_menu_item[1825926686]` (JSON changeset) dan potongan CSS |
| `HOST_NON_LEGIT` | Mayoritas | Isi utamanya `example.invalid` (placeholder template kit) & `github.com`; hanya `wp2shell.*` yang bernilai |
| "Akun baru" berbasis tanggal | Risiko | Cutoff 2026-01-01: kompromi yang membuat akun **sebelum** tanggal itu tidak akan tertandai |

---

## 9. Data: dipertahankan / ditinjau / jangan dimigrasikan sebelum verifikasi

| Kategori | Isi |
|---|---|
| **Dipertahankan (kandidat kuat)** | Konten sah situs sekolah: 1.985 post (halaman, berita, `attachment`, `elementor_library`), menu bawaan, pengaturan Elementor (`_elementor_*`), 2.263 baris `usermeta`, `options` non-IoC (termasuk `siteurl`/`home` yang konsisten). Judul konten yang terlihat sah mis. "Home", "Kontak", "LAPORAN REKAPITULASI … BOSP TAHAP 1 TAHUN 2026", "PURNAWIDYA … 2025/2026". |
| **Wajib ditinjau manual sebelum migrasi** | 4 akun `adm***` (2025-10-30); 10 akun `gmail/outlook`; 3 akun domain sekolah di Cibitung; 1 post `trash` (Kebumen); seluruh `_menu_item_url` yang menunjuk `wp2shell.*` (untuk dihapus dari menu); `session_tokens`/`ai1wm_secret_key` (kunci → **regenerasi**, jangan dimigrasikan apa adanya). |
| **JANGAN dimigrasikan (buang)** | **159 akun admin** ≥ 2026-07-25 beserta `usermeta`-nya (`capabilities`, `session_tokens`); opsi **`wp2shell_*`** (3 situs); opsi **`sso_token`** (Tambun, Sragen); item menu/changeset bertajuk `x`/`proof` → `wp2shell.*`; post `customize_changeset` yang memuat JSON `wp2shell`; sisa direktori `wp-compat` (Sragen); jejak berkas webshell/file-manager (Fase 3A) — **jangan** disalin ke pemasangan baru. |
| **Perlu regenerasi (jangan salin nilainya)** | SALT/kunci `wp-config.php`, `sso_token`, `ai1wm_secret_key`, token sesi, token langganan (`subscriber_link_tokens`, `unsubscribe_tokens`). |

---

## 10. Checklist tindak lanjut — WordPress bersih + migrasi selektif

**A. Persiapan (sebelum menyentuh produksi)**
- [ ] Ambil backup baru keempat DB **dan** salinan berkas, verifikasi hash-nya, simpan di luar webroot.
- [ ] Siapkan lingkungan staging bersih (WordPress versi resmi terbaru — root/Sragen 7.1.3, Cibitung 6.9.1, Kebumen 6.9.9 → **naikkan seragam**).
- [ ] Daftar IoC disetujui: akun (Lampiran A), opsi `wp2shell_*`, `sso_token`, item menu `wp2shell.*`, berkas Fase 3A.

**B. Pemasangan bersih**
- [ ] Pasang WordPress resmi (tanpa `mu-plugins/sso-loader.php`, tanpa `wp-file-manager` yang tidak disetujui, tanpa `wp-compat`).
- [ ] Ganti semua SALT/kunci dan kredensial DB; **jangan** pakai `wp-config.php` lama.
- [ ] Terapkan `.htaccess` yang benar (hapus "Allow from all" temuan Fase 3A).
- [ ] Pasang hanya plugin dari repositori resmi; aktifkan `auto-update` (Cibitung punya `disable-wordpress-updates` → **hapus**).

**C. Migrasi data selektif (urut)**
- [ ] Ekspor **hanya** tabel konten: `posts` (kecuali tipe/status terlarang), `postmeta` (kecuali pasangan menuju item menu `wp2shell.*`), `terms`, `termmeta`, `comments` yang sah, `options` **allowlist** (buang `wp2shell_*`, `sso_token`, `active_plugins`, `cron`, `session_tokens`).
- [ ] **Jangan** migrasikan `users`/`usermeta`: buat ulang akun yang lolos verifikasi §9 dari daftar bersih.
- [ ] Tambahkan kembali pengguna sah (peran **bukan** administrator untuk yang tidak perlu).
- [ ] Hapus/urutkan ulang menu; pastikan tidak ada `_menu_item_url` menuju `wp2shell.*`.
- [ ] Periksa `siteurl`/`home` setelah migrasi (tetap domain sekolah sendiri).

**D. Verifikasi pasca-migrasi**
- [ ] Jumlah & peran akun sesuai daftar bersih (**0 akun tak dikenal**); `wp2shell_*` & `sso_token` **tidak ada**.
- [ ] Uji `wp-admin` dari akun sah; uji front-end; uji login (endpoint SSO lama **mati**).
- [ ] Pindai berkas pemasangan baru (tidak ada webshell/file-manager tak sah).
- [ ] Uji apakah `wp2shell.research`/`.com` masih diterima (harus tidak ada tautan tersisa).
- [ ] Aktifkan 2FA, batasi login, pantau log akses & perubahan berkas (auto-update + integritas berkas).

**E. Konsistensi lintas situs**
- [ ] Perlakukan **seluruh 4 situs** sebagai terkompromi oleh tool yang sama (jejak `wp2shell` ada di semuanya) — termasuk **Kebumen**.
- [ ] Jangan menganggap "tanpa ledakan akun" = bersih.

---

## 11. Kepatuhan aturan

| Aturan | Status |
|---|---|
| Semua file SQL sumber tetap utuh | **Dipatuhi** — 4 berkas, mtime asli, 0 perubahan terverifikasi |
| Tidak menjalankan SQL/PHP/JS/plugin/tema/payload | **Dipatuhi** |
| Tidak ada koneksi jaringan / akses situs produksi | **Dipatuhi** (termasuk **tidak** membuka URL GitHub pada brief) |
| Tidak mengimpor DB lokal/remote | **Dipatuhi** |
| Tidak menghapus/memindahkan/mengganti nama berkas | **Dipatuhi** |
| Tidak mengubah akun/opsi/posts/postmeta/tabel | **Dipatuhi** |
| Laporan hanya di `audit-backup` | **Dipatuhi** |
| Tidak menaruh nilai sensitif di log/terminal/laporan | **Dipatuhi** — `user_pass` tidak pernah disimpan; login TERMASK; email hanya domain; URL tanpa query-string |
| Melaporkan keterbatasan bila parser/akses tidak memadai | **Dipatuhi** (§2.3, §8) — termasuk mengungkap 4 cacat parser sendiri |
| Tidak menyatakan database bersih | **Dipatuhi** — seluruh pernyataan dibatasi pada 4 tabel & pola yang diuji |

---

## Ringkasan berkas yang diperiksa

| Berkas | Ukuran | Dibaca | Tabel dicakup | Baris diparse | Gagal |
|---|---|---|---|---|---|
| `smkq8953_utama1.sql` (Tambun) | 80.550.250 B | 100 % | 4 tabel dari 39 | 8.263 | 0 |
| `smkq8953_cbitung.sql` (Cibitung) | 85.372.896 B | 100 % | 4 tabel dari 38 | 9.988 | 0 |
| `smkq8953_kebumen.sql` (Kebumen) | 95.834.894 B | 100 % | 4 tabel dari 39 | 9.851 | 0 |
| `smkq8953_sragen10.sql` (Sragen) | 80.897.366 B | 100 % | 4 tabel dari 39 | 8.910 | 0 |
| **Total** | **342.655.406 B** | 4/4 | **16 tabel (4/situs)** | **39.312** | **0** |

**Berkas/tabel yang BELUM selesai diperiksa (jujur, bukan klaim kosong):**
- **Tabel di luar 4 tabel di atas** untuk keempat DB: `options` (sebagian sudah dicakup Fase 4C.1), `comments`, `commentmeta`, `terms`, `termmeta`, `links`, dan sisanya (≈35 tabel/DB) — **belum** diaudit pada fase ini.
- **Serialisasi penuh** nilai `_elementor_data`/`_elementor_element_cache` (hanya pola URL/kata kunci yang diperiksa).
- **Urutan kronologis serangan** (butuh log akses HTTP — di luar izin fase ini).
- **Isi asli `wp-compat`** (direktori kosong di salinan).

**Lokasi laporan:** `C:\Users\sakha\forensik-smk10november\audit-backup\FASE-4C-2-AUDIT-AKUN-KONTEN.md`
**Artefak pendukung (di folder yang sama):** `_4c6-users.tsv`, `_4c6-findings.tsv`, `_4c6-spam.tsv`, `_4c6-posttypes.tsv`, `_4c6-parser-stats.tsv`, `_4c6-run.txt`, `_4c2-parser-test.txt`, `_4c2-parser-test-lain.txt`.

---

## Lampiran A — Daftar akun per situs (identitas dimasking)

Klasifikasi: **KUAT** = indikasi kompromi kuat (pola login tool / domain email IoC) · **VERIFIKASI** = perlu verifikasi manual · **BELUM_CUKUP** = belum cukup bukti.

**Rekap: KUAT 139 · VERIFIKASI 20 · BELUM_CUKUP 4 · total 163**

### A.1 utama1 — 41 akun

| ID | login (mask) | nama (mask) | domain email | peran | registrasi | kelas |
|---|---|---|---|---|---|---|
| 1 | `adm***` | `adm***` | smk10novembertambun.sch.id | administrator | 2025-10-30 09:08:29 | BELUM_CUKUP |
| 2 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-07-25 09:01:41 | KUAT |
| 3 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-07-25 14:03:18 | KUAT |
| 4 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-26 04:01:53 | KUAT |
| 5 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-07-26 09:09:58 | KUAT |
| 6 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 13:10:57 | KUAT |
| 7 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 20:43:32 | KUAT |
| 8 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 20:58:11 | KUAT |
| 9 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 21:14:12 | KUAT |
| 10 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 21:19:48 | KUAT |
| 11 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 21:58:12 | KUAT |
| 12 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 22:28:18 | KUAT |
| 13 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 22:48:20 | KUAT |
| 14 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 22:55:42 | KUAT |
| 15 | `wp2***` | `wp2***` | wp2shell.research | administrator | 2026-07-26 23:31:13 | KUAT |
| 16 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 23:45:52 | KUAT |
| 17 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-27 02:26:34 | KUAT |
| 18 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-07-28 02:27:47 | KUAT |
| 19 | `bun***` | `bun***` | bunk.invalid | administrator | 2026-07-30 12:37:40 | KUAT |
| 20 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-30 14:33:58 | KUAT |
| 21 | `wp_***` | `wp_***` | local.host | administrator | 2026-08-01 10:01:51 | KUAT |
| 22 | `bun***` | `bun***` | bunk.invalid | administrator | 2026-08-01 13:48:03 | KUAT |
| 23 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-02 13:32:55 | KUAT |
| 24 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-02 15:02:01 | KUAT |
| 25 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-02 19:16:42 | KUAT |
| 26 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-03 07:32:17 | KUAT |
| 27 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-03 22:28:58 | KUAT |
| 28 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-04 19:55:24 | KUAT |
| 29 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-05 09:11:49 | KUAT |
| 30 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-08 05:52:51 | KUAT |
| 31 | `wpa***` | `wpa***` | wp2shell.invalid | administrator | 2026-08-08 13:47:34 | KUAT |
| 32 | `Fx_***` | `fx_***` | Fxshell.invalid | administrator | 2026-08-08 16:07:44 | KUAT |
| 33 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-08-08 16:19:15 | KUAT |
| 34 | `bbp***` | `bbp***` | (bukan email) | administrator | 2026-08-09 04:58:50 | KUAT |
| 35 | `Suk***` | `suk***` | gmail.com | administrator | 2026-08-09 08:55:06 | VERIFIKASI |
| 36 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-08-10 01:36:02 | KUAT |
| 37 | `aws***` | `aws***` | gmail.com | administrator | 2026-08-10 05:58:16 | VERIFIKASI |
| 38 | `adm***` | `adm***` | wordpress.org | administrator | 2026-08-10 07:25:58 | VERIFIKASI |
| 39 | `adm***` | `***` | goole.com | administrator | 2026-08-13 14:40:21 | VERIFIKASI |
| 40 | `haq***` | `haq***` | outlook.com | administrator | 2026-09-01 05:51:52 | VERIFIKASI |
| 41 | `sik***` | `sik***` | outlook.com | administrator | 2026-09-11 03:21:04 | VERIFIKASI |

*Subtotal: {'BELUM_CUKUP': 1, 'KUAT': 34, 'VERIFIKASI': 6}*

### A.2 cbitung — 90 akun

| ID | login (mask) | nama (mask) | domain email | peran | registrasi | kelas |
|---|---|---|---|---|---|---|
| 1 | `adm***` | `adm***` | smk10novembercibitung.sch.id | administrator | 2025-10-30 09:08:29 | BELUM_CUKUP |
| 2 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-07-25 06:53:19 | KUAT |
| 3 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-07-25 10:50:19 | KUAT |
| 4 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-25 15:05:58 | KUAT |
| 5 | `w2s***` | `w2s***` | wp2shell.local | administrator | 2026-07-25 15:47:27 | KUAT |
| 6 | `HI_***` | `hi_***` | JF.invalid | administrator | 2026-07-25 18:07:04 | KUAT |
| 7 | `rce***` | `rce***` | rce.invalid | administrator | 2026-07-25 20:25:46 | KUAT |
| 8 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-26 04:02:17 | KUAT |
| 9 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 04:04:53 | KUAT |
| 10 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 04:04:56 | KUAT |
| 11 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 04:05:10 | KUAT |
| 12 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 04:05:14 | KUAT |
| 13 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 04:05:24 | KUAT |
| 14 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 05:03:50 | KUAT |
| 15 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 05:04:03 | KUAT |
| 16 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 05:04:08 | KUAT |
| 17 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 05:04:16 | KUAT |
| 18 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 05:04:20 | KUAT |
| 19 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 05:04:24 | KUAT |
| 20 | `w2s***` | `w2s***` | wp2shell.local | administrator | 2026-07-26 06:07:32 | KUAT |
| 21 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 08:49:18 | KUAT |
| 22 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 13:11:06 | KUAT |
| 23 | `wp_***` | `wp_***` | svc.localhost | administrator | 2026-07-26 17:21:06 | KUAT |
| 24 | `wp_***` | `wp_***` | svc.localhost | administrator | 2026-07-26 17:21:08 | KUAT |
| 25 | `wp2***` | `wp2***` | wp2shell.research | administrator | 2026-07-26 19:54:29 | KUAT |
| 26 | `bdk***` | `bdk***` | wp2shell.invalid | administrator | 2026-07-26 19:57:52 | KUAT |
| 27 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-26 20:34:15 | KUAT |
| 28 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-26 20:34:18 | KUAT |
| 29 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 20:58:10 | KUAT |
| 30 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 21:14:10 | KUAT |
| 31 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 21:19:49 | KUAT |
| 32 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 21:23:10 | KUAT |
| 33 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 21:58:11 | KUAT |
| 34 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 22:28:13 | KUAT |
| 35 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 22:28:21 | KUAT |
| 36 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 22:48:21 | KUAT |
| 37 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 22:55:38 | KUAT |
| 38 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 23:45:51 | KUAT |
| 39 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 23:46:00 | KUAT |
| 40 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-27 02:26:31 | KUAT |
| 41 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-27 03:40:43 | KUAT |
| 42 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-27 08:35:31 | KUAT |
| 43 | `upg***` | `upg***` | wordpress.com | administrator | 2026-07-28 01:20:37 | KUAT |
| 44 | `nul***` | `nul***` | wordpress.com | administrator | 2026-07-28 03:39:08 | KUAT |
| 45 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-28 05:28:38 | KUAT |
| 46 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-28 06:53:00 | KUAT |
| 47 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-28 08:38:08 | KUAT |
| 48 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-28 08:38:09 | KUAT |
| 49 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-28 10:11:21 | KUAT |
| 50 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-28 10:11:29 | KUAT |
| 51 | `wp_***` | `wp_***` | svc.localhost | administrator | 2026-07-30 05:08:46 | KUAT |
| 52 | `wp_***` | `wp_***` | svc.localhost | administrator | 2026-07-30 05:08:46 | KUAT |
| 53 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-30 08:51:18 | KUAT |
| 54 | `bun***` | `bun***` | bunk.invalid | administrator | 2026-07-30 12:36:56 | KUAT |
| 55 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-30 14:33:57 | KUAT |
| 56 | `wp2***` | `wp2***` | example.com | administrator | 2026-08-01 04:28:42 | KUAT |
| 57 | `wp2***` | `wp2***` | example.com | administrator | 2026-08-01 04:28:42 | KUAT |
| 58 | `wp_***` | `wp_***` | local.host | administrator | 2026-08-01 10:01:37 | KUAT |
| 59 | `wp_***` | `wp_***` | local.host | administrator | 2026-08-01 10:01:41 | KUAT |
| 60 | `bun***` | `bun***` | bunk.invalid | administrator | 2026-08-01 11:04:24 | KUAT |
| 61 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-01 14:45:19 | KUAT |
| 62 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-01 14:45:32 | KUAT |
| 63 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-01 15:09:18 | KUAT |
| 64 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-01 15:09:18 | KUAT |
| 65 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-01 15:22:32 | KUAT |
| 66 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-01 15:42:09 | KUAT |
| 67 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-02 17:47:53 | KUAT |
| 68 | `adm***` | `adm***` | smk10novembercibitung.sch.id | administrator | 2026-08-02 22:52:54 | VERIFIKASI |
| 69 | `w2s***` | `w2s***` | shellcode.lol | administrator | 2026-08-03 07:15:17 | KUAT |
| 70 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-03 07:32:12 | KUAT |
| 71 | `wp_***` | `wp_***` | service.localhost | administrator | 2026-08-03 08:34:54 | KUAT |
| 72 | `adm***` | `adm***` | smk10novembercibitung.sch.id | administrator | 2026-08-03 09:02:06 | VERIFIKASI |
| 73 | `wp_***` | `wp_***` | service.localhost | administrator | 2026-08-03 09:35:08 | KUAT |
| 74 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-03 14:09:53 | KUAT |
| 75 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-03 22:21:08 | KUAT |
| 76 | `w2s***` | `w2s***` | shellcode.lol | administrator | 2026-08-04 02:21:42 | KUAT |
| 77 | `w2s***` | `w2s***` | wp2shell.local | administrator | 2026-08-04 12:18:57 | KUAT |
| 78 | `w2s***` | `w2s***` | wp2shell.local | administrator | 2026-08-04 13:19:23 | KUAT |
| 79 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-04 19:55:16 | KUAT |
| 80 | `wp_***` | `wp_***` | local.host | administrator | 2026-08-05 03:54:48 | KUAT |
| 81 | `wp_***` | `wp_***` | local.host | administrator | 2026-08-05 03:56:36 | KUAT |
| 82 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-05 04:09:21 | KUAT |
| 83 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-05 09:11:47 | KUAT |
| 84 | `wor***` | `wor***` | wordpress.com | administrator | 2026-08-05 10:18:25 | VERIFIKASI |
| 85 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-05 11:21:32 | KUAT |
| 86 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-05 11:28:02 | KUAT |
| 87 | `wp_***` | `wp_***` | service.localhost | administrator | 2026-08-06 13:25:38 | KUAT |
| 88 | `adm***` | `adm***` | gmail.com | administrator | 2026-08-08 06:58:47 | VERIFIKASI |
| 89 | `web***` | `web***` | gmail.com | administrator | 2026-08-08 09:13:46 | VERIFIKASI |
| 90 | `adm***` | `***` | goole.com | administrator | 2026-08-13 14:40:22 | VERIFIKASI |

*Subtotal: {'BELUM_CUKUP': 1, 'KUAT': 83, 'VERIFIKASI': 6}*

### A.3 kebumen — 2 akun

| ID | login (mask) | nama (mask) | domain email | peran | registrasi | kelas |
|---|---|---|---|---|---|---|
| 1 | `adm***` | `adm***` | smk10novemberkebumen.sch.id | administrator | 2025-10-30 09:08:29 | BELUM_CUKUP |
| 2 | `adm***` | `***` | goole.com | administrator | 2026-08-13 14:40:23 | VERIFIKASI |

*Subtotal: {'BELUM_CUKUP': 1, 'VERIFIKASI': 1}*

### A.4 sragen10 — 30 akun

| ID | login (mask) | nama (mask) | domain email | peran | registrasi | kelas |
|---|---|---|---|---|---|---|
| 1 | `adm***` | `adm***` | smk10novembersragen.sch.id | administrator | 2025-10-30 09:08:29 | BELUM_CUKUP |
| 2 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-25 12:11:51 | KUAT |
| 3 | `Nx_***` | `nx_***` | nx.invalid | administrator | 2026-07-26 04:01:57 | KUAT |
| 4 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-26 13:11:20 | KUAT |
| 5 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-27 02:26:49 | KUAT |
| 6 | `wp2***` | `wp2***` | wp2shell.research | administrator | 2026-07-27 03:17:14 | KUAT |
| 7 | `bun***` | `bun***` | bunk.invalid | administrator | 2026-07-30 12:42:23 | KUAT |
| 8 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-07-30 14:33:59 | KUAT |
| 9 | `wp2***` | `wp2***` | example.com | administrator | 2026-08-01 06:59:18 | KUAT |
| 10 | `wp_***` | `wp_***` | local.host | administrator | 2026-08-01 10:01:39 | KUAT |
| 11 | `wp_***` | `wp_***` | local.host | administrator | 2026-08-01 10:01:42 | KUAT |
| 12 | `bun***` | `bun***` | bunk.invalid | administrator | 2026-08-01 12:30:57 | KUAT |
| 13 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-02 13:22:25 | KUAT |
| 14 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-02 14:16:52 | KUAT |
| 15 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-02 19:09:02 | KUAT |
| 16 | `Bun***` | `bun***` | Bunk.invalid | administrator | 2026-08-03 07:32:01 | KUAT |
| 17 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-03 22:21:05 | KUAT |
| 18 | `ngx***` | `ngx***` | wp2shell.invalid | administrator | 2026-08-05 09:04:35 | KUAT |
| 19 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-05 09:11:48 | KUAT |
| 20 | `wp2***` | `wp2***` | wp2shell.invalid | administrator | 2026-08-08 05:45:01 | KUAT |
| 21 | `wpa***` | `wpa***` | wp2shell.invalid | administrator | 2026-08-08 13:47:35 | KUAT |
| 22 | `Fx_***` | `fx_***` | Fxshell.invalid | administrator | 2026-08-08 16:06:38 | KUAT |
| 23 | `bbp***` | `bbp***` | (bukan email) | administrator | 2026-08-09 04:52:35 | KUAT |
| 24 | `Suk***` | `suk***` | gmail.com | administrator | 2026-08-09 08:55:11 | VERIFIKASI |
| 25 | `aws***` | `aws***` | gmail.com | administrator | 2026-08-10 05:58:12 | VERIFIKASI |
| 26 | `adm***` | `***` | goole.com | administrator | 2026-08-13 14:40:23 | VERIFIKASI |
| 27 | `haq***` | `haq***` | outlook.com | administrator | 2026-09-01 05:51:48 | VERIFIKASI |
| 28 | `sik***` | `sik***` | outlook.com | administrator | 2026-09-11 03:20:57 | VERIFIKASI |
| 29 | `roo***` | `roo***` | wordpress.org | administrator | 2026-09-11 07:21:04 | VERIFIKASI |
| 30 | `adm***` | `adm***` | wordpress.org | administrator | 2026-09-11 07:43:04 | VERIFIKASI |

*Subtotal: {'BELUM_CUKUP': 1, 'KUAT': 22, 'VERIFIKASI': 7}*

---

## Lampiran B — Statistik parser (tugas D.3)

`blok` = jumlah blok INSERT · `L1` = baris via jalur cepat · `dipulihkan` = baris via csv/state-machine · `GAGAL` = baris tidak berhasil diparse.

| Situs | Tabel | blok | L1 | dipulihkan | GAGAL | rescan |
|---|---|---|---|---|---|---|
| utama1 | smk_postmeta | 1360 | 7213 | 1000 | 0 | 0 |
| utama1 | smk_posts | 18 | 266 | 112 | 0 | 0 |
| utama1 | smk_usermeta | 1 | 568 | 5 | 0 | 0 |
| utama1 | smk_users | 1 | 41 | 0 | 0 | 0 |
| cbitung | cbt_postmeta | 1451 | 8267 | 1094 | 0 | 0 |
| cbitung | cbt_posts | 37 | 437 | 244 | 0 | 0 |
| cbitung | cbt_usermeta | 1 | 1197 | 5 | 0 | 0 |
| cbitung | cbt_users | 1 | 90 | 0 | 0 | 0 |
| kebumen | kbm_postmeta | 1562 | 7990 | 1170 | 0 | 0 |
| kebumen | kbm_posts | 66 | 371 | 271 | 0 | 0 |
| kebumen | kbm_usermeta | 1 | 48 | 2 | 0 | 0 |
| kebumen | kbm_users | 1 | 2 | 0 | 0 | 0 |
| sragen10 | srg_postmeta | 1377 | 7160 | 1007 | 0 | 0 |
| sragen10 | srg_posts | 11 | 206 | 78 | 0 | 0 |
| sragen10 | srg_usermeta | 1 | 431 | 7 | 0 | 0 |
| sragen10 | srg_users | 1 | 30 | 0 | 0 | 0 |

> Jumlah blok INSERT **tidak** sama dengan jumlah baris/post (aturan D.4).
