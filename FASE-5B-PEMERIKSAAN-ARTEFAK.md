# FASE 5B — PEMERIKSAAN ARTEFAK MIGRASI (Kebumen, pilot)

**Tanggal:** 14 Oktober 2026 · **Situs:** Kebumen (`smk10novemberkebumen.sch.id`) · **Status:** pemeriksaan read-only, **belum ada impor**.

## 0. Ruang lingkup & batasan yang dipatuhi

| Aturan | Kepatuhan |
|---|---|
| Tidak mengimpor / tidak mengubah database staging | **Dipatuhi** — tidak ada perintah tulis ke MariaDB dijalankan |
| Tidak menyalin media | **Dipatuhi** — berkas hanya dibaca/stat/hash |
| Tidak mengubah dump SQL sumber | **Dipatuhi** — 4 dump tetap utuh (0 berubah) |
| Tidak menjalankan kode dari backup | **Dipatuhi** — hanya parsing byte/teks |
| `goly1264_ams` tidak disentuh | **Dipatuhi** — 29 tabel, tidak diakses |
| Situs produksi tidak berubah | **Dipatuhi** — tanpa koneksi jaringan ke hosting |

Artefak yang diperiksa: `kebumen-media-manifest.tsv`, `kebumen-staging-pilot.wxr`, `kebumen-staging-pilot.wxr.dryrun.txt`.
Alat baru: `tools/wxr_crosscheck.py`, `tools/ref_detail.py`, `tools/media_verify.py`, `tools/ref_map.py`, `tools/ref_media.py`, `tools/k8_plan.py`.

---

## 1. Tugas 1 — Validasi ulang WXR & silang dengan laporan mode kering

| Pemeriksaan | Hasil |
|---|---|
| XML well-formed (`xml.etree`) | **YA** |
| `wp:wxr_version` | 1.2 · `<channel>` utuh (title, link, pubDate RFC 2822, language id-ID) |
| Item di WXR vs rencana mode kering | **174 vs 174 — cocok** |
| ID hanya di rencana | tidak ada |
| ID hanya di WXR | tidak ada |
| Beda tipe/status | **tidak ada** |
| `wp:post_id` duplikat | **tidak ada** |
| `wp:post_type` | attachment 122 · nav_menu_item 19 · page 11 · post 10 · elementor_library 8 · wp_global_styles 2 · wp_navigation 1 · wpdmpro 1 |
| `wp:status` | inherit 122 · publish 52 |
| `wp:postmeta` / `<category>` | 357 / 39 |
| `dc:creator` unik | hanya `kebumen_admin` (tidak ada akun penyerang) |
| String IoC dalam berkas | **0** (`wp2shell`, `sso_token`, `_codehidden`, `shellcode`, `fm_key`, `wp_edvl`, `wp_zqyq`, `uvisyfyj`, `sso-loader`) |

**16 item tanpa `<title>` — dipastikan bukan cacat:** semuanya `nav_menu_item` (id 20, 749, 780, 781, 834, 835, 908–910, 967–968, 973–977). Di WordPress, judul item menu boleh kosong; label diambil dari objek yang ditautkan. Field wajib lain (`post_name`, `post_date`, `post_type`, `status`) **terisi**.

---

## 2. Tugas 2 — Verifikasi 122 entri manifes terhadap salinan bukti

Hasil `tools/media_verify.py` (rincian per berkas + SHA-256 di `kebumen/export/_5b-media-verify.tsv`):

| Ukuran | Hasil |
|---|---|
| Entri manifes | **122** |
| **Berkas ADA** | **122 (100 %)** — 0 hilang, 0 byte-kosong, 0 di lokasi lain |
| Total ukuran | **28.086.567 B** (± 26,8 MiB) |
| Jenis | `.jpeg` 72 · `.jpg` 21 · `.webp` 21 · `.png` 6 · `.mp4` 1 · `.svg` 1 |
| Turunan (thumbnail WP) di disk | 104 entri punya turunan; total **452** berkas turunan |
| Catatan khusus | tidak ada |

**Pemeriksaan dua berkas non-gambar:**
- `2026/09/WhatsApp-Video-2026-07-13-at-09.18.36.mp4` — 1.317.788 B, SHA-256 `4b2b1c2b6ede066b…`.
- `2026/09/Kalender-Pendidikan-Minggu-Efektif-Senov_00001.svg` — 456.918 B, SHA-256 `8683e271f924f3b2…`. Pemindaian isi: **tidak ada** `<script`, `onload`, `onerror`, `javascript:`, `<!ENTITY`, `<foreignObject`, `<?php` (hanya 1.294 `xlink:href`, lazim untuk SVG hasil konversi). **Bersih**, namun lihat §6 butir 4.

