# FASE 6I — AUDIT CSS KEBUMEN, REGRESI TIGA STAGING, DAN RENCANA STAGING SRAGEN

**Tanggal:** 17 Oktober 2026 · **Sifat fase:** audit read-only + 1 koreksi CSS kecil (Kebumen) · **Tidak ada eksekusi untuk Sragen**
**Ringkas:** CSS Kebumen **valid & benar-benar diterapkan** ✓ · ditemukan **satu ketidaktepatan pengukuran saya sendiri di 6H** (angka = tinggi kotak, bukan tinggi logo tercetak) → dikoreksi + checkpoint ✓ · regresi Cibitung/Tambun/Kebumen: **tidak ada** ✓ · **rencana staging Sragen disusun, tidak dijalankan** ✓

---

## A. Audit CSS logo Kebumen

### A1. Identitas target (tanpa kredensial)
`C:\xampp\htdocs\staging\kebumen` → `DB_NAME` dari `wp-config.php` = **`smk10_staging_kebumen`** ✓ (bukan database lain).

### A2. CSS yang tersimpan pada Custom CSS post **ID 1975** (terbaca apa adanya)
```css
/* FASE 6H - ukuran logo Kebumen ... */
.site-logo-img img, .custom-logo { height: 60px; width: auto; max-height: 60px; object-fit: contain; flex: 0 0 auto; }
@media (max-width: 921px) { .site-logo-img img, .custom-logo { height: 48px; max-height: 48px; } }
@media (max-width: 544px) { .site-logo-img img, .custom-logo { height: 44px; max-height: 44px; } }
+ (ditambahkan pada fase ini, lihat A5)
```

### A3. Validasi sintaks
| Pemeriksaan | Hasil |
|---|---|
| Kurung kurawal | **seimbang** (5 buka / 5 tutup) ✓ |
| Nilai px dengan spasi (`60 px`) | **tidak ada** ✓ |
| Media query | `@media (max-width: 921px)`, `@media (max-width: 544px)` ✓ |
| Deklarasi | `height`, `width`, `max-height`, `object-fit`, `flex` ✓ |
| Nilai px dipakai | 44, 48, 60 (+ ambang 544/921) ✓ |

### A4. Apakah CSS benar-benar disajikan & diterapkan browser?
Disajikan pada `<style id="wp-custom-css">` (516 byte, HTTP **200**) ✓. **Computed style pada logo yang terlihat** memakai nilai CSS itu:

| Viewport | `computed height` | `computed max-height` | `object-fit` |
|---|---|---|---|
| 360 / 390 / 412 | **44px** ✓ | 44px | contain |
| 768 | **48px** ✓ | 48px | contain |
| 1366 | **60px** ✓ | 60px | contain |

⇒ CSS **valid, tersaji, dan diterapkan** — bukan sekadar tersimpan.

### A5. ⚠️ Koreksi temuan 6H saya sendiri + perbaikan
Pengukuran pada fase ini memakai metrik yang **benar** (ukuran yang **benar-benar dilukis**, memperhitungkan `object-fit: contain`) dan menemukan **kesalahan saya di laporan 6H**:

| | 6H (klaim lama) | Kenyataan terukur (6I, sebelum koreksi) |
|---|---|---|
| @390 | "35×44 px" | kotak **34,8×44 px**, tetapi logo **dilukis hanya ≈31×29 px** ✗ |
| Penyebab | — | **wadah `.site-logo-img` diperas oleh flex Astra** (`flex-shrink`) ⇒ kotak ±31 px lebar ⇒ `object-fit: contain` mengecilkan lukisan di dalamnya |

**Dampak:** pada mobile/tablet logo tampil **lebih kecil dari target 40–48 px** meski tidak terdistorsi dan tidak meluber. Angka "35×44 px" pada laporan 6H adalah **tinggi kotak**, bukan tinggi logo tercetak — itu keliru dan dikoreksi di sini.

