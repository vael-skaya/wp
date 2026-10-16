# FASE 6D — INVESTIGASI SUMBER FITUR PPDB & VERIFIKASI LOGO CIBITUNG

**Tanggal:** 16 Oktober 2026 · **Sifat fase:** **READ-ONLY** — tidak ada situs, database, konfigurasi, menu, atau berkas yang diubah
**Kesimpulan singkat:** **Tidak ada plugin/fitur PPDB** yang pernah terpasang di Cibitung maupun Tambun — halaman "Pendaftaran" **memang** placeholder asli dari sumber. **Logo Cibitung yang Anda siapkan terverifikasi benar dan merupakan kandidat terbaik** untuk pemasangan di fase berikutnya.

---

# BAGIAN A — INVESTIGASI FITUR PPDB

## A1. Ringkasan hasil

| Pertanyaan | Jawaban | Tingkat kepastian |
|---|---|---|
| Ada fitur/plugin PPDB? | **Tidak ada** (di kedua situs) | **Tinggi** — dibuktikan dari daftar plugin aktif, CPT, opsi, tabel, konten, dan menu |
| Plugin PPDB? | **Tidak teridentifikasi satu pun** | Tinggi |
| Berupa formulir, sistem PPDB, atau halaman informasi? | **Halaman informasi statis + tautan WhatsApp** | Tinggi |
| Placeholder "Halaman dalam pengembangan…" | **Konten ASLI yang tersimpan di sumber** (bukan komponen hilang) | **Tinggi** |

## A2. Bukti dan sumbernya (Cibitung & Tambun diperiksa terpisah)

**1) Daftar plugin AKTIF pada sumber (`wp_options.active_plugins` di dump):**

| Situs | Plugin aktif (7 / 2) | Ada plugin PPDB? |
|---|---|---|
| **Cibitung** | `classic-editor`, `disable-wordpress-updates`, **`elementor-pro`**, `elementor`, **`elementskit-lite`**, **`skyboot-custom-icons-for-elementor`**, **`wp-file-manager`** | **tidak ada** |
| **Tambun** | `super-easy-maintenance-mode`, **`wp-file-manager`** | **tidak ada** |

Catatan: `elementskit-lite` + `skyboot-custom-icons-for-elementor` menjelaskan **sumber ikon `ekiticons`** yang diperbaiki pada Fase 6C. `elementor-pro` menjelaskan widget `form`/Theme Builder yang tidak dipulihkan. `wp-file-manager` adalah plugin berisiko yang **tidak** dipulihkan.

**2) Custom post type:** di Cibitung `{wp_navigation:1, elementor_library:7, page:15, nav_menu_item:22, attachment:153, envato_tk_import:1, revision:153, customize_changeset:89, oembed_cache:149, request:89, post:2}`; Tambun serupa (11 halaman, 8 library, 34 changeset, 45 oembed). **Tidak ada CPT pendaftaran/siswa/PPDB.**

**3) Opsi:** pencarian kata kunci `ppdb|spmb|pendaftar|penerimaan|admission|enroll` pada `wp_options` → **0 baris** di kedua situs.

**4) Tabel plugin pada dump (tanpa mengimpor):** yang berpola form adalah **`*_frmt_form_entry*` (Formidable Forms)**, **`*_wpforms_*` (WPForms)**, `*_mailpoet_forms*`, `*_hostinger_reach_forms*` — **tidak ada satu pun tabel bergaya PPDB**. Penting: plugin pemilik tabel-tabel itu **tidak ada di `active_plugins`** → **tabel ada ≠ plugin dipakai** (persis peringatan Anda).

**5) Shortcode pada konten & metadata:** yang dominan adalah `[elementor-tag …]` (1256×/1197×), `[all-fields]` (216×/180×), `[elementor-element …]`, serta `[name]`, `[email]`, `[message]`, `[field_0babf3a]` → **tanda tangan formulir Elementor Pro**, bukan shortcode plugin PPDB.

**6) Shortcode/tabel Formidable & WPForms:** ada jejak instalasi lama, tetapi **tidak aktif** dan **tidak ada** form pendaftaran yang direferensikan halaman mana pun.

**7) Email `ppdb@demo.net` muncul 447×** di metadata → ini **teks demo template kit "Skoola"** (termasuk kalimat "telah dibuka pendaftaran (ppdb) tahun ajaran 2026"), **bukan** konfigurasi sistem sekolah.

**8) Halaman Pendaftaran (ID 984) di sumber — identik di kedua situs:**
```
konten (45 byte): <p><em>Halaman dalam pengembangan...</em></p>
```
Revisi Tambun menunjukkan halaman ini **pernah** berisi 2.420 B **Lorem ipsum** (demo), lalu diganti placeholder → **tidak pernah ada sistem pendaftaran**.

