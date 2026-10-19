# FASE 6K-C-1 — REKONSILIASI MANIFEST MEDIA SRAGEN

**Tanggal:** 19 Oktober 2026 · **Mode:** audit ulang + koreksi output manifest (offline, read-only terhadap sumber)
**Hasil:** **dua kesalahan logika path ditemukan dan diperbaiki**; jumlah kandidat **524 → 418**; status **`missing` 125 → 0**; **418/418 kandidat `verified`**; **107/107 berkas asli terverifikasi**

---

## 1. Status sumber (diverifikasi ulang, tidak berubah)
| Sumber | Identitas | Status |
|---|---|---|
| Dump `smkq8953_sragen10.sql` | 80.897.366 B · SHA-256 `0448259bb0a117636e2e40bcdd11b15ea2f66d1f7615a84769dac0680b46f3af` | **identik dengan checkpoint 6K-C** ✓ (dihitung ulang) |
| Inventaris | 1.082.251 B · 7.445 entri | mtime tidak berubah (13-10-2026 13:14:43) ✓ |
| Checksum | 1.110.274 B · 7.445 entri | mtime tidak berubah (13-10-2026 13:15:25) ✓ |
| Manifest v1 | `c6e9fcf3…` | **tidak dipakai**, tidak diubah ✓ |
| Manifest v2 | `0615976c…` | **tidak diubah** (tidak ditimpa) ✓ |
| Tree bukti | `D:\smk10november` | read-only, dapat dibaca ✓ |

## 2. Kesalahan logika yang ditemukan (dan yang tidak terbukti)
### ✅ Terbukti — Bug A: `metadata['file']` diperlakukan sebagai turunan dan ditempel direktori
Nilai `_wp_attachment_metadata['file']` **sudah memuat `YYYY/MM`** (relatif terhadap `wp-content/uploads/`), tetapi v2 menempelkannya ke direktori lampiran ⇒ path ganda.
- **Sebelum (salah):** `…/uploads/2025/12/2025/12/header_001.webp`
- **Sesudah (benar):** nilai itu **identik** dengan `_wp_attached_file` di **107/107 lampiran** ⇒ **tidak** menghasilkan kandidat tambahan (bukan turunan).
- Dampak: **106 baris** `derivative` palsu, semuanya berstatus `missing` ✗.

### ✅ Terbukti — Bug B: `sizes[].file` yang sudah berupa **path** ikut ditempel direktori
Sebagian `sizes` (mis. thumbnail Elementor) menyimpan `file` sebagai path relatif ke uploads (`elementor/thumbs/<nama>-<hash>.webp`), bukan basename.
- **Sebelum (salah):** `…/uploads/2025/12/elementor/thumbs/img_002-…webp`
- **Sesudah (benar):** `…/uploads/elementor/thumbs/img_002-…webp` ✓ (18 baris; 19 baris `missing` v2 → **0**)
- Dampak: **19 baris** `missing` palsu ✗.

### ❌ Tidak terbukti — semantik `original_image`
`original_image` **bukan** varian biasa: ia berkas **resolusi penuh** ketika berkas utama adalah versi `-scaled`. Data mengonfirmasi: lampiran **1061** → `_wp_attached_file = SPMB-GELOMBANG-II-scaled.webp` (berkas utama) + `original_image = SPMB-GELOMBANG-II.webp` (2 gambar berbeda) ⇒ keduanya dicatat terpisah dengan tipe `original_image`, bukan dicampur sebagai `derivative`.

### ❌ Tidak terbukti — "semua situs memakai struktur identik"
Diperiksa per lampiran (bukan diasumsikan): 107/107 konsisten (`file` = `_wp_attached_file`), **tetapi** 18 lampiran memakai path di dalam `file` `sizes` (Elementor) ⇒ **struktur tidak seragam**, karena itu resolusi path kini **berbasis isi nilai** (mengandung `/` ⇒ relatif ke uploads; tanpa `/` ⇒ basename di direktori lampiran).

## 3. Metode koreksi (deterministik)
```
resolve(frag):
  frag kosong / diawali "/" / mengandung ".."        → tidak valid (kandidat dibuang & dilaporkan)
  frag mengandung "/"  → SITE/wp-content/uploads/ + frag        (path relatif ke uploads)
  selain itu            → dirname(_wp_attached_file) + "/" + frag (basename varian)
```
Kandidat dibangun dari: `_wp_attached_file` (tipe `original`), `sizes[*].file` (tipe `derivative`), `original_image` (tipe `original_image`). `metadata['file']` hanya **dibandingkan**, tidak dijadikan kandidat bila identik. Deduplikasi per lampiran berdasarkan path persis.

## 4. Perbandingan angka
| Metrik | v2 (sebelum koreksi) | v3 (setelah koreksi) |
|---|---:|---:|
| Baris kandidat | 524 | **418** |
| — original | 107 | 107 |
| — derivative | 417 | **310** |
| — original_image | 0 | **1** |
| — path ganda tahun/bulan | **106** ✗ | **0** ✓ |
| `verified` | 399 | **418** ✓ |
| `missing` | **125** ✗ | **0** ✓ |
| `mismatch` / `ambiguous` / `inventory_only` / `excluded` / `needs_manual_review` | 0 / 0 / 0 / 0 / 0 | 0 / 0 / 0 / 0 / 0 |
| Berkas asli terverifikasi | 107/107 | **107/107** ✓ |
| Berkas unik | 399 | **418** (total **21,1 MB**) |

