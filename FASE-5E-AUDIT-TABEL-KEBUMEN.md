# FASE 5E — AUDIT TABEL DATABASE KEBUMEN SEBELUM IMPOR

**Tanggal:** 15 Oktober 2026 · **Lingkup:** audit saja (tidak ada impor, tidak ada perubahan)
**Target:** `smk10_staging_kebumen` (`C:\xampp\htdocs\staging\kebumen`, `http://localhost/staging/kebumen`)
**Sumber:** `D:\backup-database-smk10november\smkq8953_kebumen.sql` (95.834.894 B, SHA-256 `d91e42bf…`)
**Alat:** artefak audit Fase 4C.3/4C.6 (`_4c3-tabel*.tsv`, `_4c3-options.tsv`, `_4c3-ioc.tsv`, `_4c6-users.tsv`) + pengukuran baru `tools/fase5e_compare.py`, `tools/fase5e_opts.py`

**Bukti tidak ada perubahan selama fase ini:** tabel staging tetap **13**, 4 dump sumber **0 berubah**, `posts` tetap 183.

> **Catatan akurat tentang `options` (koreksi halus):** saat audit dimulai, `wp_options` berisi **169** baris; saat audit berakhir, **176**. Penambahan 7 baris itu **bukan dari tindakan audit** (audit hanya menjalankan `SELECT`), melainkan **cache/transien yang ditulis otomatis** oleh WordPress/Elementor/Astra ketika halaman dimuat: `elementor_atomic_cache_validity__{base,default,global,global_related,local,component-styles-related-posts}`, `_elementor_assets_data`, `_elementor_design_system_sync_css_meta`, `_transient_astra-theme-cron-test-ok(+timeout)`, `_site_transient_wp_theme_files_patterns-…(+timeout)`. Seluruhnya dapat dibuang dan dibuat ulang (13 nama cache dari sumber juga diklasifikasikan “buang” di §2.2).

---

## 1. Ringkasan kondisi database

| Ukuran | Sumber (dump Kebumen) | Staging | Selisih / catatan |
|---|---:|---:|---|
| Jumlah tabel | **138** | **13** | 125 tabel sumber tidak ada di staging |
| Tabel berisi data | **39** | 12 berisi + 1 kosong | 29 tabel berisi data tidak ada di staging |
| Tabel kosong di sumber | 99 | — | tidak perlu dibawa |
| `posts` | 642 | 183 (174 hasil impor + 9 bawaan/revisi) | sisa = revision 214 · oembed_cache 172 · customize_changeset 78 · auto-draft 2 · trash 1 (semuanya memang tidak dimigrasikan) |
| `postmeta` | 9.160 | 547 | 76–87 % di sumber adalah **yatim** (pemiliknya tidak diekspor) |
| `options` | 329 | 169 → 176 | **175 nama opsi** belum ada di staging; penambahan 7 baris saat audit = cache/transien otomatis (lihat catatan di atas) |
| `terms` / `term_taxonomy` / `term_relationships` | 16 / 16 / 49 | 12 / 12 / 39 | selisih sudah terjelaskan (lihat §2) |
| `users` / `usermeta` | 2 / 50 | 1 / 16 | akun lama **sengaja tidak diimpor** |
| `termmeta` | 4 | 0 | milik kategori Download Manager yang tidak dimigrasikan |
| `comments` / `commentmeta` / `links` | 0 / 0 / 0 | 1 / 0 / 0 | sumber tanpa komentar; 1 komentar di staging = bawaan WordPress |

**Kesimpulan ringkas:** seluruh **konten** (halaman, berita, lampiran, menu, templat Elementor) sudah dipulihkan dan lulus V-01…V-12 (Fase 5C/5D). Yang belum dipulihkan adalah **(a)** sebagian `options`, **(b)** akun lama & `usermeta`, **(c)** `termmeta` mikro, dan **(d)** seluruh **tabel plugin** yang pluginnya tidak dipasang di staging.

