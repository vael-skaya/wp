# FASE 5E.1 — PENERAPAN 8 OPSI WHITELIST (Kebumen)

**Tanggal:** 15 Oktober 2026 · **Status:** selesai, seluruh pengujian LULUS
**Target:** `C:\xampp\htdocs\staging\kebumen` · DB `smk10_staging_kebumen` · `http://localhost/staging/kebumen`
**Checkpoint sebelum perubahan:** `recovery-work\staging\checkpoints\checkpoint-05-pre-5e1.sql` — 2.293.789 B, SHA-256 `d894a6d44c684dcc…`
**Snapshot nilai lama:** `recovery-work\staging\backup-5e1\opt-*.txt` (8 berkas)

Batasan yang dipatuhi: hanya staging Kebumen · tidak ada tabel tambahan · tidak ada akun lama/usermeta/opsi sensitif/cache/tabel plugin/`elementor_unfiltered_files_upload` · produksi, tiga situs lain, `goly1264_ams`, dump sumber **tidak disentuh**.

---

## 1. Verifikasi sebelum perubahan (nilai, tipe, dan sumber)

Nilai sumber dibaca ulang dari dump dan dibandingkan dengan yang tercatat di laporan audit, sekaligus dengan keadaan staging saat itu:

| Opsi | Nilai SUMBER | Cocok laporan | Staging sebelum | Tipe ditulis |
|---|---|---|---|---|
| `WPLANG` | `id_ID` | ✔ | _(tidak ada)_ | string |
| `timezone_string` | `Asia/Jakarta` | ✔ | `""` (kosong) | string |
| `date_format` | `d/m/Y` | ✔ | `j F Y` | string |
| `time_format` | `H:i` | ✔ | `g:i a` | string |
| `site_icon` | `1390` | ✔ | `0` | string |
| `elementor_cpt_support` | `['page']` | ✔ | _(tidak ada)_ | array |
| `elementor_google_font` | `1` | ✔ | _(tidak ada)_ | string |
| `nav_menu_options` | `{'0': false, 'auto_add': {}}` | ✔ | _(tidak ada)_ | array |

### 1.1 Temuan saat pemeriksaan (transparansi — sebagaimana diwajibkan batasan #5)
Pemeriksaan pertama **berhenti otomatis** dan menandai **3 item**: `site_icon`, `elementor_cpt_support`, `nav_menu_options`. Setelah ditelusuri, ketiganya **bukan perbedaan nilai, melainkan perbedaan notasi tipe**:
1. `site_icon` — sumber menyimpan **string** `"1390"`, laporan menuliskannya tanpa tanda kutip (dibaca alat sebagai bilangan). Tindakan: diterapkan sebagai **string** (paling setia).
2. `elementor_cpt_support` — sumber menyimpan **peta PHP** `a:1:{i:0;s:4:"page";}`, laporan menuliskannya sebagai list `["page"]` (semakna).
3. `nav_menu_options` — sumber menyimpan `a:2:{i:0;b:0;s:8:"auto_add";a:0:{}}`, yaitu **memuat elemen tambahan `0 => false`** yang tidak ditulis di laporan (laporan hanya menyebut `{"auto_add":[]}`).
Tindakan: perbandingan diperbaiki agar sadar-semantik PHP (array ber-kunci 0..n-1 ≡ list) dan **nilai yang diterapkan adalah struktur PERSIS dari sumber**, bukan versi ringkas laporan. Setelah itu kedelapan opsi **COCOK seluruhnya**.

---

## 2. Perubahan yang diterapkan (8 opsi, via `update_option`, tipe benar)

| # | Opsi | Jenis | Nilai lama → baru | Tipe tersimpan |
|---|---|---|---|---|
| 1 | `WPLANG` | **insert** | _(tidak ada)_ → `id_ID` | string |
| 2 | `timezone_string` | update | `""` → `Asia/Jakarta` | string |
| 3 | `date_format` | update | `j F Y` → `d/m/Y` | string |
| 4 | `time_format` | update | `g:i a` → `H:i` | string |
| 5 | `site_icon` | update | `0` → `1390` | string |
| 6 | `elementor_cpt_support` | **insert** | _(tidak ada)_ → array(`page`) | array → `a:1:{i:0;s:4:"page";}` |
| 7 | `elementor_google_font` | **insert** | _(tidak ada)_ → `1` | string |
| 8 | `nav_menu_options` | **insert** | _(tidak ada)_ → array(0=>false, `auto_add`=>array()) | array → `a:2:{i:0;b:0;s:8:"auto_add";a:0:{}}` |

**Bukti tipe tersimpan** (dibaca langsung dari `wp_options`, nilainya **identik dengan sumber**):
```
elementor_cpt_support   a:1:{i:0;s:4:"page";}
nav_menu_options        a:2:{i:0;b:0;s:8:"auto_add";a:0:{}}
```
Semua nilai array ditulis sebagai **array PHP** melalui `update_option()` sehingga **tidak terjadi serialisasi ganda** (prosedur sama seperti K-8).

