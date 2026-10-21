# FASE 8E — QA VISUAL + INTERAKTIF, LOCALE SRAGEN, PRECHECK PRODUKSI

**Tanggal:** 21–22 Oktober 2026 · Perubahan: **hanya locale Sragen (staging)** · Produksi/Rumahweb tidak disentuh · Tanpa push/commit

## 1. Ringkasan per situs
| Situs | Perubahan fase ini | QA otomatis | QA visual/browser |
|---|---|---|---|
| Tambun | tidak ada | PASS (HTTP, 8A/8B) | **BLOCKED** (browser timeout) |
| Cibitung | tidak ada | PASS (HTTP, 8D) | **BLOCKED** |
| Kebumen | tidak ada | PASS (HTTP, 8D) | **BLOCKED** |
| **Sragen** | **locale `en_US` → `id_ID`** | PASS | **PASS sebagian (3 viewport)** |

## 2. Locale Sragen — SELESAI ✓
| Bukti | Nilai |
|---|---|
| Sebelum | `WPLANG` kosong → `get_bloginfo('language')` = **en-US** |
| Checkpoint baru | `checkpoint-8e-pre-locale-id-id-20261022-002520.sql` · **829950 B** · SHA `ee8b3fe65e0381be12fb9fa048d11dd57f8b228708a04965f230196b719400c2` |
| Berkas bahasa | **77 berkas `id_ID`** disalin **lokal** dari staging Kebumen (tanpa unduh dari internet) |
| Perubahan | `wp option update WPLANG id_ID` (satu opsi) — **tidak** ada penggantian massal teks/URL/konten |
| Sesudah | `WPLANG`=`id_ID` · `get_locale()`=`id_ID` · `<html lang="id">` · `get_bloginfo('language')`=`id` ✓ |
| Integritas konten | posts **152 → 152**, halaman publish **11 → 11**, `site_logo` **1452 → 1452** (identik) ✓ |
| Checkpoint lama | **tidak ditimpa** ✓ |

## 3. Matriks QA
### 3a. Terukur di browser (hanya Sragen yang berhasil)
| Viewport | Overflow | Elemen meluber | Logo | Toggle mobile | Resource 4xx | Console error | `lang` |
|---|---:|---:|---:|---:|---:|---|---|
| 360 | **0** | 0 | 48 px | ada | **0** | **[]** | **id** ✓ |
| 768 | **0** | 0 | 48 px | ada | **0** | **[]** | **id** ✓ |
| 1366 | −15 (scrollbar) | 0 | 72 px | — | **0** | **[]** | **id** ✓ |

⚠️ **Caveat gambar:** pada ketiga viewport, **14 dari 17 `<img>`** melaporkan `naturalWidth=0`. Kemungkinan besar **lazy-load** (gambar di bawah lipatan belum dimuat), tetapi **saya tidak memverifikasi** ini karena pemeriksaan ini tidak mengecualikan `loading="lazy"`. **Status: BELUM TERVERIFIKASI** — bukan klaim "14 gambar rusak".

### 3b. Status per kategori
| Kategori | Status | Bukti / alasan |
|---|---|---|
| A. Smoke HTTP & redirect (semua halaman publish) | **PASS** | 8A/8B/8D: Tambun 11, Cibitung 12, Kebumen 11, Sragen 11 halaman → 200; `/home` → 301 ke `/` |
| B. Responsif 360/390/412/768/1366 | **PASS (Sragen 360/768/1366)** · **BLOCKED (3 situs lain)** | Chrome timeout berulang pada Tambun/Cibitung/Kebumen |
| C. Console JS / failed requests | **PASS (Sragen)** · **BLOCKED (3 situs)** | Sragen: 0 error, 0 resource ≥400 |
| D. Navigasi: klik buka/tutup, aria-expanded, keyboard | **NOT TESTED** ✗ | toggle terdeteksi ada (Sragen), tetapi **klik tidak berhasil diverifikasi** pada fase ini |
| E. Formulir | **NOT TESTED** ✗ | tidak ada form aktif yang terdeteksi; tidak ada pengiriman dicoba |
| F. `mailto:`/`tel:`/WhatsApp | **PASS (statis)** | 8C: 5 `mailto:` terverifikasi; `tel:`/WA ada pada markup |
| G. Halaman kontak/beranda/privasi | **PASS (HTTP)** · privasi tetap **draft** ✓ | tidak dipublikasikan |
| H. Trash Kebumen ID 2 & Cibitung 1275/1281/1341 | **PASS** | 8D: status `trash`, URL 404 ✓; **tidak** dihapus permanen |
| I. Aset setelah locale berubah | **PASS (Sragen)** | 0 resource 4xx; aset lokal resolvable |