**Tindakan (Kebumen saja, sesuai izin bagian A7):**
1. Checkpoint khusus dibuat **sebelum** perubahan: `checkpoint-6i-pre-kebumen.sql` (**2.116.031 B**) + salinan CSS lama `…\kebumen\qa6i\css-post-1975-sebelum.txt`.
2. Ditambahkan 3 baris terarah: `.ast-site-identity .site-logo-img, .site-logo-img { flex: 0 0 auto; width: auto; }` dan `.site-logo-img img, .custom-logo { width: auto; max-width: none; }` (CSS aktif 516 → **824 byte**).
3. **Uji ulang 5 viewport** (metrik ukuran-dilukis):

| Viewport | Ukuran **dilukis** | Rasio dilukis vs asli (232×218 = 1,064) | Overflow | Konsol |
|---|---|---|---|---|
| 360×800 | **46,8×44 px** | 1,064 → **1,064** ⇒ tidak terdistorsi ✓ | 0 px | 0 error |
| 390×844 | **46,8×44 px** | 1,064 ✓ | 0 px | 0 error |
| 412×915 | **46,8×44 px** | 1,064 ✓ | 0 px | 0 error |
| 768×1024 | **51,1×48 px** | 1,064 ✓ | 0 px | 0 error |
| 1366×768 | **63,8×60 px** | 1,064 ✓ | −14 px (scrollbar) | 0 error |

4. **Verifikasi visual** (screenshot `logo-setelah-koreksi@{390x844,1366x768}.png`): logo proporsional (tidak gepeng) ✓, jelas ✓, tersusun rapi di kiri judul dua baris "SMK 10 November Kebumen" ✓, **tidak menutupi/berdempetan** ✓, header seimbang ✓, judul situs utuh ✓.

**Tidak diubah:** berkas logo asli & attachment **1440** ✓, tagline ✓, konten ✓, data Elementor ✓, menu ✓.

### A6. Catatan kejujuran metodologi (mengapa dua alarm palsu muncul)
- **`tumpang=True`/jarak negatif** pada perhitungan rect saya **bukan** bukti tumpang tindih: `.site-title` di header Astra membentang satu baris penuh sehingga rect-nya mulai di kiri logo. Diverifikasi dengan mata (Kebumen & Cibitung @390): **tidak ada tumpang tindih** ⇒ **heuristik itu saya buang**, bukan dipakai sebagai temuan.
- **HTTP `000` untuk Cibitung** pada pemeriksaan awal = **artefak perintah saya sendiri** (`curl -w` sambil menangkap body ⇒ `client returned ERROR on write`), bukan situs mati. Curl langsung hari ini: **200** ✓ (judul `SMK 10 November Cibitung` ✓).

---

## B. Verifikasi regresi tiga staging (read-only)

DB diverifikasi dari masing-masing `wp-config.php`: `smk10_staging_kebumen` · `smk10_staging_cibitung` · `smk10_staging_tambun` ✓ (tidak ada situs yang menunjuk ke database situs lain).

| Pemeriksaan | Cibitung | Tambun | Kebumen |
|---|---|---|---|
| Nama sekolah (`blogname`) | SMK 10 November Cibitung ✓ | SMK 10 November Tambun Selatan ✓ | SMK 10 November Kebumen ✓ |
| Tagline (`blogdescription`) | *(kosong, sesuai sumber)* | *(kosong, sesuai sumber)* | Merajut Mimpi Penuh Cinta ✓ |
| Lampiran logo (`site_logo`) | **1948** ✓ (berkas pengguna) | **1316** ✓ (aset Tambun, **tidak** tertukar) | **1440** ✓ |
| Logo tersaji | 48 px ≤768 · **72 px** desktop ✓ | 48 px ≤768 · **72 px** desktop ✓ | **46,8/51,1/63,8 px** ✓ (setelah koreksi A5) |
| Distorsi rasio | **tidak** (1,000→1,000) ✓ | **tidak** (1,000→1,000) ✓ | **tidak** (1,064→1,064) ✓ |
| Kontainer `6c931a7` | **`display:none`** ✓ (tidak dihapus dari data ✓) | **`display:none`** ✓ | tidak ada elemen itu (Kebumen tidak memilikinya) |
| CSS kustom tersaji | ada (**1065 B**; logo + sembunyi kotak + aturan konten) ✓ | ada (**1065 B**) ✓ | ada (**824 B**) ✓ |
| Overflow horizontal (360→1366) | **0 px** (−15 px desktop) ✓ | **0 px** (−15 px desktop) ✓ | **0 px** (−14 px desktop) ✓ |
| Label `STAGING`/`pemulihan` | **0 / 0** ✓ | **0 / 0** ✓ | **0 / 0** ✓ |
| HTTP beranda | **200** ✓ | **200** ✓ | **200** ✓ |
| Aset tema Astra | utuh (44.905 B `main.min.css`) ✓ | utuh ✓ | utuh ✓ |