---

## 2. Tabel yang sudah lengkap dan yang belum dipulihkan

### 2.1 Sudah lengkap (terverifikasi Fase 5C/5D)
`posts`, `postmeta` (untuk objek yang dimigrasikan), `terms`, `term_taxonomy`, `term_relationships`, `users` (admin staging bersih), `options` (inti yang dibutuhkan + tema/Elementor terpilih), `comments`/`commentmeta` (sumber kosong), `links` (kosong).

Perbandingan taksonomi per slug — **semua yang terpakai sudah ada**:

| Taksonomi | Status |
|---|---|
| `nav_menu` (main-menu 12, footer-menu 7) | **OK** |
| `elementor_library_type` (6 tipe) | **OK** |
| `wp_theme` (astra, hello-elementor) | **OK** |
| `category` (`uncategorized` 10) | **OK** — selisih 3 relasi milik post **trash/auto-draft** yang memang tidak diekspor |

Selisih `elementor_library_type` (sumber archive 5 / footer 2 / header 3 vs staging 1/1/1) berasal dari **baris relasi yatim** di dump (menunjuk term yang tidak terpakai) — bukan templat yang hilang: kedelapan templat `elementor_library` yang dimiliki situs **sudah ada** di staging.

### 2.2 Belum dipulihkan

**(a) `options` — 175 nama belum ada**, terbagi:
- **27 sensitif** (kredensial/API key/token) → tidak boleh diimpor (§3)
- **13 cache/transien** → dibuang
- **135 lain**: mayoritas milik plugin yang tidak dipasang (Hostinger onboarding, Action Scheduler schema, AIOSEO/WPForms redirect, ElementsKit/Skyboot, WPDM, Jetpack), plus **opsi inti ber-prefix dump** yang padanannya sudah ada di staging:

| nama di dump | padanan di staging | status |
|---|---|---|
| `kbm_user_roles` | `wp_user_roles` | **ADA** |
| `kbm_page_for_privacy_policy` | `wp_page_for_privacy_policy` | **ADA** |
| `kbm_attachment_pages_enabled` | `wp_attachment_pages_enabled` | **ADA** |
| `kbm_force_deactivated_plugins` | `wp_force_deactivated_plugins` | **ADA** |
| `kbm_notes_notify` / `kbm_calendar_block_has_published_posts` | `wp_…` | **ADA** |

**(b) `users` (2) + `usermeta` (50)** — tidak diimpor; `usermeta` memuat **`session_tokens`** (2 baris) dan preferensi admin.

**(c) `termmeta` (4 baris)** — semuanya milik kategori `wpdmcategory: kalender-akademik` (`__wpdm_icon`, `__wpdm_access`, `__wpdm_style`, `__wpdm_pagestyle`) → milik paket Download Manager yang tidak dimigrasikan.

**(d) 29 tabel berisi data milik plugin yang tidak dipasang:**

| Kelompok | Tabel (jml baris) | Plugin |
|---|---|---|
| WooCommerce | `gla_budget_recommendations` 4.230 · `wc_admin_note_actions` 93 · `woocommerce_order_itemmeta` 80 · `wc_admin_notes` 65 · `wc_product_meta_lookup` 18 · **`woocommerce_sessions` 16** · `woocommerce_order_items` 8 · `wc_category_lookup` 3 · `wc_product_download_directories` 2 · `wc_tax_rate_classes` 2 | WooCommerce (tanpa produk/pesanan — hanya catatan admin & sesi) |
| MailPoet | `mailpoet_newsletter_templates` 76 · `mailpoet_migrations` 36 · `mailpoet_newsletter_option_fields` 31 · `mailpoet_scheduled_tasks` 15 · `mailpoet_settings` 13 · `mailpoet_segments` 3 · **`mailpoet_subscribers` 1** · `mailpoet_subscriber_segment` 1 | MailPoet |
| WPForms / Formidable | `wpf_meta_values` 28 · `wpf_meta_keys` 12 · `wpf_filters` 6 · `wpforms_tasks_meta` 5 · `frmt_form_reports` 1 | Form plugin |
| Action Scheduler | `actionscheduler_logs` 5 · `actionscheduler_actions` 3 · `actionscheduler_groups` 2 | antrean pekerjaan |
| Jetpack | `jetpack_sync_queue` 3 · `jet_taxonomies` 2 | cache sinkronisasi |
| CartFlows | `cartflows_ca_cart_abandonment` 1 | keranjang terbengkalai (PII) |
| Elementor | `e_events` 1 | tabel ada di staging tetapi 0 baris |

