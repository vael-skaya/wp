# FASE 4A — Inventarisasi Backup WordPress (Read-Only)

- **Tanggal pemeriksaan:** 2026-10-13 (± 20:00–21:00 WIB)
- **Cakupan:** lokasi kandidat yang diperiksa secara **terbatas** (Downloads, Desktop, Documents, direktori `backup` yang diketahui, root `D:\`, direktori laporan forensik). **Tidak** ada pemindaian disk tanpa batas.
- **Sifat kerja:** READ-ONLY. Tidak ada arsip yang diekstrak, tidak ada file yang diubah/dipindah/dihapus/diganti nama, tidak ada dump SQL yang diimpor, tidak ada koneksi jaringan/hosting/cPanel, tidak ada kredensial/OTP yang diminta, tidak ada perubahan Def/konfigurasi sistem.
- **Aturan 7 dipatuhi:** tidak ada kata sandi/token/cookie/private key yang ditampilkan. Isi baris `INSERT` pada dump SQL **tidak** dicetak.
- **Aturan 10 dipatuhi:** tidak ada backup yang dinyatakan bersih. Klasifikasi di bawah berbasis **isi/struktur**, bukan nama saja.
- **Catatan mtime (aturan D):** waktu modifikasi filesystem **tidak** dipakai sebagai bukti tunggal kapan backup dibuat; tanggal internal arsip dipakai sebagai sumber sekunder.
- **Tidak ada audit isi ZIP/SQL** yang dilakukan pada tahap ini (sesuai instruksi: berhenti setelah inventarisasi).

---

## 1. Ringkasan

Ditemukan **satu kandidat backup pemulihan utama** dan **satu dump database** yang saling melengkapi untuk salah satu domain, serta tiga domain yang **belum memiliki dump database**:

| Ringkas | Temuan |
|---|---|
| Backup file situs utama | **`C:\Users\sakha\Downloads\cgi-bin.zip`** — 1.335.066.857 B (1,33 GB), **berisi keempat instalasi** (root/tambun + cibitung + kebumen + sragen), 43.968 anggota, tanpa kata sandi |
| Dump database | **`C:\Users\sakha\Downloads\smkq8953_utama1.sql`** — 80.550.250 B, 132 tabel berprefiks **`smk_`** |
| Bukti asal dump | **`C:\Users\sakha\Downloads\index.htm`** — halaman phpMyAdmin tersimpan: `brantas.iixcp.rumahweb.net / localhost / smkq8953_utama1 | phpMyAdmin 5.2.3` |
| Backup AI1WM di dalam arsip | **`wp-content/ai1wm-backups/smk10novembertambun.sch.id-20260109-012612-736.wpress`** — 442.722.278 B (**hanya untuk domain tambun**) |
| **Domain tanpa pasangan DB** | **cibitung, kebumen, sragen** — tidak ada dump SQL di cakupan yang diperiksa |
| Temuan penting | `cgi-bin.zip` **memuat seluruh artefak berbahaya** yang terverifikasi Fase 2/3A (24 dari 24 yang diperiksa ADA) ⇒ backup ini diambil **setelah** kompromi, **bukan** sumber pemulihan bersih tanpa pembersihan |

**Pemetaan domain #4 terpecahkan:** instalasi pada root `D:\smk10november` (mis. `wp-admin/`, `wp-includes/`, `wp-content/`, `wp-config.php`, `index.html` defacement) adalah **smk10novembertambun.sch.id**.

---

## 2. Lokasi yang diperiksa

| Lokasi | Hasil |
|---|---|
| `C:\Users\sakha\Downloads` (77 entri level-1) | **Ada kandidat utama**: `cgi-bin.zip`, `smkq8953_utama1.sql`, `index.htm`, `admin.php`, `error_log__1126ebc` |
| `C:\Users\sakha\Desktop` (9 entri) | Tidak ada backup WordPress (shortcut, `KeyFreeze`, `next update lms.txt`) |
| `C:\Users\sakha\Documents` (13 entri) | Tidak ada backup WordPress (dokumen pribadi/sekolah) |
| `C:\Users\sakha` (level-1) | Tidak ada backup WordPress (hanya file profil Windows) |
| `D:\` (level-1) | `sakaa.zip`, `goldencar.zip`, `goldencar-deployment.zip` (semua **bukan** WP sekolah — lihat §3), `BACKUP SYSTEM - JGN DIHAPUS\` (installer lama + image `win10-21h1_full_b1_s1_v1.tib` 21,5 GB), `backup.txt`, `backup-parse.txt` |
| `D:\downloads` (6 entri) | Tidak ada backup WP (`eoa.htm`, installer Firefox lama) |
| `D:\dokumen` (27 entri) | Tidak ada backup WP (docx/xlsx soal & jadwal sekolah) |
| `D:\backup` (5 entri) | **Bukan** backup: 5 berkas video `.mp4` (total ±3,7 GB) |
| `D:\duplicate`, `D:\dup` | Direktori kode (`api/`, `fe/`/`frontend/`) — bukan backup WP |
| `D:\BACKUP SYSTEM - JGN DIHAPUS` | Installer/utilitas lama (2019–2022) + image sistem `.tib` — bukan backup WP |
| `C:\Users\sakha\forensik-smk10november` | Direktori laporan forensik (dibuat pada fase sebelumnya) — bukan backup |
| `D:\smk10november` | **Arsip bukti yang sudah diketahui** (bukan kandidat backup pemulihan) — lihat §3.3 |

**Belum diperiksa (di luar cakupan atas, menunggu persetujuan):** `D:\ai`, `D:\ams`, `D:\sisko`, `D:\telegram-bot`, `D:\whatsapp-sticker-bot`, `D:\veylix`, `D:\skaya`, `D:\-skaya`, `D:\--skay`, `D:\filem`, `D:\locate`, `D:\F`, `D:\gabut`, `D:\belajar`, `D:\bls9-beta`, `D:\trias`, `D:\uji-parse`, `D:\My Documents`, `D:\goldencar*` (5 direktori), `D:\Folder Baru (2)`, `D:\gambar*`, `D:\musik`, `D:\fonts`, `D:\dev`, `D:\Recovery`, lokasi C: lain, kedalaman >3 pada Downloads/Desktop/Documents, media eksternal/cloud.

---

## 3. Daftar arsip & kandidat (ukuran, mtime, SHA-256)

### 3.1 Kandidat backup / dump (relevan)

| # | Path lengkap | Nama file | Ukuran (B) | mtime | SHA-256 | Dapat dibaca? | Perlu sandi? |
|---|---|---|---|---|---|---|---|
| 1 | `C:\Users\sakha\Downloads\` | `cgi-bin.zip` | 1.335.066.857 | 2026-10-12 11:20:54 | `bb6e832bdf549a37a6a736de97159d1beee53a1f31c567d53279c1df70d3175c` | **Ya** (43.968 anggota, 2.346,9 MB terbuka) | **Tidak** (0 anggota terenkripsi) |
| 2 | `C:\Users\sakha\Downloads\` | `smkq8953_utama1.sql` | 80.550.250 | 2026-10-13 19:40:56 | `04a7b303ac978e4cbd1dacac6a69a56b9742bbf01dad73d588d014ed82309506` | **Ya** (teks plain, **bukan** gzip) | n/a |
| 3 | `C:\Users\sakha\Downloads\` | `index.htm` | 276.044 | 2026-10-13 19:40 | `44eac3e2ee5912dee51c8d19b5ef8c357102ebaeba88ec8b32f1483338eecc6d` | Ya (HTML) | n/a |
| 4 | `C:\Users\sakha\Downloads\` | `admin.php` | 84.070 | 2026-10-13 12:30 | `7841b2f294c38d5213b0fac8256a766de2efb5d8eb8d60f7533549684a38be69` | Ya (PHP) | n/a |
| 5 | `C:\Users\sakha\Downloads\` | `error_log__1126ebc` | 1.771.807 | 2026-10-12 20:48 | `e93ec877e06612966cffd7c4bfe85de01c342805da4068688c20dccb00f5face` | Ya (teks) | n/a |

**Catatan penting untuk #4 dan #5 — keduanya BUKAN backup:**
- `admin.php` (84.070 B) **identik** (SHA-256 sama) dengan berkas inti WordPress yang **digantikan** oleh "PHP File manager ver 1.5" pada instalasi tambun di `D:\smk10november\wp-admin\includes\admin.php` (Fase 3A, F3A-1). ⇒ Ini **salinan artefak berbahaya**, bukan backup pemulihan.
- `error_log__1126ebc` (1.771.807 B) **identik** dengan `D:\smk10november\error_log`. ⇒ Salinan berkas bukti.

### 3.2 Kandidat besar yang **bukan** backup sekolah (diklasifikasi dari ISI, bukan nama)

| # | Path | Ukuran (B) | mtime | SHA-256 | Isi (bukti) | Kesimpulan |
|---|---|---|---|---|---|---|
| 6 | `D:\sakaa.zip` | 244.135.966 | 2026-05-13 23:49 | `901b2f03d8180af45fa41e67840a5a81dff1095100d94e0540c47d6571a39b09` | 11.180 anggota, prefix level-1 `sakaa/` berisi repo git (`sakaa/--aya/.git/…`); **0** anggota terkait WordPress | **Bukan** backup WordPress |
| 7 | `D:\goldencar.zip` | 268.383.144 | 2026-09-15 20:52 | `f83d86f197ad06ad18d00a2760d903e93dcc53bffb68c275ca432aaabbfd7709` | 20.911 anggota; `apps.golden-car.co.id`, `bandesrekayasadigital.com`, `.ftpquota`, `.well-known/acme-challenge` → backup akun hosting **lain** | **Bukan** backup 4 domain target |
| 8 | `D:\goldencar-deployment.zip` | 28.119.967 | 2026-09-25 15:58 | `5b2209b97112221e9714c4ec0e0f3f3da628e47125a1919e09501608b71e0748` | Subset proyek yang sama (`apps.golden-car.co.id` 14.266 anggota) | **Bukan** backup 4 domain target |

### 3.3 Arsip yang sudah diketahui ada di dalam bukti (bukan kandidat pemulihan)

Di dalam `D:\smk10november` terdapat 16 arsip: 7 arsip dropper di root Sragen (`item.zip`, `hiroshi.zip`, `wp-blog-header.zip`, `google006f33b90c2ea6f4.zip`, `shopsdetails.zip`, `goodsdetails.zip`, `index.zip`), 2 arsip file-manager (`themes/archive.zip`, `wp-includes/block-supports/archive.zip`), `themes/admin.zip`, `.zip` (berisi `.htaccess`), `wp-slgnup.gz`, dan 4 salinan `source-424.zip` (template kit). Semuanya sudah dianalisis pada Fase 2/3A dan **bukan** kandidat pemulihan.

### 3.4 Arsip lain di lokasi yang diperiksa (tidak relevan, untuk kelengkapan)

Perkakas pengembangan/font di `Downloads`: `php-8.4.22-Win32-vs17-x64.zip` (34,9 MB), `ngrok-v3-stable-windows-amd64.zip` (12,0 MB), `platform-tools-latest-windows.zip` (8,0 MB), `nvm-setup.zip`, `MusicBeeSetup_3_6_Update1.zip`, `RAMMap.zip`, `KeyFreeze.zip`, `helvetica-neue-5.zip`, `univers.zip`, `arial.zip`. Lain-lain: `KMSpico Portable.rar`, `Windows Update Blocker 1.6.zip`, `Nitro Pro … Crack.rar`, `win10-21h1_…tib` (21,5 GB) di `D:\BACKUP SYSTEM - JGN DIHAPUS`.
→ **Tidak** ada kandidat backup WordPress di antara ini.

---

## 4. Pemetaan kandidat ke domain + tingkat keyakinan

### 4.1 `cgi-bin.zip` → berisi **keempat** instalasi

Distribusi anggota berdasarkan prefix level-1 (dari central directory):

| Prefix di dalam arsip | Jumlah anggota | Pemetaan domain |
|---|---|---|
| `smk10novemberkebumen.sch.id/` | 13.904 | **kebumen** |
| `smk10novembercibitung.sch.id/` | 13.837 | **cibitung** |
| `smk10novembersragen.sch.id/` | 8.220 | **sragen** |
| `wp-content/` + `wp-includes/` + `wp-admin/` + 23 berkas root + `cgi-bin/` | 8.007 | **root = tambun** (lihat §4.2) |
| **Total** | **43.968** | |

### 4.2 Root `D:\smk10november` = **smk10novembertambun.sch.id** — keyakinan **TINGGI**

Bukti (bukan asumsi):
1. Di dalam instalasi root sendiri: `wp-content/ai1wm-backups/smk10novembertambun.sch.id-20260109-012612-736.wpress` — plugin AI1WM menyimpan backup dengan **nama domain situs itu sendiri**.
2. `D:\smk10november\robots.txt` memuat `Sitemap:https://smk10novembertambun.sch.id/sitemap.xml` (dan varian http).
3. Dump `smkq8953_utama1.sql` hanya memuat URL domain **`smk10novembertambun.sch.id`** (lihat §4.4) → DB utama = tambun.
4. Tiga instalasi lain berada sebagai **subdirektori bernama domain mereka** di dalam satu akun hosting yang sama (`/home/smkq8953/public_html/…`, terbukti dari `error_log` Fase 2).

### 4.3 cibitung / kebumen / sragen — keyakinan **TINGGI**

Nama direktori = nama domain, struktur WordPress utuh di dalam masing-masing, dan setiap instalasi memuat `wp-content/ai1wm-backups/` sendiri (namun **kosong** — hanya `.htaccess`, `index.php`, `web.config`; tidak ada `.wpress` untuk ketiga domain ini).

### 4.4 `smkq8953_utama1.sql` → database **tambun** — keyakinan **TINGGI**

| Properti | Nilai (bukan rahasia) |
|---|---|
| Jenis | teks plain (bukan gzip) |
| Tabel | **132** `CREATE TABLE`, prefiks tabel **`smk_`** (bukan `wp_`) |
| Tabel inti terlihat | `smk_options`, `smk_users`, `smk_usermeta`, `smk_posts`, `smk_comments`, `smk_commentmeta` |
| Plugin terlihat dari nama tabel | `smk_actionscheduler_*` (Action Scheduler), `smk_cartflows_*`, `smk_mailpoet_*`, `smk_e_events`/`smk_e_notes*` (Elementor) |
| Pola URL domain yang muncul | **hanya `https://smk10novembertambun.sch.id/...`** (ratusan kemunculan) |
| Kemunculan kata sensitif (pass/token/secret/salt/key) | **2.734** kemunculan → **tidak ditampilkan** (aturan 7) |

### 4.5 Bukti asal-usul dump — `index.htm` — keyakinan **TINGGI**

Judul halaman: `brantas.iixcp.rumahweb.net / localhost / smkq8953_utama1 | phpMyAdmin 5.2.3`
⇒ Dump `smkq8953_utama1.sql` diekspor dari **phpMyAdmin 5.2.3** pada host **`brantas.iixcp.rumahweb.net`**, database **`smkq8953_utama1`**. Ini konsisten dengan akun `smkq8953` pada jejak `error_log` Fase 2 (`/home/smkq8953/public_html`) dan menegaskan lingkungan **Rumahweb (cPanel)**.

### 4.6 `UNKNOWN` (tidak dipetakan)

Tidak ada kandidat backup yang berstatus UNKNOWN — seluruh arsip besar berhasil diklasifikasikan (§3.2, §3.4). Yang **belum dapat dipetakan** adalah **database untuk cibitung/kebumen/sragen** (lihat §5).

---

## 5. Backup yang belum memiliki pasangan database atau file

| Domain | Backup file (ZIP) | Backup database (SQL) | Status |
|---|---|---|---|
| **smk10novembertambun.sch.id** | ✅ ada — di `cgi-bin.zip` (root instalasi) **dan** di dalam `.wpress` AI1WM 442,7 MB | ✅ ada — `smkq8953_utama1.sql` (80,5 MB) | **Lengkap** |
| **smk10novembercibitung.sch.id** | ✅ ada — 13.837 anggota di `cgi-bin.zip` | ❌ **tidak ditemukan** | **Tanpa pasangan DB** |
| **smk10novemberkebumen.sch.id** | ✅ ada — 13.904 anggota di `cgi-bin.zip` | ❌ **tidak ditemukan** | **Tanpa pasangan DB** |
| **smk10novembersragen.sch.id** | ✅ ada — 8.220 anggota di `cgi-bin.zip` | ❌ **tidak ditemukan** | **Tanpa pasangan DB** |

Tambahan:
- Di dalam `cgi-bin.zip` **tidak ada anggota `.sql`/`.sql.gz`/dump** sama sekali (0 anggota) — jadi DB ketiga domain **tidak** berada di dalam arsip ini.
- Satu-satunya backup AI1WM (`.wpress`) adalah **milik tambun** dan berukuran 442.722.278 B bertanggal **2026-01-09 01:26:12** (dari nama berkas). `.wpress` umumnya memuat **file situs + dump database** di dalam satu kontainer.
- **Peringatan aturan 10:** tanggal 2026-01-09 (lebih tua daripada aktivitas dropper Agu–Sep 2026 dan defacement 12-Okt 2026) **bukan bukti** bahwa backup ini bersih. Kebersihannya **belum diverifikasi** dan tidak dapat disimpulkan dari nama/tanggal/hash.

---

## 6. Kendala akses dan berkas yang belum dapat diperiksa

1. **Cakupan sengaja dibatasi** (aturan 8). Lokasi yang belum diperiksa tercantum di §2; perlu persetujuan sebelum diperluas.
2. **Tidak ada ekstraksi** (aturan 1/E) ⇒ seluruh verifikasi isi ZIP dilakukan pada **central directory** (nama, ukuran, CRC, tanggal internal, flag enkripsi). **Isi berkas di dalam arsip belum diperiksa.**
3. **`.wpress` belum dibuka.** Membaca indeks AI1WM = pekerjaan tingkat ekstraksi/parsing kontainer dan berkemungkinan memuat dump DB + data sensitif ⇒ **menunggu persetujuan terpisah**.
4. **Dump SQL tidak diperiksa isinya** selain: jenis berkas, jumlah tabel, nama tabel, dan pola URL domain. Baris `INSERT` **tidak** dibaca/ditampilkan (aturan 7); jumlah baris pada laporan ini indikatif.
5. **Tidak ada akses jaringan/hosting/DB** ⇒ nama database cibitung/kebumen/sragen tidak dapat dipastikan dari sisi server; tidak ada verifikasi apakah backup mereka tersedia di hosting.
6. **Risiko integritas bukti masih terbuka** (dibawa dari Fase 3A): Windows Defender terbukti mengarantina berkas malware di direktori laporan (ThreatID 2147761377) dan **bisa** bertindak terhadap berkas di direktori bukti/`Downloads` kapan saja. Pada fase ini **tidak** ada perubahan konfigurasi antivirus yang dilakukan.
7. **Ketidaksesuaian jumlah berkas** antara arsip dan salinan bukti: `cgi-bin.zip` memuat **lebih banyak** berkas daripada `D:\smk10november` (cibitung 13.837 vs 12.156; kebumen 13.904 vs 12.029; sragen 8.220 vs 7.445; root 8.007 vs 7.276). Selisih ini **belum ditelusuri** (berpotensi memuat berkas yang tidak ada di salinan bukti).

---

## 7. Langkah berikutnya yang direkomendasikan

| # | Langkah | Prasyarat |
|---|---|---|
| 1 | **Perluas cakupan pencarian** ke lokasi yang belum diperiksa (§2), sebagai prioritas mencari dump DB untuk cibitung/kebumen/sragen | **Persetujuan Saka** (aturan 8) |
| 2 | Konfirmasi nama database untuk cibitung/kebumen/sragen (mis. `smkq8953_cibitung`, `smkq8953_sragen`, …) atau keberadaan dump-nya (mis. di folder hosting/backup lain) | Informasi dari Saka / hosting |
| 3 | **Selisih berkas arsip vs salinan bukti** (§6.7): daftar lengkap berkas yang ada di `cgi-bin.zip` tetapi tidak ada di `D:\smk10november` | Tidak ada (dapat dilakukan read-only dari central directory) |
| 4 | Verifikasi integritas arsip lebih kuat: `testzip()`/CRC menyeluruh (membaca isi, read-only, **tanpa** menulis) untuk memastikan arsip tidak korup | Persetujuan (membaca 1,33 GB) |
| 5 | Analisis isi `.wpress` (apakah memuat dump DB tambun & apakah memuat artefak berbahaya) | **Persetujuan terpisah** (kontainer AI1WM) |
| 6 | Audit isi dump SQL (tabel/opsi mencurigakan, URL situs, akun admin) memakai kerangka query Fase 3A §7 | **Persetujuan terpisah** + tetap tanpa menampilkan rahasia |
| 7 | Tentukan strategi pemulihan: karena `cgi-bin.zip` **memuat artefak berbahaya**, pemulihan darinya **wajib** disertai pembersihan pasca-restore | Keputusan Saka |

---

## 8. Kepatuhan aturan

| Aturan | Status |
|---|---|
| 1. READ-ONLY, tanpa ekstraksi | **Dipatuhi** — tidak ada ekstraksi; hanya central directory dibaca |
| 2. Tidak mengubah `D:\smk10november` / laporan lama | **Dipatuhi** — hanya ditambah berkas baru di `audit-backup\` |
| 3. Tidak mengeksekusi berkas dari arsip | **Dipatuhi** |
| 4. Tidak mengimpor dump SQL | **Dipatuhi** — tidak ada koneksi DB sama sekali |
| 5. Tidak ada jaringan/login hosting/cPanel/kredensial/OTP | **Dipatuhi** |
| 6. Tidak menonaktifkan antivirus / mengubah Defender | **Dipatuhi** |
| 7. Tidak menampilkan kata sandi/token/cookie/private key | **Dipatuhi** — 2.734 kemunculan kata sensitif di SQL **tidak** ditampilkan; hanya jumlahnya |
| 8. Tidak memindai seluruh disk tanpa batas; lapor lalu tanya | **Dipatuhi** — cakupan terbatas, daftar lokasi diperiksa & belum diperiksa dilaporkan (§2) |
| 9. Tidak membuat perubahan sistem | **Dipatuhi** — seluruh analisis lewat skrip Python read-only |
| 10. Tidak menyimpulkan backup bersih dari nama/tanggal/hash | **Dipatuhi** — dinyatakan eksplisit di §5 |