### 2b. Media yang DIRUJUK KONTEN tetapi tidak ada di salinan bukti

Konten menunjuk **217 path unik** di `wp-content/uploads` (293 rujukan): **190 ADA**, **27 TIDAK ADA**.

**19 berkas asli yang hilang:** `2025/12/`: `KALDIK_2025_2026.pdf`, `Signature.png`, `img_005.webp`, `img_007.webp`, `img_009.webp`, `img_014.webp`, `img_015.webp`, `img_016.webp`, `img_017.webp`, `img_ekstrakurikuler_002.webp`, `img_fasilitas_002.webp`, `img_fasilitas_008.webp`, `img_fasilitas_011.webp`, `img_fasilitas_015.webp`, `img_guru1.webp`, `img_prestasi_003.webp`, `img_prestasi_004.webp`, `img_prestasi_005.webp`, `img_prestasi_006.webp`.

**8 turunan yang hilang dan induknya juga tidak ada** (jadi tidak bisa diregenerasi): `SPMB-GELOMBANG-II-scaled.webp`, `img_fasilitas_002-300x200.webp`, `img_fasilitas_008-300x200.webp`, `img_fasilitas_011-300x200.webp`, `img_fasilitas_015-300x200.webp`, `img_guru1-214x300.webp`, `elementor/thumbs/img_dkv_001-…webp`, `elementor/thumbs/img_mp_001-…webp`.

**Penting:** tidak satu pun dari 27 berkas itu ada di manifes 122 → **tidak punya record lampiran di ekspor**, sehingga setelah impor akan menjadi gambar/tautan rusak. Konteks: direktori `uploads/2025/12` di salinan bukti hanya berisi 25 berkas, jauh di bawah jumlah yang dirujuk. Dugaan sumber pengganti: arsip backup hosting `cgi-bin.zip` (1,33 GB, Fase 4A) — **memerlukan persetujuan + prosedur ekstraksi terpandu** karena arsip itu berasal dari instalasi yang terkompromi.

**Koreksi kesalahan saya sendiri (dicatat demi transparansi):** versi pertama analisis ini melaporkan "126 berkas hilang" — itu **artefak regex** (path ikut menyerap `]]`). Setelah regex diperbaiki dan diverifikasi ulang dengan uji `-f` langsung, angka benar adalah **27**. Dua metode independen kini sepakat pada angka 27.

---

## 3. Tugas 3 — Audit rujukan ID & URL lama, dan cara pemetaannya

### 3.1 Bentuk rujukan yang ada

| Bentuk rujukan | Jumlah | Target |
|---|---|---|
| `_thumbnail_id` | 10 | **semua 10 → lampiran yang diekspor** ✓ |
| `_menu_item_object_id` | 19 | **semua → item yang diekspor** ✓ (16 halaman, 3 item bertingkat) |
| `_menu_item_object` | 19 | `page` 16 · `custom` 3 |
| `_elementor_data` pasangan `{"id":N,"url":…}` | 62 pasangan unik / 130 rujukan | 73 → lampiran diekspor · **57 → ID yang tidak ada di DB** |
| `_elementor_data` `"id":N` lain (indeks slide/objek) | 401 | bukan rujukan lampiran (mis. `"id":1..9` = indeks slide carousel) |
| Rujukan ID di konten (`"ids":[…]`, `wp:image {"id":N}`) | **0** | **tidak ada** — media di konten dirujuk lewat URL saja ✓ |
| **Rujukan ID menggantung** (`_thumbnail_id`/`_menu_item_object_id` tanpa target) | **0** | ✓ |

**Konsekuensi bagus:** karena konten tidak memakai ID lampiran sama sekali, seluruh media di konten akan ikut terselesaikan oleh penulisan-ulang URL (bukan pemetaan ID).

### 3.2 URL domain lama

**170 kemunculan** `https://smk10novemberkebumen.sch.id` — semuanya di **konten**: page 89 · post 44 · elementor_library 37. Contoh path: `/wp-content/uploads/2026/01/gallery-2.jpg`, `/wp-content/uploads/elementor/thumbs/img_kebumen_005-…webp`.
Karena URL-nya **absolut**, tanpa penulisan ulang situs staging akan menarik gambar dari **situs produksi** (ketergantungan pada mesin yang terkompromi + kebocoran pengunjung). ⇒ **wajib** di-rewrite.

### 3.3 Rencana pemetaan (belum dijalankan)