**Tipe post di sumber yang tidak dimigrasikan:** `revision` 214 · `oembed_cache` 172 · `customize_changeset` 78 · `auto-draft` 4 · **`trash` 1** ("PURNAWIDYA SMP & SMK 10 NOVEMBER KEBUMEN 2025/2026") · `envato_tk_import` 1 · `wpdmpro` 1 (paket Download Manager) — masing-masing dengan alasan.

---

## 3. Temuan keamanan (tanpa membocorkan rahasia)

1. **Tidak ada indikator kompromi di database Kebumen.** `_4c3-ioc.tsv` **kosong** untuk Kebumen; tidak ada `wp2shell_*`, `sso_token`, `_codehidden`, `fm_key` di `options` Kebumen (berbeda dari Tambun/Cibitung/Sragen).
2. **Akun lama (2) semuanya mencurigakan** — tidak ada yang diimpor:
   - id 1 `ad***`, terdaftar **2025-10-30 09:08:29** (stempel waktu yang sama dengan klaster provisi lintas-situs), domain email sekolah;
   - id 2 `ad***`, terdaftar 2026-08-13 14:40:23, domain email **`goole.com`** (menyerupai `google.com`).
   Staging hanya memiliki **1 akun admin bersih**.
3. **`usermeta` memuat `session_tokens` (2 baris)** → tidak boleh diimpor (token sesi lama).
4. **27 opsi sensitif** di sumber (tidak satu pun diimpor, nilainya tidak pernah ditampilkan): antara lain kunci API Elementor Pro (Mailchimp, Drip, ActiveCampaign, GetResponse, ConvertKit, MailerLite, Stripe test/live, reCAPTCHA v2/v3), Google Maps API key, Facebook App ID, `elementor_connect_site_key`, lisensi Elementor Pro, `ai1wm_secret_key`, `__wpdm_enc_key`, dan `mnx_wp_auth`.
   ⇒ **Rekomendasi tegas: rotasi seluruh kredensial tersebut** karena pernah tersimpan di situs yang terkompromi.
5. **`elementor_unfiltered_files_upload = 1` di situs lama** — mengizinkan unggahan berkas tanpa penyaringan tipe. Ini **tidak boleh diimpor**; biarkan nonaktif (staging saat ini tidak memilikinya).
6. **Data pribadi di tabel plugin yang tidak diimpor** (harus tetap tidak diimpor): `mailpoet_subscribers` 1 baris, `woocommerce_sessions` 16 baris, `cartflows_ca_cart_abandonment` 1 baris.
7. Data lain yang tampak sensitif: `mailserver_pass` (konfigurasi SMTP lama) → tidak diimpor.
8. Opsi yang sudah benar di kedua sisi: `users_can_register = 0`, `default_role = subscriber`, `blog_public = 1`.

---

## 4. Rekomendasi per tabel/kelompok data

