# FASE 6J — AUDIT AWAL SRAGEN DAN INVESTIGASI CONTAINER ELEMENTOR KOSONG

**Tanggal:** 18 Oktober 2026 · **Mode:** **read-only menyeluruh** · **Perubahan data: 0** (tidak ada direktori, database, berkas, CSS, plugin, atau container yang dibuat/diubah/dihapus)

---

## 1. Ringkasan
| Butir | Hasil |
|---|---|
| Manifest media Sragen | **belum ada** ✗ — hanya inventaris pohon berkas; rencana pembuatan manifest offline disusun |
| Plugin/tema sumber Sragen | tema **hello-elementor** (bukan Astra), 11 plugin aktif dari dump (termasuk **elementor-pro**), **119 tabel plugin**, `wp-compat` **0 bukti** |
| Indikator kompromi | `wp2shell` **18 kemunculan** di dump (menu item) ✗ → wajib dikecualikan; domain C2 **0 di database** (batas pemeriksaan dinyatakan) |
| Disk | C: **107 GB** kosong · D: **872 GB** → estimasi kebutuhan **1–1,8 GB** ✓ |
| DB tujuan | `smk10_staging_sragen` **TIDAK ADA** (dicek tertarget) — **tidak dibuat** |
| Container `7743b359`/`37b2b453` | **ditemukan di ketiga staging**, halaman **899 "Struktur Organisasi"** · **bukan container tanpa isi data** — penyebab kotak kosong: **heading dengan dynamic tag `page-title` (fitur Elementor Pro)** |

---

## 2. Prasyarat staging Sragen

### 2.1 Manifest media — **TIDAK ADA**, hanya inventaris
| Sumber | Isi |
|---|---|
| `manifest\inventory-smk10novembersragen.sch.id_.txt` | **7.445 baris**; **2.421 entri** di `wp-content/uploads` (1.046 .jpg · 833 .webp · 170 .png · 49 .jpeg · 1 .zip · 1 .pdf) + `sha256-…sragen….txt` |
| `recovery-work\sragen\` | **kosong** (tidak ada artefak) |
| Dump `smkq8953_sragen10.sql` (baca saja, 80.897.366 B) | `post_type='attachment'` = **107** · `_wp_attached_file` = **107** · `_thumbnail_id` = **0** · rujukan `/wp-content/uploads/` = **1.730** |

**Celah yang teridentifikasi:** inventaris berisi **2.421 berkas** sedangkan lampiran asli hanya **107** — selisihnya adalah **turunan ukuran (thumbnails) + aset tema/plugin**. ⇒ manifest media Sragen **harus dibuat** dari (a) 107 `_wp_attached_file` di dump + (b) pencocokan nama dasar thumonal ke inventaris, **offline**, tanpa mengarang nama/path/ID (rencana: §9).

### 2.2 Plugin dan tema
**Tema sumber: `hello-elementor`** ✓ (berbeda dari tiga situs lain yang memakai Astra ⇒ rencana staging Sragen **tidak boleh** menyalin asumsi Astra).

**`active_plugins`** (a:11 = 11 entri ✓; cuplikan nilai dump — sebagian nama terbaca utuh, sisanya belum dibaca penuh ⇒ ditandai ⚠):
`classic-editor` · `disable-wordpress-updates` · **`elementor-pro`** ✗(butuh lisensi) · `elementor` · `envato-elements` · `extensions-for-elementor-form` · `google-listings-and-ads` · ⚠(3 entri lain belum dibaca penuh)

**Bukti tabel plugin di dump (119 tabel non-inti):** `woocommerce_*` (belanja/e-commerce) · `wpforms_*` · `wpf_*` (**wp-file-manager**) · `woof_*` (filter produk WooCommerce) · dll.

**Plugin yang ada di inventaris berkas:** `wp-file-manager` (**914 berkas**) ✗ dan `custom-codes` (**330 berkas**) ✗ — keduanya kelas **berisiko tinggi** (pengelola berkas & penyuntik kode arbitrer) ⇒ rekomendasi: **jangan dipasang** di staging.

**`wp-compat`:** **0 kemunculan** di dump **dan** di inventaris ⇒ **tidak ada bukti** plugin ini ada di Sragen. (Catatan: ketidakhadiran bukti bukan bukti ketiadaan di luar cakupan yang diperiksa — lihat §11.)

### 2.3 Indikator kompromi (terarah, tanpa menjalankan apa pun)
| Indikator | Hasil di **database** (dump) |
|---|---|
| `wp2shell` | **18 kemunculan** ✗ — nilai `_menu_item_url`: `https://wp2shell.research`, `https://wp2shell.com` + data menu Elementor `{"type":"custom","title":"x","url":"https://wp2shell.research"}` |
| `expansor.xyz` / `clarivum.top` / `zephyrous.xyz` / `quiddity.top` | **0** |