**Kesimpulan penting:** setelah kandidat yang dibentuk keliru dikeluarkan, **tidak ada satu pun media yang benar-benar hilang** — seluruh kumpulan media yang dirujuk 107 lampiran Sragen **lengkap dan cocok checksum** ✓. Angka "125 hilang" pada v2 **tidak valid** dan digantikan **0**.

## 5. Cara hash diverifikasi
Untuk setiap kandidat yang path-nya ada di inventaris: ukuran dibandingkan dengan inventaris, **SHA-256 dihitung langsung dari byte berkas di tree bukti** (`D:\smk10november\…`, read-only) lalu dibandingkan dengan nilai di `sha256-…sragen….txt`. **418/418 cocok** ⇒ status `verified` (bukan `inventory_only`). **Hash hanya membuktikan identitas byte**, bukan bebas malware.

## 6. Distribusi
- Kandidat per lampiran: 1×1, 2×1, 3×48, 4×24, 5×27, 6×4, 8×2 (total 107 lampiran)
- Ekstensi: `.webp` 258 · `.jpg` 116 · `.png` 33 · `.jpeg` 10 · **`.pdf` 1** — **tidak ada** `.php`/`.js`/`.html`/arsip/executable ✓
- **PDF 1127 `KALDIK_2025_2026.pdf`** (2.855.173 B) terverifikasi sebagai **berkas asli**; 4 berkas `KALDIK_2025_2026-pdf*.jpg` = **pratinjau JPG** (dicatat `derivative`, catatan eksplisit "pratinjau JPG dari PDF — BUKAN berkas PDF asli")

## 7. Pengecualian keamanan (tidak berubah)
Tetap dikecualikan: `wc-logs/`, `cartflows-logs/`, log impor `astra-sites/`, `wpforms/cache/`, `wp-file-manager-pro/fm_backup/`, sisa sesi `ai-builder/`/`template-kits/`/`forminator/`, serta apa pun yang terkait IoC terverifikasi. **0 kandidat lampiran** jatuh di path tersebut ✓. **2.022 berkas uploads tidak dirujuk** tetap `unreferenced_quarantine` (tidak masuk kandidat hanya karena hash cocok) ✓. `cgi-bin.zip` **tidak** diekstrak; `goodsdetails.php` **tidak** dijalankan ✓.

## 8. Batasan
- Kesesuaian bergantung pada tree bukti yang tidak berubah sejak 13-10-2026 (saat ini cocok ✓).
- Tidak ada pemindaian malware terhadap isi media.
- `cgi-bin.zip` dan payload biner tetap di luar cakupan.
- `metadata['file']` diverifikasi **identik** di 107 lampiran; bila kelak ada lampiran dengan `file` berbeda, aturan yang sama akan menandainya sebagai `metadata_file` + `needs_manual_review` (tidak digabung diam-diam).

## 9. Output
| Berkas | Ukuran | SHA-256 |
|---|---:|---|
| `recovery-work\FASE-6K-C-MANIFEST-MEDIA-SRAGEN-v3.csv` **(resmi)** | 178.797 B | **`bf9ec91253007cca645d7f982430287f03b305ace6bfe8cfee5c31896534a808`** |
| `recovery-work\FASE-6K-C-REKONSILIASI-MANIFEST-SRAGEN.md` (laporan ini) | 7799 B | dicatat di respons akhir 6K-C-1 (hash berubah bila laporan disunting) |
| v1 `c6e9fcf3…` / v2 `0615976c…` | — | dibiarkan utuh, **tidak dipakai** |

**Validasi v3:** dibaca ulang ✓ · **418 baris** = 418 hubungan lampiran–kandidat ✓ · 14 kolom ✓ · 107 lampiran unik ✓ · **0 path dengan `YYYY/MM` ganda** ✓ · **0 baris tanpa ukuran** ✓ · **0 ukuran/hash tidak cocok** ✓ · **0 duplikat** ✓ · **0 email, 0 hash akun, 0 kata password/token** ✓

## 10. Konfirmasi
- **Tidak ada** staging/database Sragen dibuat; **tidak ada** impor SQL/WXR; **tidak ada** arsip diekstrak; **tidak ada** berkas dijalankan ✓
- Dump, inventaris, checksum, tree bukti, dan output v1/v2 **tidak diubah** ✓
- **Produksi tidak disentuh**; tanpa deployment/DNS/hosting/DB produksi ✓
- **`goly1264_ams` tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah** ✓ · **proyek AMS tidak disentuh** ✓
- **`C:\Users\sakha\Downloads\admin.php` tidak disentuh** (tidak dibaca/dijalankan/dipindai/diedit/dipindah/diganti nama/dihapus) ✓
- Tanpa `git push`/commit/deployment; output tidak diunggah ke GitHub ✓