| Kelompok | Verdict | Alasan |
|---|---|---|
| `posts`, `postmeta`, `terms`, `term_taxonomy`, `term_relationships` | **Sudah selesai** — jangan impor ulang | konten lengkap; impor ulang = duplikasi |
| `comments`/`commentmeta`/`links` | **Abaikan** | sumber kosong |
| **`options` (whitelist 8 nama)** | **IMPOR selektif** | memulihkan bahasa, zona waktu, format tanggal, ikon situs, perilaku Elementor — lihat rencana §Rencana |
| `options` sensitif (27) | **JANGAN impor + rotasi** | kredensial tersimpan di situs terkompromi |
| `options` cache/transien (13) | **Buang** | dibuat ulang |
| `options` plugin/hostinger/ElementsKit/WPDM/Jetpack (≈122) | **Abaikan** | plugin tidak dipasang |
| `usermeta` (50) | **JANGAN impor** | `session_tokens`, preferensi admin tidak relevan |
| `users` (2) | **Rekonstruksi manual** | akun tidak tepercaya; buat akun baru setelah verifikasi sekolah |
| `termmeta` (4) | **Abaikan** | milik `wpdmcategory` Download Manager |
| `post_tag` (2 kosong), `post_format` (1), `category juara-3-pencak-silat` | **Abaikan** | tidak terpakai / milik post trash-auto-draft |
| `trash` post "PURNAWIDYA …" | **Abaikan (tetap terhapus)** | memang dihapus sekolah; keputusan bila ingin dipulihkan |
| WooCommerce (10 tabel) | **Abaikan** | tidak ada produk/pesanan; 16 sesi berisi PII |
| MailPoet (8 tabel) | **Abaikan** | plugin tidak dipakai; 1 langganan = data pribadi |
| WPForms/Formidable (5 tabel) | **Abaikan** | tidak ada shortcode form di konten |
| Action Scheduler (3) | **Abaikan** | antrean lama, akan dibuat ulang bila plugin dipasang |
| Jetpack (2), CartFlows (1), GLA (1), `e_events` (1) | **Abaikan** | cache/analitik/PII |

**Catatan ID/relasi:** karena semua data yang akan diimpor adalah **opsi** (bukan objek ber-ID), **tidak ada risiko konflik ID**. Satu-satunya nilai ber-ID adalah `site_icon = 1390` — sudah diverifikasi: lampiran 1390 ada di staging (`2026/05/1779123407138.png`) dan **berkasnya ada di disk**.

---

## 5. Ketergantungan data & pencegahan duplikasi

1. **Whitelist, bukan SQL massal.** Impor dilakukan **per nama opsi** dengan `update_option()` (idempoten) — bukan `INSERT` tabel.
2. **Tidak menyentuh tabel yang sudah terisi konten** (`posts`, `postmeta`, `term*`).
3. **Terjemahan prefix dump → staging:** `kbm_*` → `wp_*`. Enam opsi inti ber-prefix sudah diverifikasi ada padanannya (§2.2) sehingga **tidak perlu ditulis ulang**.
4. Nilai yang aman untuk `nav_menu_options` dan `elementor_cpt_support` adalah array PHP → ditulis sebagai **array** via `update_option` (hindari serialisasi ganda), seperti prosedur K-8 yang sudah terbukti.
5. **Snapshot & rollback:** `checkpoint-05-pre-5e.sql` dibuat sebelum penerapan; snapshot nilai lama setiap opsi ditulis ke berkas teks.
6. **Aturan berhenti:** bila satu nama opsi tidak dapat dipastikan maknanya, jangan diterapkan.

---

## 6. Langkah validasi setelah impor (bila disetujui)

1. `V-01…V-11` (skrip `tools/verify_v.py`) → harus tetap **11 LULUS**.
2. `V-12` aset CSS/JS harus tetap 200.
3. Cek bahasa & waktu: `WPLANG=id_ID` terbaca, tanggal situs tampil dalam format **d/m/Y** dan zona **Asia/Jakarta**; berkas bahasa `id_ID.mo` sudah tersedia di `wp-content/languages/`.
4. Cek ikon situs (favicon) tampil dan lampiran 1390 tidak menjadi gambar rusak.
5. Cek IoC: `options` dan `postmeta` bebas `wp2shell/sso_token/_codehidden/fm_key` = 0.
6. Cek JSON Elementor tetap valid (20 nilai) dan tidak ada URL domain produksi.
7. Pastikan jumlah tabel staging tetap 13 dan tidak ada tabel plugin yang ikut masuk.