**Artinya:** dump Sragen memuat **item menu yang disisipkan penyerang** (`wp2shell.*`) ⇒ wajib **dikecualikan** dari migrasi dan diverifikasi 0 pada hasil (aturan eksklusi: menu item dengan URL berdomain `wp2shell.*` atau judul `x` dari sumber tidak diimpor).
**Batasan yang harus dibaca:** pencarian ini **hanya database**. Nol hasil untuk domain C2 **tidak membuktikan** dump/berkas aman; pemeriksaan berkas (IoC di file) dilakukan pada fase forensik 2–4, bukan pada fase ini. Tidak ada berkas/jaringan yang dijalankan.

### 2.4 Kapasitas disk
| Drive | Kapasitas | Terpakai | Kosong |
|---|---|---|---|
| C: | 223 GB | 117 GB (53%) | **107 GB** |
| D: | 932 GB | 61 GB (7%) | **872 GB** |

**Estimasi kebutuhan staging Sragen** (belum diverifikasi terhadap ukuran uploads sebenarnya — ukuran 2.421 berkas belum dijumlahkan):

| Komponen | Perkiraan |
|---|---|
| WordPress 7.0.x + tema hello-elementor + Elementor (paket resmi) | ~250 MB |
| Uploads 2.421 berkas (rata-rata 100–300 KB) | **0,25–0,75 GB** |
| Turunan thumbnail + CSS Elementor | ~0,15 GB |
| Database (80,9 MB dump → tabel inti saja) | ~0,1–0,15 GB |
| Checkpoint SQL (3–5 buah) | ~0,15 GB |
| **Total** | **≈1–1,8 GB** ⇒ **aman** (107 GB tersedia) |

### 2.5 Database tujuan (**tidak dibuat**)
- Layanan yang diperiksa: **MariaDB lokal XAMPP 10.4.32, port 3306** ✓ (bukan layanan lain).
- `SHOW DATABASES LIKE 'smk10_staging_sragen'` → **kosong** ⇒ **belum ada** ✓.
- **Tidak ada** pembuatan database, **tidak ada** enumerasi database lain, **tidak ada** akses ke database lain.

---

## 3. Identitas container `7743b359` dan `37b2b453` — **DITEMUKAN**
| Aspek | Fakta terverifikasi |
|---|---|
| Lokasi | **Halaman post 899 "Struktur Organisasi" (publish)** — ada di **Cibitung, Tambun, dan Kebumen** sekaligus (berasal dari template kit yang sama) |
| URL | `http://localhost/staging/<situs>/struktur-organisasi/` (HTTP **200** di ketiganya) |
| Cara ditemukan | pencarian tertarget pada `_elementor_data` di **tiga staging terverifikasi** saja; **tidak** ada di `post_content`; **tidak** diperluas ke database lain/produksi |

---

## 4. Bukti pengaturan Elementor & penyebab area kosong