**Tidak ditemukan regresi** ⇒ **tidak ada perubahan** untuk Cibitung & Tambun pada fase ini ✓. Kontainer `6c931a7` **tidak dihapus** (sesuai instruksi) ✓.

---

## C. Rencana pembuatan staging Sragen — **DISUSUN, TIDAK DIJALANKAN**

> Sumber: `D:\backup-database-smk10november\smkq8953_sragen10.sql` — 80.897.366 B, SHA-256 `0448259bb0a11763…`, prefix sumber **`srg_`** (hanya dibaca; **tidak** diimpor, **tidak** diubah).

### C1. Yang sudah dipelajari (tanpa menjalankan apa pun)
- Pipeline terbukti: `forensik-smk10november\recovery-work\tools\pipeline6a.py <situs> <step>` (tahap: install/core/media/import/wipe/attach-make/purge-att/fixmime/regen/remap/k8/url-apply/perms/verify/checkmenu).
- Perbaikan penting yang sudah ada dan wajib dipakai: `unescape` yang benar (`tools\fase4c6.py`), emisi blok `<wp:term>` (menu terimpor otomatis), sanitasi `<wp:attachment_url>` + `WP_HTTP_BLOCK_EXTERNAL true` (mencegah importer mengunduh media dari produksi), `attach-make` + `fixmime` + `media regenerate` lokal.
- Metode: **konverter offline dump→WXR** lalu `wp import` ⇒ tidak ada data penyerang/plugin lama yang dimuat, tidak ada unduhan dari produksi.