---

## 7. Keputusan yang memerlukan persetujuan pengguna

| # | Keputusan | Usulan saya |
|---|---|---|
| 1 | `elementor_unfiltered_files_upload` | **Jangan impor** (keamanan) |
| 2 | Akun lama (2) — rekonstruksi? | **Tidak diimpor**; buat akun baru sesuai kebutuhan sekolah |
| 3 | Rotasi 27 kredensial (API key/lisensi) | **Lakukan di sisi produksi/sekolah** |
| 4 | `blogname` | staging memakai penanda “(STAGING pemulihan)”; lepaskan hanya bila Anda minta |
| 5 | Post `trash` “PURNAWIDYA …” | biarkan terhapus (keputusan bila ingin dipulihkan) |
| 6 | Tabel plugin (WooCommerce/MailPoet/WPForms/Action Scheduler/Jetpack/CartFlows/GLA) | **tidak diimpor**; konfirmasi |
| 7 | Data pribadi di dump (`mailpoet_subscribers`, `woocommerce_sessions`, `cartflows_ca_cart_abandonment`) | tidak diimpor; pertimbangkan penghapusan dari arsip |
| 8 | Dari Fase 5D yang masih terbuka | `2025/10/logo_SMK.png`, pengganti widget/templat Pro, peringatan PHP kit |

---

## RENCANA IMPOR YANG DAPAT DITINJAU (belum dieksekusi)

**Metode:** `update_option()` per nama, satu transaksi logis, dengan snapshot + checkpoint. **Whitelist 8 opsi:**

| # | Opsi | Nilai (sumber) | Alasan |
|---|---|---|---|
| 1 | `WPLANG` | `id_ID` | bahasa situs; paket bahasa sudah ada |
| 2 | `timezone_string` | `Asia/Jakarta` | tanggal/jam benar (staging kini `gmt_offset=0`) |
| 3 | `date_format` | `d/m/Y` | format tanggal sekolah |
| 4 | `time_format` | `H:i` | format jam 24 jam |
| 5 | `site_icon` | `1390` | favicon; lampiran & berkas terverifikasi ada |
| 6 | `elementor_cpt_support` | `["page"]` | batasi Elementor ke halaman (seperti situs lama) |
| 7 | `elementor_google_font` | `1` | kit memakai Google Fonts (tipografi kit) |
| 8 | `nav_menu_options` | `{"auto_add":[]}` | konfigurasi menu klasik |

**Tambahan opsional (keputusan Anda):** `blogdescription` sudah diimpor di 5D ✓ · `blogname` tanpa penanda staging.

**Tidak termasuk (tegas):** 27 opsi sensitif · `elementor_unfiltered_files_upload` · 13 cache · ±122 opsi plugin · seluruh `usermeta` · 2 akun lama · `termmeta` · 29 tabel plugin · post trash/auto-draft.

**Perkiraan dampak:** 8 baris opsi, tanpa perubahan skema, tanpa konflik ID, dapat dibatalkan dengan `checkpoint-05-pre-5e.sql` atau dengan mengembalikan snapshot nilai lama per opsi.

**Setelah disetujui, langkah yang akan saya jalankan:** (1) `checkpoint-05-pre-5e.sql`, (2) tulis snapshot nilai lama, (3) terapkan 8 opsi, (4) jalankan `V-01…V-12` + pemeriksaan bahasa/zona waktu/favicon, (5) laporkan perubahan dan hasil uji.

**Status:** audit selesai, **belum ada perubahan apa pun** pada database/tabel/konten/opsi/berkas/konfigurasi.