### 4.1 Struktur data (sama di ketiga situs)
**Induk `7743b359`** — `elType: container`, `isInner: false`, **1 anak**, widget di dalam = 1:
| Pengaturan | Nilai |
|---|---|
| `flex_direction` | column |
| `boxed_width` | 640 px |
| **`background_background`** | **classic** |
| `background_image` | foto **unggahan nyata** — Cibitung `2025/12/header_001.webp` (id 78) · Tambun `2026/01/WhatsApp-Image-2025-04-28-at-17.59.21-1.jpeg` (id 1215) · Kebumen `2026/06/WhatsApp-Image-2026-05-02-at-09.03.17.jpeg` (id 1477) — **berkasnya ADA di disk** (55.970 / 61.950 / 119.050 B) ✓ |
| `background_size` / `position` / `repeat` | cover / center center / no-repeat |
| `background_overlay_background` | classic |
| `background_overlay_image` | `overlay2.png` (id 444) — **ADA** (935 B) ✓ |
| **`background_overlay_color`** | **`#01295199`** (biru-navy, alpha 0x99 ≈ 60%) |
| `background_overlay_opacity` | 1 |
| `padding` / `_tablet` / `_mobile` | 7em / 5em / 3em atas-bawah |

**Anak `37b2b453`** — `isInner: true`, 1 anak (widget), `width: 50%`, `padding: 2em`, `gap tablet 14px`, `padding_mobile: 1em`.

**Widget di dalamnya = `heading` id `60d69bd2`** (Cibitung/Tambun/Kebumen):
| Pengaturan | Nilai |
|---|---|
| `title` | **"" (kosong)** ✗ |
| **`__dynamic__`** | `{"title":"[elementor-tag id=\"78232fa\" name=\"page-title\" settings=\"{}\"]"}` |
| `header_size` / `align` | h1 / center |
| `__globals__` | `title_color: globals/colors?id=text` |

### 4.2 ⚠️ Penyebab sebenarnya kotak kosong (fakta)
1. Isi satu-satunya container adalah **heading yang judulnya kosong** dan **diikat ke dynamic tag `page-title`** — **dynamic tag milik Elementor Pro**.
2. **Elementor Pro tidak terpasang** di staging (tanpa lisensi sekolah) ⇒ dynamic tag **tidak ter-resolve** ⇒ heading dirender **kosong** ⇒ band hanya menampilkan **latar (foto + overlay biru `#01295199`)** tanpa teks.
3. **Verifikasi render (CDP, 1366 px):** container `7743b359` = **1200×288 px**, `innerText` kosong, 0 `<img>` (background = CSS) → **band kosong ±288 px** ✓
4. **Bukti visual** (`…\staging\cibitung\qa6j\struktur-organisasi.png`): band biru-kelabu **kosong** tepat di atas gambar "STRUKTUR ORGANISASI" ✓ — persis gejala yang dilaporkan.
5. **BUKAN** "container tanpa isi data": data punya 1 heading + background + overlay. Klaim "`.e-con-inner` kosong" pada cuplikan HTML adalah **gejala render**, bukan struktur data.

### 4.3 Temuan tambahan yang terverifikasi — CSS latar tidak konsisten
| Situs | `post-899.css` | Aturan `7743b359` | `background-image` di CSS | Hasil render latar |
|---|---|---|---|---|
| Kebumen | 4.576 B | ada ✓ | **1** ✓ | `background-image: url(…)` **diterapkan** ✓ |
| Cibitung | 4.449 B | ada ✓ | **0** ✗ | computed `background-image: none` ✗ |
| Tambun | 4.449 B | ada ✓ | **0** ✗ | computed `background-image: none` ✗ |

⇒ CSS Elementor halaman 899 di **Cibitung & Tambun tampak basi/tidak lengkap** (aturan background tidak ikut tergenerasi), sementara di Kebumen lengkap. **Belum diperbaiki** (perubahan ⇒ butuh persetujuan). Kandidat perbaikan: regenerasi CSS Elementor (`wp elementor flush-css`) — **tindakan global**, harus disetujui lebih dulu.