1. **Media (122 lampiran)** — berkas dimasukkan lebih dulu ke `wp-content/uploads` mempertahankan struktur tahun/bulan (di luar cakupan fase ini; langsung menyalin dilarang pada fase ini). Selanjutnya `wp media regenerate --yes` membangun turunan **dan** `_wp_attachment_metadata` yang sah.
2. **Pemetaan ID lama → baru** — WXR tidak membawa tabel pemetaan, jadi pemetaan dilakukan **deterministik** setelah impor dengan kunci **(post_type, post_name, post_date)**: `post_name` unik per tipe di WordPress, dan `post_date` memecah sisa seri. Diverifikasi dengan mencocokkan 174 item lama ke 174 item baru (jumlah per tipe harus sama).
3. **Remap setelah impor:** `_menu_item_object_id` (19), `_thumbnail_id` (10), `_elementor_data` `{"id":N}` yang sah (73 rujukan), `_elementor_screenshot` (1), dan URL di `_elementor_page_settings` (2). Catatan jujur: perlu **diverifikasi empiris** apakah importer bawaan sudah memetakan sebagian kunci ini; bila belum, langkah remap dijalankan. Semua ini **menulis ke DB staging ⇒ butuh persetujuan**.
4. **Penulisan ulang URL:** `wp search-replace 'https://smk10novemberkebumen.sch.id' 'http://localhost/staging/kebumen' --all-tables --precise` (setelah impor, dengan persetujuan). Termasuk varian `http://` dan `www.`.
5. **Rujukan mati** (57 rujukan/`_elementor_data` ke ID yang tidak ada di DB, mis. `id=444` ×13, `454`, `265`, `260`, `259`, `351`, `352`, `462`, `651`) — berasal dari **template kit pihak ketiga "Skoola"** yang gambarnya tidak pernah ada di instalasi ini. Rencana: dibiarkan kosong (default Elementor) atau diarahkan ke gambar pengganti; **tidak** dibuat tebakan pemetaan.

---

## 4. Tugas 4 — Rencana K-8 untuk 199 metadata yang ditunda

Hasil `tools/k8_plan.py`. **Validasi serialisasi: 199 dari 199 VALID**, **0 objek** (`O:` tidak ditemukan) — parser aman tanpa `unserialize()`.

| Kunci | Jml | Kelompok | Rekomendasi |
|---|---:|---|---|
| `_wp_attachment_metadata` | **121** | metadata lampiran | **Jangan impor — regenerasi** (`wp media regenerate`). Alasannya: metadata lama menunjuk turunan yang sebagian tak ada (§2b), sedangkan regenerasi menghasilkan ukuran sesuai tema baru. (Serialisasi valid, jadi mengimpor pun aman bila diinginkan.) |
| `_elementor_page_assets` | 24 | cache Elementor | **Buang** — turunan; dibangkitkan ulang. |
| `_menu_item_classes` | 19 | metadata menu | **Abaikan/impor** — praktis array kelas kosong (`a:1:{i:0;s:0:"";}`). |
| `_elementor_css` | 14 | cache Elementor | **Buang** — turunan. |
| `_elementor_page_settings` | **14** | **kit style / pengaturan halaman** | **Impor (prioritas tertinggi)** — di sinilah warna & typografi kit "Skoola". 2 nilainya memuat URL lama → perlu remap. |
| `_elementor_conditions` | 4 | Theme Builder | **Buang tanpa Pro / tunda bila Pro dipasang** — mengatur di mana templat Single/Archive tampil. |
| `_elementor_screenshot` | 1 | metadata Elementor | **Impor dengan remap, atau buang** (sekadar pratinjau editor). |
| `__wpdm_files`, `__wpdm_access` | 1 + 1 | plugin Download Manager | **Tunda** — hanya bila plugin itu dipasang. |

⇒ Bila rekomendasi diikuti: **impor 36 baris** (kit 14 + menu 19 + screenshot 1 + wpdm 2), **buang 38** (cache+conditions), **regenerasi 121**. Semua pemetaan ID pada kelompok ini mengikuti aturan §3.3 dan **ditulis hanya ke DB staging setelah disetujui**.

---

## 5. Tugas 5 — Konten & metadata yang perlu ditinjau lebih lanjut

XML valid **tidak** dianggap bukti aman; isi diperiksa terpisah. Hasil pemindaian seluruh berkas:

| Pola | Hasil |
|---|---|
| `<?php`, `eval(`, `base64_decode`, `<script` | **0** |
| `onerror=` / `onload=`, `javascript:`, `data:text/html` | **0** |
| `display:none` / `visibility:hidden` | **0** |
| Kata kunci spam (judi/slot/togel/gacor/maxwin/sbobet) | **0** |
| URL pemendek (bit.ly, t.co, is.gd, cutt.ly) | **0** |
| `wp2shell`, `sso_token`, `shellcode` | **0** |
| `<iframe>` | **1** — page `Kontak` (id 826) → `https://maps.google.com/maps?q=SMK%2010%20NOVEMBER%20KEBUMEN…output=embed` — **embed peta sah** |
| Shortcode nyata | **1** — `[gallery columns="3"]` pada id 1761, **tanpa atribut `ids`** → tidak perlu pemetaan ID |
| Domain eksternal | `wa.me` 6 (tautan WhatsApp sekolah) · `youtube.com` 1 · `youtu.be` 1 · `maps.google.com` 1 — **semuanya lazim untuk situs sekolah** |
| SVG inline (`_elementor_inline_svg`, lampiran 1920) | bersih (lihat §2) |

**Butir yang tetap perlu keputusan/tinjauan manusia:**
1. **170 URL absolut ke domain produksi** (§3.2) — harus di-rewrite, dan **sebelum** itu tidak boleh dibuka dari browser agar tidak memuat aset dari mesin produksi.
2. **27 berkas media hilang** (§2b) — perlu keputusan: terima gambar kosong, cari di arsip backup hosting, atau ganti gambar.
3. **57 rujukan mati ke ID kit "Skoola"** (§3.3 butir 5) — kosong secara desain, bukan indikasi kompromi.
4. **Berkas `.svg`** (456 KB): isinya bersih, tetapi SVG adalah format yang bisa memuat script di kemudian hari. Rekomendasi: pertahankan, namun bila situs dipublikasikan ulang, pertimbangkan konversi ke PNG dan larang unggah SVG tanpa sanitasi.
5. **`wpdmpro` (id 1930)** ikut diekspor beserta 2 meta Download Manager; hanya berguna bila plugin Download Manager dipasang — perlu keputusan.
6. **`elementor/screenshots/Elementor-post-screenshot_809_*.png`** dirujuk tetapi tidak ada di salinan bukti (bagian dari 27 berkas hilang).
7. Tidak ditemukan indikator kompromi baru di dalam WXR.

---

## 6. Kendala baru

| # | Kendala | Dampak |
|---|---|---|
| **K-10** | 27 berkas media yang dirujuk konten tidak ada di salinan bukti (19 asli + 8 turunan), tanpa record lampiran | gambar rusak setelah impor kecuali disediakan sumber lain |
| **K-11** | 170 URL absolut ke domain produksi di dalam konten | wajib di-rewrite; jangan dibuka sebelum itu |
| **K-12** | 57 rujukan `_elementor_data` ke ID yang tidak ada di DB (kit pihak ketiga "Skoola") | elemen gambar kosong; tidak dapat dipetakan |
| **K-13** | Pemetaan ID lama→baru bergantung pada pencocokan slug+tanggal pasca-impor | perlu langkah verifikasi + remap (menulis DB ⇒ butuh izin) |

---

## 7. Rekomendasi langkah berikutnya (menunggu persetujuan)

**Urutan yang saya sarankan:**
1. **Setujui/modifikasi rencana ini** (§3.3 + §4) — belum ada yang dijalankan.
2. **Salin 122 berkas media** dari salinan bukti ke `staging/kebumen/wp-content/uploads` dengan struktur tahun/bulan tetap, lalu catat SHA-256 sebelum/sesudah (langkah menyalin = butuh izin tersendiri).
3. **Impor WXR** ke pilot (`wp import`), lalu **verifikasi V-01…V-10** (jumlah per tipe, judul, status, menu, relasi).
4. **K-8 terpilih**: impor 36 baris (kit style prioritas) + `wp media regenerate` untuk 121 metadata lampiran.
5. **Remap ID** (§3.3 butir 3) dan **penulisan ulang URL** (§3.3 butir 4), lalu verifikasi tampilan halaman (Home, Kontak, Sejarah, Visi & Misi, berita) dan menu.
6. **Keputusan K-10** (media hilang): terima, cari di arsip backup hosting, atau ganti.
7. Setelah pilot terbukti, replikasi ke Tambun/Cibitung/Sragen (kini masih dilarang).

**Yang saya TIDAK lakukan tanpa izin:** menyalin media, impor, menulis ke DB staging, mengubah dump sumber, ekstraksi arsip backup hosting, koneksi ke produksi.

---

## 8. Lampiran — artefak pemeriksaan

| Berkas | Isi |
|---|---|
| `kebumen/export/_5b-media-verify.tsv` | 122 baris: path · ada · ukuran · SHA-256 · jumlah turunan · catatan |
| `kebumen/export/_5b-refmap.txt` | keluaran klasifikasi rujukan ID & keberadaan berkas |
| `tools/wxr_crosscheck.py`, `ref_detail.py`, `media_verify.py`, `ref_map.py`, `ref_media.py`, `k8_plan.py` | alat pemeriksaan (read-only) |