**Perubahan lain: tidak ada.** Tidak ada tabel baru (tetap **13**), tidak ada perubahan skema, tidak ada penyuntingan berkas, tidak ada perubahan `posts` (tetap 183) atau pengguna (tetap 1). Jumlah opsi **176 → 180** = +4, persis sesuai hitungan (4 insert + 4 update).

---

## 3. Hasil pengujian setelah penerapan

| Uji | Hasil |
|---|---|
| **V-01** jumlah objek per tipe | **LULUS** |
| **V-02** judul + slug halaman/berita | **LULUS** (23 halaman/berita) |
| **V-03** lampiran & berkasnya | **LULUS** (122/122) |
| **V-04** `_thumbnail_id` → lampiran sah | **LULUS** (10) |
| **V-05** menu (19 item, 2 menu, 19 relasi) | **LULUS** |
| **V-06** URL domain produksi (berskema) | **LULUS** (0; 3 baris alamat email dipertahankan) |
| **V-07** data Elementor + kit style | **LULUS** (JSON valid 20; invalid 3 = revisi lama; kit 158 elemen) |
| **V-08** halaman inti HTTP 200 | **LULUS** (12 halaman) |
| **V-09** bebas IoC (opsi & meta) | **LULUS** (0/0) |
| **V-10** HTML bebas aset domain produksi | **LULUS** (0) |
| **V-11** media tampil dapat dimuat | **LULUS** (**304 rujukan diperiksa, 0 gagal**) |
| **V-12** aset CSS/JS termuat | **LULUS** (0 gagal pada 5 halaman) |
| **Bahasa** | `WPLANG=id_ID`; HTML `<html lang="id">` ✔ |
| **Zona waktu & format** | `timezone_string=Asia/Jakarta`; tanggal situs **15/10/2026 17:41** (d/m/Y H:i, WIB) ✔ |
| **Favicon** | 3 tautan tampil: `rel="icon"` 32×32 & 192×192, `apple-touch-icon` → `2026/05/1779123407138.png` ✔ |
| **JSON Elementor** | 20 nilai valid; 3 invalid hanya pada **revisi** lama (tidak dirender) |
| **Indikator kompromi** | 0 pada `options` & `postmeta`; 0 opsi sensitif baru (`api_key`/`secret`/`mnx_wp_auth`/`ai1wm_secret_key`/`__wpdm_enc_key`/`elementor_unfiltered_files_upload`) |
| **Keadaan menyeluruh** | tabel **13**, `posts` **183**, `users` **1**, dump sumber **4 berkas / 0 berubah**, `goly1264_ams` **29 tabel**, staging hanya `kebumen` |

**Tidak ada pengujian yang gagal** ⇒ batasan #8 (berhenti bila ada kegagalan) tidak terpicu; **tidak ada rollback yang dijalankan**.

**Catatan teknis kecil:** `gmt_offset` tetap `0` dan **tidak diubah** karena berada di luar 8 opsi yang disetujui; WordPress mengabaikannya selama `timezone_string` terisi (terbukti: jam situs menunjukkan WIB) ✔

---

## 4. Status rollback

| Aset | Lokasi | Isi |
|---|---|---|
| Checkpoint database penuh | `checkpoint-05-pre-5e1.sql` (2.293.789 B, SHA-256 `d894a6d44c684dcc…`) | keadaan **sebelum** 5E.1 |
| Snapshot per opsi | `recovery-work\staging\backup-5e1\opt-<nama>.txt` (8 berkas) | nilai lama tiap opsi (JSON) |
| Checkpoint sebelumnya | `checkpoint-04-pre-5d.sql`, `checkpoint-03-post-recovery.sql`, `checkpoint-02-post-import.sql`, `checkpoint-01-pre-import.sql` | tahap sebelumnya |

**Cara rollback penuh (bila diminta):**
`mysql -u root smk10_staging_kebumen < recovery-work\staging\checkpoints\checkpoint-05-pre-5e1.sql`
**Cara rollback selektif (lebih halus):** kembalikan tujuh opsi dari snapshot, dan hapus empat opsi yang baru dibuat (`WPLANG`, `elementor_cpt_support`, `elementor_google_font`, `nav_menu_options`) — perintah tidak dijalankan dan akan disiapkan lebih dulu bila Anda meminta.

⚠️ Sesuai batasan #8, **rollback tidak dilakukan otomatis**. Semua pengujian lulus sehingga tidak ada dasar untuk rollback.

---

## 5. Yang masih terbuka (tidak dikerjakan di fase ini)

Menunggu keputusan Anda, tanpa tindakan tambahan dari saya:
1. `2025/10/logo_SMK.png` (2 rujukan di pengaturan kit + templat Header/Footer) — berkasnya tidak ada di mana pun.
2. Pengganti widget/templat **Elementor Pro** (form Kontak, Single/Archive/Header/Footer).
3. Peringatan PHP non-fatal `body_background_image_tablet` dari data kit lama.
4. 250 salinan berkas `-1` di `uploads` (sengaja tidak dihapus).
5. Rotasi 27 kredensial yang tersimpan di dump lama (tindakan di sisi produksi/sekolah).
6. Data pribadi di dump (`mailpoet_subscribers`, `woocommerce_sessions`, `cartflows_ca_cart_abandonment`) — tidak diimpor.