---

## 5. Evaluasi fungsi bagi guru
| Pertanyaan | Jawaban (dibedakan fakta/dugaan) |
|---|---|
| Bisa diedit lewat Elementor? | **Fakta:** ya — container + heading adalah elemen Elementor normal di halaman 899. |
| Container kosong? | **Fakta:** tidak kosong secara data; **terlihat kosong** karena heading-nya dirender tanpa teks (dynamic tag Pro). |
| Guru bisa mengisi area itu via editor visual? | **Fakta:** bisa **mengganti heading** dengan teks biasa (tidak butuh Pro). **Dugaan:** sebagian guru mungkin bingung karena judul tak tampak di editor (ditampilkan sebagai chip dynamic tag). |
| Role membatasi? | **Fakta:** mengedit halaman butuh kapabilitas `edit_pages` (Editor/Admin). Staging saat ini hanya punya 1 akun (admin). |
| Pertahankan / hapus / ubah? | **Rekomendasi:** **PERTAHANKAN** container (ia adalah *banner band* halaman; menghapusnya menghilangkan latar + menggeser layout). Perbaikan tepat sasaran = **(a)** ganti heading dynamic-tag menjadi teks statis "Struktur Organisasi", atau **(b)** sembunyikan band dengan aturan `[data-id="7743b359"]` bila band memang tidak diinginkan. **Bukan** hapus seluruh container. |

---

## 6. Rekomendasi penghapusan/penyesuaian (untuk fase berikutnya, **belum dijalankan**)
Karena terbukti **tidak** kosong dan **punya fungsi layout**, **penghapusan tidak direkomendasikan**. Bila tetap dipilih:
1. Checkpoint DB khusus per situs.
2. Ubah **hanya** elemen `7743b359` / `37b2b453` di halaman 899 (bukan `.e-con-inner` global).
3. Verifikasi desktop + mobile, serta pastikan tidak ada section lain yang hilang.
4. Rollback: pulihkan `_elementor_data` post 899 dari salinan sebelum perubahan (atau restore checkpoint).

**Opsi perbaikan yang lebih aman (pilihan Anda):**
- **O1** Isi heading dengan teks statis (per situs, sesuai nama halaman) ⇒ band kembali bermakna.
- **O2** Sembunyikan band: `[data-id="7743b359"]{display:none}` (pola sama seperti `6c931a7`), **tanpa** menyentuh data.
- **O3** Regenerasi CSS Elementor (Cibitung & Tambun) agar latar foto tampil seperti Kebumen.
- **O4** Pasang Elementor Pro dengan lisensi sekolah ⇒ dynamic tag `page-title` berfungsi (paling "asli", tapi butuh lisensi).

---

## 7. Daftar prasyarat yang belum terselesaikan
1. **Manifest media Sragen** harus dibangun offline (107 lampiran ⇄ 2.421 berkas) — belum ada.
2. Ukuran total uploads Sragen belum dijumlahkan (estimasi ruang masih perkiraan).
3. **Daftar plugin aktif lengkap** (11 entri; 3 belum dibaca penuh) + keputusan plugin mana yang dipetakan (Elementor Pro/WooCommerce/WPForms?) — **keputusan sekolah**.
4. **Tema staging Sragen** harus **hello-elementor** (bukan Astra) — menyimpang dari 3 situs lain.
5. Aturan **eksklusi IoC** (`wp2shell.*` menu item) harus diterapkan & diverifikasi 0 pada hasil.
6. K-1 selisih versi (produksi 7.1.3 vs paket 7.0.4/x).
7. Opsi perbaikan container (O1–O4) — pilihan Anda.

---