**9) Tautan pendaftaran yang benar-benar ada:**
- Cibitung: `https://wa.me/6289644763265`, `https://wa.me/6281310966434` (footer 799 & halaman Kontak 826).
- Tambun: bilangan WhatsApp yang sama + `oembed_cache` berisi URL **situs sendiri** `https://smk10novembertambun.sch.id/pendaftaran/embed/#?secret=…` (34 entri) — jejak penyematan halaman Pendaftaran sendiri, bukan sistem eksternal.
- Keduanya juga memuat gambar pengumuman `SPMB-GELOMBANG-II-scaled.webp` (lampiran 1061).
→ Pola yang ada: **menu "Pendaftaran" → halaman statis + kontak WhatsApp + banner pengumuman SPMB**.

## A3. Perbedaan Cibitung vs Tambun
- **Sama:** tidak ada plugin PPDB, halaman Pendaftaran identik (placeholder 45 B), nomor WhatsApp sama, teks demo template kit sama.
- **Berbeda:** Cibitung punya `elementor-pro` + `elementskit-lite` + `skyboot` + `classic-editor` aktif; Tambun hanya `super-easy-maintenance-mode`. Tambun punya 34 `oembed_cache` penyematan halaman Pendaftaran sendiri; Cibitung tidak.
- Keduanya sama-sama punya `wp-file-manager` (tidak dipulihkan).

## A4. Langkah berikutnya yang aman (tanpa menebak)
1. Tanyakan ke sekolah: **apakah PPDB/SPMB 2026/2027 dijalankan daring**, dan kalau ya, **di sistem mana** (domain PPDB terpisah, Google Form, atau formulir di situs).
2. Sampai ada jawaban resmi, **pertahankan placeholder** dan cukup rapikan tampilannya.
3. Jika sekolah ingin formulir di situs: pakai plugin formulir gratis yang sudah dipasang (**WPForms Lite**, lihat 6C) — **bukan** plugin PPDB lama, dan **bukan** Formidable/WPForms lama dari backup yang belum diverifikasi.

---

# BAGIAN B — VERIFIKASI LOGO CIBITUNG