### C2. Rencana (belum disetujui)
| Item | Rencana | Perlu persetujuan |
|---|---|---|
| Direktori tujuan | `C:\xampp\htdocs\staging\sragen` (**belum dibuat**) | ya |
| URL lokal | `http://localhost/staging/sragen/` (pola sama dengan 3 situs) | ya |
| Database baru | **usulan nama `smk10_staging_sragen`** — **tidak** dibuat sebelum disetujui; keberadaannya harus dicek dengan **satu perintah tertarget** (`SHOW DATABASES LIKE 'smk10_staging_sragen'`), **bukan** pemindaian massal & **bukan** lewat database lain | ya |
| Paket inti | WordPress + tema **Astra** + **Elementor** dari paket resmi yang sudah dipakai 3 situs lain (versi staging saat ini 7.0.7 Astra 4.14.0) — **tanpa** plugin baru/Pro | ya |
| Impor konten | dump→WXR (prefix `srg_`→`wp_`) + manifest media Sragen; **tanpa** memuat tabel plugin lama, akun lama, kredensial, atau payload IoC (`wp2shell_aa673039837f` **tidak** boleh terbawa) | ya |
| Checkpoint | `checkpoint-6j-01-pre-impor-sragen.sql` … di `recovery-work\staging\checkpoints\` | ya |
| Validasi | pola V-01..V-12 (JSON Elementor, rujukan media, aset CSS/JS, bahasa/TZ/favicon, 0 URL produksi, 0 IoC) + suite QA 6G/6H (label, logo, overflow CDP 6 viewport, konsol) | ya |
| Rollback | drop database `smk10_staging_sragen` + hapus direktori `htdocs\staging\sragen` (hanya atas persetujuan; tidak menyentuh backup/checkpoint lain) | ya |

### C3. Prasyarat/risiko yang harus diselesaikan **sebelum** eksekusi
1. **Manifest media Sragen** — harus dipastikan sudah ada & lengkap untuk prefix `srg_`; bila belum, harus dibuat dari dump (analisis 4D) **sebelum** impor, agar lampiran tidak dibangun dari tebakan.
2. **Media hilang** — risiko sama seperti Kebumen (27 berkas hilang). Perlu keputusan sebelum impor: terima apa adanya, atau jalankan prosedur ekstraksi terpandu terpisah (`cgi-bin.zip`) — **butuh persetujuan terpisah**.
3. **Perbedaan plugin/tema sumber** — daftar plugin Sragen (termasuk `wp-compat` yang belum diketahui) belum final; keputusan: hanya inti+Astra+Elementor (seperti 3 situs lain) atau memetakan plugin yang sah.
4. **Kontaminasi sumber** — IoC Sragen (`wp2shell_aa673039837f`, C2 `*.expansor.xyz`/`*.top`) harus **dikecualikan** dari impor dan diverifikasi 0 pada hasil.
5. **Kapasitas/ruang** — mengimpor dump 80,9 MB + media; perlu cek ruang disk (`C:`) sebelum mulai.
6. **K-1 selisih versi** (produksi 7.1.3 vs paket 7.0.4/7.0.7) tetap terbuka seperti situs lain.
7. **Label "(STAGING pemulihan)"** tidak pernah masuk ke dump sumber ⇒ tidak perlu "dibersihkan"; yang ada hanya di instalasi staging (sudah bersih di 3 situs).

### C4. Yang **tidak** boleh diasumsikan
Staging Sragen **tidak ada** saat ini (hasil pencarian 6H) ⇒ rencana ini adalah **pembuatan baru**, bukan perbaikan. Saya tidak menebak bahwa direktori/database itu sudah pernah ada.

---

## D. Kepatuhan batasan mutlak
- **Database `goly1264_ams`: tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah** — tidak ada satu pun perintah pada fase ini yang menargetkannya, dan tidak dipakai untuk mencari instalasi.
- **`C:\Users\sakha\Downloads\admin.php`: tidak disentuh** — tidak dibaca, tidak dijalankan, tidak dipindai, tidak diedit, tidak dipindah, tidak diganti nama, tidak dihapus.
- **Situs produksi tidak disentuh**; tidak ada deployment; tidak ada unduhan media dari produksi; **tidak ada impor ulang dump sumber**.
- Situs di luar lingkup (Sragen) **tidak diubah**: 0 direktori, 0 database, 0 berkas dibuat.
- Tidak ada pengujian yang diklaim tanpa dijalankan (termasuk: 768×1024 hanya diuji untuk beranda; halaman lain hanya 390/1366 pada 6H).

## E. Keterbatasan
- Pengujian responsif = **emulasi viewport via CDP**, **bukan perangkat Android fisik**; konsol diperiksa saat koreksi Kebumen (hook JS), bukan DevTools interaktif.
- Ukuran logo dinilai dari **metrik + verifikasi visual screenshot**, bukan pengukuran fisik perangkat.
- Regresi diperiksa pada **beranda** (6 viewport) dan sebagian halaman pada 6H; tidak semua halaman setiap situs.
- Rencana Sragen belum divalidasi lapangan (manifest media, plugin, ruang disk) — item C3 masih terbuka.

## F. Checkpoint & perubahan fase ini
| Item | Nilai |
|---|---|
| Checkpoint baru | `checkpoint-6i-pre-kebumen.sql` (**2.116.031 B**) |
| Perubahan | **hanya** isi post `custom_css` **1975** (Kebumen): 516 → 824 byte |
| Perubahan lain | **tidak ada** (Cibitung/Tambun/Sragen: 0 perubahan) |
| Rollback koreksi | hapus 3 baris CSS 6I pada post 1975 (salinan lama di `…\kebumen\qa6i\css-post-1975-sebelum.txt`) atau `mysql < checkpoint-6i-pre-kebumen.sql` |

## G. Bukti visual
`recovery-work\staging\{kebumen,cibitung,tambun}\qa6i\` — `beranda@{390x844,1366x768}.png` untuk ketiga situs + `logo-setelah-koreksi@{390x844,1366x768}.png` (Kebumen).