## 8. Fakta vs dugaan vs belum dilakukan
- **Fakta teruji:** §2.1 hitungan dump & inventaris; §2.2 nama plugin/tema/tabel; §2.3 hit IoC; §2.4 disk; §2.5 DB tidak ada; §3 lokasi container; §4 pengaturan Elementor, render 1200×288 kosong, perbedaan CSS; §4.3 Kebumen vs Cibitung/Tambun.
- **Dugaan (belum dibuktikan):** bahwa guru akan bingung dengan dynamic tag; bahwa band "biru muda" yang dimaksud = band `7743b359` (kesamaan posisi/tinggi/warna sangat mendukung, tetapi tidak ada konfirmasi dari Anda).
- **Belum dilakukan:** pembuatan apa pun untuk Sragen; perbaikan CSS/container; regenerasi CSS; pemeriksaan berkas (IoC di filesystem) pada fase ini; pemeriksaan konsol untuk halaman 899.

---

## 9. Rencana fase berikutnya (belum dieksekusi)
**A. Sragen — pembuatan staging (butuh persetujuan eksplisit):**
1. Bangun manifest media offline dari dump (107 `_wp_attached_file`) + pencocokan ke inventaris; laporkan yang hilang.
2. Checkpoint & direktori: `C:\xampp\htdocs\staging\sragen`, DB `smk10_staging_sragen` (**dibuat hanya setelah disetujui**), checkpoint awal.
3. Instal WordPress + **hello-elementor** + Elementor dari paket resmi; `WP_HTTP_BLOCK_EXTERNAL true`.
4. Konversi dump→WXR dengan **pengecualian**: menu item `wp2shell.*`, akun lama, tabel plugin, opsi sensitif; sanitasi `<wp:attachment_url>`.
5. Impor → media dari manifest → `attach-make`/`fixmime`/`regen` → ID map → URL remap → perms → verify.
6. Validasi: V-01..V-12 + suite QA 6G/6H (label, logo, overflow 6 viewport, konsol) + **verifikasi 0 IoC**.
7. Rollback: drop DB + hapus direktori (atas persetujuan), checkpoint SQL tersimpan.

**B. Container (bila disetujui):** pilih O1/O2/O3/O4 → checkpoint → ubah hanya `7743b359`/`37b2b453` (atau CSS `[data-id]`) → verifikasi desktop/mobile → catat rollback.

---

## 10. Kepatuhan
- **Database `goly1264_ams`: tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah**, dan tidak dipakai mencari instalasi ✓. Satu-satunya Database yang diperiksa: **`SHOW DATABASES LIKE 'smk10_staging_sragen'`** (tertarget, hasil kosong) ✓.
- **`C:\Users\sakha\Downloads\admin.php`: tidak disentuh** — tidak dibaca, dijalankan, dipindai, diedit, dipindah, diganti nama, atau dihapus ✓.
- **Produksi tidak disentuh**; tidak ada deployment; tidak ada unduhan media; **dump sumber tidak diubah** (hanya dibaca) ✓; `cgi-bin.zip` **tidak diekstrak** ✓.
- **Staging Cibitung/Kebumen/Tambun tidak diubah** pada fase ini (0 perubahan) ✓.
- **Tidak ada** direktori/database/berkas dibuat untuk Sragen ✓ · **tidak ada** container dihapus/diubah ✓ · **tidak ada** plugin dipasang/dinonaktifkan ✓.

## 11. Keterbatasan
- Pencarian IoC **hanya pada database**; filesystem Sragen tidak dipindai ulang pada fase ini.
- `active_plugins` terbaca sebagian (11 entri, 8 nama utuh).
- Ukuran uploads Sragen belum dihitung (estimasi ruang berbasis asumsi ukuran rata-rata).
- Pengujian render dilakukan pada **emulasi viewport CDP**, bukan perangkat fisik; konsol halaman 899 tidak diperiksa.
- Kesamaan "kotak biru muda" dengan band `7743b359` didukung posisi/tinggi/warna, **belum dikonfirmasi** pengguna.