## B1. Koreksi penting tentang lokasi
Path yang Anda sebut sebagai **folder** ternyata **sebuah berkas**:
`C:\Users\sakha\Downloads\logo-SMK-10-November-Cibitung.png` (folder `…\logo-SMK-10-November-Cibitung\` **tidak ada**). Pencarian nama serupa di `Downloads` (maks 4 level) menemukan tepat satu kandidat itu.

## B2. Kandidat dari sumber lokal (metadata lengkap)

| # | Path | Berkas | Ukuran | Format | Dimensi | Transparansi | SHA-256 (24 char) |
|---|---|---|---|---|---|---|---|
| **L1** | `C:\Users\sakha\Downloads\logo-SMK-10-November-Cibitung.png` | logo-SMK-10-November-Cibitung.png | **261.615 B** (255,5 KB) | PNG 8-bit, color_type 6 | **740×740** (rasio 1,000; 0,55 MP) | **ADA (RGBA)** | `d55fcb09c3b74503c25633a32…` |
| L2 | `C:\Users\sakha\Downloads\1.-Logo-SMK-10-November-Tambun-Selatan.png` | (logo **Tambun**) | 276.210 B | PNG 8-bit, RGBA | 740×740 | ADA | `31fbd7e19cdb76a9ffd7d39…` |
| L3 | `C:\xampp\htdocs\staging\cibitung\wp-content\uploads\2025\12\1.-Logo-…Tambun-Selatan.png` | dipakai Cibitung sekarang (lampiran **1818**) | 276.210 B | PNG RGBA | 740×740 | ADA | (sama dengan L2, byte identik) |

**Pratinjau visual L1 (dijalankan dengan alat visi):** lambang **perisai** berlatar **merah**, teks kuning **"SMK 10 NOVEMBER"** (atas) dan **"CIBITUNG – BEKASI"** (bawah), motif lilin/obor di atas buku terbuka, dikelilingi padi dan kapas → **benar dan spesifik untuk Cibitung**.

## B3. Pencarian tambahan di backup/media lama

| Sumber | Hasil |
|---|---|
| Manifest media Cibitung (`cibitung-media-manifest.tsv`, 153 lampiran) | hanya **1** berkas bernama "logo": `2025/12/1.-Logo-SMK-10-November-**Tambun**-Selatan.png` |
| Pohon bukti Cibitung `wp-content/uploads` (pencarian `*logo*`) | hanya gambar **demo ThemeGrill** (`2018/12/logo1-free-img*.png`, `logo@2x-free-img*.png` — placeholder tema, **bukan** logo sekolah) |
| Lampiran staging Cibitung | hanya lampiran **1818** = "1. Logo SMK 10 November Tambun Selatan" |
| Manifest arsip `cgi-bin.zip` (43.968 anggota, dibaca sebagai daftar — **tidak diekstrak**) | **0** entri memuat "logo" atau "cibitung" |
| Opsi tema staging | `site_icon = 0` (Cibitung **belum punya favicon**); `site_logo` dihapus di 6C (fallback teks) |

**Kesimpulan pencarian:** pada sumber yang diperiksa, **tidak ada logo Cibitung asli** — yang ada hanyalah logo **Tambun** (lampiran 1818, masih dirujuk data sumber Cibitung) dan placeholder demo tema.

## B4. Kandidat yang direkomendasikan

> **L1 — `logo-SMK-10-November-Cibitung.png`** (740×740 PNG RGBA, transparan, hashes di atas).
> **Alasan:** (a) teksnya benar "CIBITUNG – BEKASI"; (b) **dimensi 740×740 identik** dengan logo yang sekarang dipakai Astra, sehingga penggantian tidak akan mengubah tata letak header (dimensi blok tidak berubah); (c) PNG + alpha → latar bersih di header berwarna; (d) rasio 1:1 (kotak) → cocok untuk lambang perisai.
> **Tidak ada ambiguitas** yang memaksa saya menyodorkan beberapa kandidat: hanya L1 yang merupakan logo Cibitung.

**Favicon:** L1 juga layak dipakai sebagai favicon (`site_icon` Cibitung saat ini **0** / kosong). Idealnya file terpisah berukuran 512×512 atau 32×32; karena belum ada, L1 dapat dipakai sebagai alternatif sementara (hasil tidak diuji di fase ini). Lampiran lama **1818 = logo Tambun → jangan dipakai** untuk Cibitung (sesuai catatan Anda).

## B5. Logo lokal vs logo di backup — sama atau berbeda?
**Berbeda.** L1 (Cibitung) dan L2/L3 (Tambun) **byte-berbeda** (`cmp` ≠ identik; SHA-256 berbeda) meskipun berdimensi sama 740×740 — dua lambang sekolah yang berbeda teks/wilayah.

## B6. Sumber yang diperiksa & keterbatasannya
- **Diperiksa:** folder/berkas yang Anda sebutkan; `Downloads` (maks 4 level); manifest media Cibitung; pohon bukti uploads Cibitung (partial `*logo*`); lampiran staging Cibitung; manifest arsip `cgi-bin.zip`; `wp_options`/`wp_postmeta` Cibitung; dump Cibitung & Tambun.
- **Belum/tidak diperiksa:** **isi** arsip `cgi-bin.zip` (hanya daftar anggotanya — sesuai larangan mengekstrak); pohon bukti situs lain (Kebumen/Sragen) untuk kemungkinan logo Cibitung; arsip media lain di `D:\` bila ada yang belum diinventarisasi; **attachment metadata** Cibitung untuk berkas non-"logo" yang mungkin berupa lambang (mis. `site_icon`/favicon lama).
- **Tidak menyimpulkan** "logo Cibitung tidak ada di seluruh backup" — hanya: **tidak ada pada sumber yang diperiksa di atas**.

## B7. Rekomendasi pemasangan (untuk fase berikutnya, belum dijalankan)
1. Unggah `logo-SMK-10-November-Cibitung.png` ke Media Library Cibitung (**tanpa mengubah/menghapus** berkas lokal, dan tanpa menyentuh lampiran 1818).
2. Catat ID lampiran baru, lalu set `site_logo` = ID itu (menggantikan fallback teks).
3. Opsional: set `site_icon` = ID tersebut sebagai favicon sementara.
4. Verifikasi: header desktop + mobile tidak overflow, ukuran logo wajar, tidak ada permintaan aset gagal; bandingkan sebelum/sesudah.
5. **Rollback:** hapus `site_logo` lagi → kembali ke fallback teks (berkas lokal & lampiran 1818 tidak pernah dihapus).
6. Pertimbangkan menyimpan master SVG/PNG vektor bila sekolah punya (PNG 740×740 cukup untuk header, kurang ideal untuk cetak/banner besar).

---

## C. Keamanan & integritas
- Cibitung dan Tambun tetap terpisah; Kebumen & Sragen **tidak disentuh**; `goly1264_ams` tidak disentuh; produksi tidak disentuh; tidak ada impor database; tidak ada plugin dipasang/diaktifkan; tidak ada berkas dihapus/dipindah/diganti nama/ditimpa; tidak ada perubahan `site_logo`, menu, Elementor, Astra, atau metadata; tidak ada kredensial ditampilkan.
- **Tidak ada berkas dari backup atau folder logo yang dieksekusi** (PNG hanya dibaca sebagai data; manifest arsip hanya dibaca sebagai teks).
- **Catatan keamanan saat menelusuri `Downloads`:** di sana ada berkas terkait kasus — `admin.php` (84.070 B, webshell lama) dan `cgi-bin.zip` (1.335.066.857 B). Keduanya **tidak disentuh** dan di luar lingkup fase ini; sebaiknya dipindahkan/dikarantina ke folder bukti bila belum.

## D. Pernyataan
**Selama Fase 6D tidak ada perubahan pada situs, database, konfigurasi, menu, Elementor, Astra, opsi logo, maupun berkas mana pun** — seluruh pekerjaan bersifat baca (dump, manifest, pohon bukti, folder lokal) dan pembuatan dokumen laporan ini. **Menunggu persetujuan Anda** sebelum memasang logo, mengubah menu, memasang/mengaktifkan plugin, memulihkan fitur PPDB, atau mengubah konten situs.