## 4. Bug & perbaikan
| # | Temuan | Status |
|---|---|---|
| B-1 | Sragen `en_US` | **SELESAI** ✓ (di atas) |
| B-2 | `WP_MEMORY_LIMIT` 40M | **PRECHECK** — target produksi ≥256M |
| B-3 | Kebijakan privasi template | **DRAF tersedia**, belum dipublikasikan |
| B-4 | `elementor-1252` publish yatim | **dipertahankan** (sesuai instruksi), tidak diubah ✓ |
| B-5 | Item `#` / `#akademik` | **tidak diubah** ✓ (hanya diaudit) |
| B-6 | Elementor Pro | keputusan tertunda |
| B-7 | WXR Kebumen di repo publik | **MASIH PUBLIK** ✗ — lihat §5 |
| **B-8** | **Browser automation tidak stabil** (3 dari 4 situs timeout) | **BLOCKER untuk QA visual menyeluruh** |

## 5. Repo GitHub
- Status: **tetap PUBLIC** ✗ — `gh` CLI tidak terpasang; sesi GitHub terautentikasi tidak tersedia; **tidak** mencoba bypass autentikasi ✓
- Manual: `gh auth login` → `gh repo edit vael-skaya/wp --visibility private --accept-visibility-change-consequences`
- Reminder: privatisasi **tidak** menarik salinan/fork yang sudah diunduh ✓
- **Tidak** ada commit/push/perubahan remote/riwayat ✓

## 6. Precheck produksi (read-only, tanpa tindakan)
**Sudah terbukti di staging ✓:** PHP 8.2.12 · MariaDB 10.4.32 · WP 7.0.7 (Kebumen 7.1.3) · Astra 4.14.0/4.14.1 · locale `id_ID` di **keempat** situs ✓ · satu akun admin per situs · theme+plugin dari repo resmi · semua halaman publish 200 · aset lokal resolvable · checkpoint tersedia.

**Harus diverifikasi di hosting ⚠️:** versi PHP & `memory_limit` (staging 40M → target ≥256M) · HTTPS + redirect kanonik · DNS 4 domain · izin file · cron · email pengiriman.

**Perlu konfirmasi sekolah/Pak Eko ❓:** alamat email resmi per situs (`info@…` pada Kontak) · nama badan penyelenggara untuk kebijakan privasi · persetujuan teks kebijakan · siapa pemegang akun admin.

**Syarat minimum sebelum migrasi:** backup penuh + **restore drill** yang diuji · rollback plan tertulis · migrasi **satu situs dahulu** → verifikasi → baru situs lain · kriteria stop: halaman 500, aset 404 massal, atau locale berubah.

## 7. Gerbang persetujuan produksi
**BELUM memenuhi syarat.** Tes kritis yang **belum dijalankan**: QA visual 3 situs lain, navigasi klik/keyboard, formulir, dan verifikasi gambar (lazy-load). Fase ini **tidak** menyatakan siap produksi.

## 8. Artefak
- Laporan: `FASE-8E-QA-VISUAL-INTERAKTIF-HASIL.md`
- Checkpoint: `staging/checkpoints/checkpoint-8e-pre-locale-id-id-20261022-002520.sql`
- Data: `sragen/export/_8e.json`, `_8e_qa.json`
- **Screenshot: tidak ada** (browser tidak stabil) ✗

## 9. Langkah berikutnya (singkat)
1. Jalankan QA visual ulang saat Chrome stabil (tutup semua Chrome pribadi agar profil headless tidak terkunci).
2. `gh auth login` → privatkan repo.
3. Konfirmasi email resmi + teks kebijakan privasi.
4. Set `WP_MEMORY_LIMIT` ≥256M di produksi saat migrasi (bukan sekarang).

## 10. Konfirmasi batas
`goly1264_ams` & `Downloads\admin.php` **tidak disentuh** ✓ · dump/backup/pohon bukti/inventaris/checksum/manifest **tidak diubah** ✓ · staging lain **tidak diubah** ✓ · checkpoint lama **tidak ditimpa** ✓ · tanpa impor ke produksi ✓ · payload tidak dijalankan, `cgi-bin.zip` tidak diekstrak ✓ · tanpa akun/kredensial lama ✓ · tanpa push/commit ✓ · tanpa layanan berbayar ✓ · tanpa secrets di laporan ✓
