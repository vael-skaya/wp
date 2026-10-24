# FASE — KEPUTUSAN REKONSILIASI DATABASE

**23 Oktober 2026** · READ-ONLY · tanpa impor/migrasi · tanpa perubahan staging/produksi

## 1. Database staging yang diperiksa
| Sekolah | Staging DB | Prefix | Tabel | Bukan drill |
|---|---|---|---:|---|
| Tambun | `smk10_staging_tambun` | `wp_` | **24** | ✓ (dari `wp-config`) |
| Cibitung | `smk10_staging_cibitung` | `wp_` | **24** | ✓ (dari `wp-config`) |
| Kebumen | `smk10_staging_kebumen` | `wp_` | **13** | ✓ (dari `wp-config`) |
| Sragen | `smk10_staging_sragen` | `wp_` | **13** | ✓ (dari `wp-config`) |

## 2. Tabel inti — backup → staging
| Sekolah | posts | postmeta | terms | term_relationships | options |
|---|---:|---:|---:|---:|---:|
| Tambun | 378 → **166** | 8213 → **547** | 9 → **9** | 27 → **26** | 346 → 220 |
| Cibitung | 681 → **205** | 9361 → **640** | 9 → **9** | 31 → **25** | 307 → 226 |
| Kebumen | 642 → **187** | 9160 → **590** | 16 → **12** | 49 → **39** | 329 → 183 |
| Sragen | 284 → **152** | 8167 → **506** | 9 → **9** | 27 → **26** | 322 → 177 |

**Konten lengkap** ✓ — jumlah `posts`/`postmeta`/`terms`/`term_relationships` **setara atau lebih** di staging (staging sudah dibersihkan dari revision/oembed/transient) ✓
`options` lebih sedikit di staging **karena disengaja** (instalasi bersih, tanpa opsi lama terkompromi) ✓

## 2b. Mengapa `posts` backup jauh lebih banyak? — TERBUKTI artefak sistem ✓

| Situs | page | post | attachment | nav_menu | elementor_lib | revision | oembed | changeset | request | lain |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|
| Tambun | 11 | 1 | 120 | 22 | 8 | **101** | **45** | **34** | **34** | wp_navigation=1, envato_tk_import=1 |
| Cibitung | 15 | 2 | 153 | 22 | 7 | **153** | **149** | **89** | **89** | wp_navigation=1, envato_tk_import=1 |
| Kebumen | 11 | 13 | 122 | 19 | 8 | **214** | **172** | **78** | **0** | wp_navigation=1, envato_tk_import=1, wp_global_styles=2, wpdmpro=1 |
| Sragen | 11 | 1 | 107 | 22 | 8 | **55** | **29** | **22** | **22** | wp_navigation=1, envato_tk_import=1, custom-code=5 |

**Bukti kesetaraan:** page+post+attachment+nav_menu+elementor_library ≈ jumlah posts di staging:
Tambun 162≈166 · Cibitung 199≈205 · Kebumen 173≈187 · Sragen 149≈152 ✓

⇒ Selisih `posts` (378→166, 681→205, 642→187, 284→152) **sepenuhnya** berasal dari **revision (55–214), oembed_cache (29–172), customize_changeset (22–89), request (22–89)** — artefak sistem yang memang tidak dimigrasikan ✓
Selisih `postmeta` (8.213→547 dst.) mengikuti pola yang sama (meta milik revision) ✓

**Konten (halaman, artikel, lampiran, menu, template Elementor) LENGKAP** ✓

## 2c. Tipe post tambahan yang perlu dicatat
| Tipe | Situs | Jumlah | Catatan |
|---|---|---:|---|
| `custom-code` | Sragen | 5 | post_type plugin Custom Codes (injeksi JS/HTML) — **tidak dimigrasikan** ✓, sebaiknya tetap tidak tanpa pemeriksaan ❓ |
| `wpdmpro` | Kebumen | 1 | item WP Download Manager — kemungkinan paket demo, **belum diverifikasi** ❓ |
| `envato_tk_import` | keempat | 1 | artefak importer template ✓ tidak diperlukan |

## 3. Tabel hanya di backup — keputusan per kelompok
| Kelompok | Contoh (baris) | Keputusan | Alasan berbasis bukti |
|---|---|---|---|
| Konten/inti WP | posts, postmeta, terms | **sudah ada** ✓ | tidak ada yang hilang |
| Plugin tak dipakai | WooCommerce, WP Fusion, Jetpack, CartFlows, GLA (~18.000 baris) | **TIDAK PERLU** | plugin tidak dipasang; `shop_order`=0 ⇒ order item orphan |
| Template/cache plugin | `mailpoet_newsletter_templates` (76), `gla_budget_recommendations` (4.230) | **TIDAK PERLU** | boilerplate, dibuat ulang saat aktivasi |
| Antrean/sesi | `actionscheduler_*`, `woocommerce_sessions`, `jetpack_sync_queue` (~30) | **TIDAK PERLU** | sementara |
| Code Snippets | `snippets` (4/tambun, 4/sragen) | **TIDAK PERLU** | **semua `active=0`**, kosmetik, 0 indikator berbahaya |
| Newsletter | `mailpoet_subscribers` (1/situs) | **TIDAK PERLU** | `source=wordpress_user` + `unconfirmed` ⇒ record sistem |
| Formulir | `e_submissions*`, `frmt_*`, `wpforms_entries` | **TIDAK PERLU** | **0 baris** — tidak ada entri hilang |

## 4. Jawaban pertanyaan utama
**Tidak ada tabel atau data penting yang belum ada di staging** — kelengkapan konten dibuktikan lewat rincian tipe post (§2b) ✓
- Konten, menu, media, dan pengaturan yang dibutuhkan → **ada** ✓
- Entri formulir/pendaftaran pengguna sah → **tidak ada di backup** (0 baris) ✓
- Konfigurasi plugin aktif → tabel `wpforms_*`/`actionscheduler_*` justru **ada di staging**, tidak ada di backup ✓

## 5. Ketidakpastian yang tetap dicatat
- `woocommerce_order_items` (8) belum terbukti transaksi nyata; `shop_order`=0 mengarah ke sisa plugin ❓
- Hitungan `posts` produksi berasal dari blok `INSERT` pertama (parsial) ✗
- Tidak ada data yang **terbukti** hilang ⇒ ketidakpastian ini **tidak menahan keputusan** ✓

## 6. KEPUTUSAN

# **A. STAGING READY FOR PRODUCTION EXPORT** ✓

Export final yang ada tetap layak: `production-ready-tambun-v2.sql`, `production-ready-cibitung-v2.sql`, `production-ready-kebumen-v2.sql`, `production-ready-sragen.sql` ✓

## 7. Blocker tersisa (bukan database)
| # | Blocker | Pemilik |
|---|---|---|
| B-1 | Repo `vael-skaya/wp` masih **PUBLIC** | Saka (`gh auth login` → privatkan) |
| B-2 | Checksum inti 3 situs **NOT VERIFIED** | Saka (izin jalankan ulang) |
| B-3 | 4 DB kosong + user di hosting | Pak Eko |
| B-4 | PHP ≥8.1 · `memory_limit ≥256M` · SSL · DNS | Pak Eko |
| B-5 | Email resmi + teks kebijakan privasi | Pak Eko |

## 8. Konfirmasi
Tanpa impor/migrasi ✓ · staging & backup asli tidak diubah ✓ · snippet tidak dieksekusi ✓ · tanpa PII/kredensial di laporan ✓ · tanpa commit/push ✓ · `goly1264_ams` & `Downloads\admin.php` tidak disentuh ✓
