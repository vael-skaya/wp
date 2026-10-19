# FASE 6K-C — HASIL VALIDASI & MANIFEST MEDIA SRAGEN

**Tanggal:** 19 Oktober 2026 · **Mode:** offline read-only · **Tidak ada** staging/database dibuat, **tidak ada** impor, **tidak ada** arsip diekstrak, **tidak ada** perubahan pada sumber/produksi
**Kesimpulan utama:** **107/107 lampiran memiliki berkas asli yang terverifikasi** (path + ukuran + SHA-256 dihitung dari byte aktual) ✓ · **125 varian turunan hilang** (bukan berkas asli) ✗ · **2.022 berkas uploads tidak dirujuk** → karantina ✓ · 0 mismatch · 0 ambiguous

---

## 1. Sumber yang benar-benar diperiksa (Tahap A)
| Sumber | Lokasi terverifikasi | Ukuran | mtime | Hash / validasi |
|---|---|---:|---|---|
| Dump SQL Sragen | `D:\backup-database-smk10november\smkq8953_sragen10.sql` | **80.897.366 B** | 2026-10-13 23:27:26 | **SHA-256 dihitung: `0448259bb0a117636e2e40bcdd11b15ea2f66d1f7615a84769dac0680b46f3af`** — **identik** dengan hash penuh yang tercatat di `audit-backup\FASE-4C-1-INVENTARIS-DATABASE.md`, `audit-backup\_dbaudit-raw.txt`, dan `INDEX.md` ✓ |
| Inventaris berkas | `…\manifest\inventory-smk10novembersragen.sch.id_.txt` | 1.082.251 B | 2026-10-13 13:14:43 | 7.445 baris, format `mtime\|waktu\|ukuran\|mode\|path` ✓ |
| Checksum berkas | `…\manifest\sha256-smk10novembersragen.sch.id_.txt` | 1.110.274 B | 2026-10-13 13:15:25 | 7.445 baris `sha256 *path` ✓ |
| Laporan 6K-B | `…\recovery-work\FASE-6K-B-AUDIT-OFFLINE-SRAGEN.md` | 12.731 B | 2026-10-19 11:31 | acuan |
| Tree bukti | `D:\smk10november\smk10novembersragen.sch.id` | — | — | **dapat diakses read-only** ⇒ hash dihitung dari byte aktual |

### Hasil validasi input
| Pemeriksaan | Hasil |
|---|---|
| Baris inventaris | 7.445 · **entri unik 7.445** · baris rusak **0** · path duplikat **0** · path traversal (`..`) **0** · ukuran negatif/tidak valid **0** |
| Hash format | 7.445 entri valid (`^[0-9a-f]{64} \*path$`) · format salah **0** |
| **Kecocokan path (bukan jumlah)** | Himpunan path inventaris ⇄ checksum: hanya-di-inventaris **0**, hanya-di-checksum **0**, irisan **7.445** ✓ |
| Hash dump | **penuh, cocok** (bukan hanya potongan) ✓ |

## 2. Ekstraksi metadata lampiran (Tahap B)
Dump dibaca **sebagai teks** dengan parser SQL tertutup (tanpa eksekusi pernyataan SQL). `_wp_attachment_metadata` (PHP-serialized) dibaca dengan **parser struktural sendiri yang sadar panjang byte** — **tidak** memakai `unserialize()` dan **tidak** menjalankan kode dari data.

| Item | Hasil |
|---|---|
| Lampiran (`srg_posts.post_type='attachment'`) | **107** (semuanya `post_status=inherit`) |
| Tipe MIME | **106 `image/*`**, **1 `application/*`** (`KALDIK_2025_2026.pdf`, 2.855.173 B) |
| `_wp_attached_file` terparse | 107/107 ✓ (0 kosong/tidak valid) |
| `_wp_attachment_metadata` terparse | **524/524 kandidat `metadata_status=ok`** (0 `unparsed`) ✓ |
| Kandidat dibangkitkan | 107 original + **417 turunan** (dari `sizes`, `original_image`, `file`) = **524** |

## 3. Pemetaan & validasi checksum (Tahap C–D)
Pencocokan **path relatif persis** (bukan nama berkas), dengan prefiks direktori situs `smk10novembersragen.sch.id/`.

| Status | Jumlah | Arti |
|---|---:|---|
| **`verified`** | **399** | path ada di inventaris **dan** ukuran cocok **dan** SHA-256 cocok — **hash dihitung langsung dari byte berkas aktual**, bukan hanya dari inventaris |
| **`missing`** | **125** | kandidat tidak ada di inventaris |
| `mismatch` | **0** | — |
| `inventory_only` | **0** | seluruh kandidat yang ditemukan dapat di-hash langsung |
| `ambiguous` | **0** | tidak ada lampiran dengan >1 kandidat untuk path yang sama |
| `excluded` | **0** | **tidak ada** kandidat lampiran yang jatuh di path yang dikecualikan |
| `needs_manual_review` | **0** | — |

### Per lampiran (penting)
| | Jumlah |
|---|---:|
| Lampiran diproses | **107** |
| **Berkas ASLI terverifikasi** | **107 (100 %)** ✓ |
| Berkas asli hilang | **0** ✓ |
| Lampiran tanpa kandidat asli | **0** ✓ |
| **Varian turunan hilang** | **125** (dari 417 turunan) — tidak menghalangi migrasi: WordPress membuat ulang thumbnail saat regenerasi media |

**Tidak ada satu pun berkas `verified` ber-ekstensi berisiko** (`.php`, `.phtml`, `.phar`, `.js`, `.html`, `.htaccess`, `.exe`, `.sh`) → **0** ✓.

## 4. Pengecualian keamanan (Tahap E)
- **Kandidat lampiran:** 0 berkas jatuh di path yang dikecualikan (`wc-logs/`, `cartflows-logs/`, `astra-sites/`, `wpforms/cache/`, `wp-file-manager-pro/`, `ai-builder/`, `template-kits/`, `forminator/`, `elementor/forms/`) ✓
- **Berkas uploads yang tidak dirujuk lampiran: 2.022** → status **`unreferenced_quarantine`** (tidak ikut migrasi tanpa evaluasi):

| Kategori | Jumlah | Alasan |
|---|---:|---|
| Media lain tidak dirujuk (jpg/webp/png/jpeg) | 1.701 | tak ada relasi lampiran; mungkin sisa unggahan lama |
| Aset Elementor (`uploads/elementor/`) | 166 | CSS/ikon hasil generator, bukan konten |
| Log impor kit (`astra-sites/`) | 72 | log impor |
| Sesi `template-kits/` | 34 | sisa kit |
| Log WooCommerce/Midtrans (`wc-logs/`) | 23 | **log transaksi** (data pelanggan — tidak boleh masuk manifest) |
| `wpforms/cache/` | 12 | cache |
| Sesi `forminator/` | 5 | sisa form |
| `ai-builder/` | 4 | sisa wizard |
| Log CartFlows | 3 | log |
| `wp-file-manager-pro/fm_backup/` | 2 | **cadangan oleh plugin file-manager** |

- 8 berkas `.php` di uploads: **semuanya stub plugin sah** (0–99 B) — dicatat sebagai false-positive, **tidak** dikecualikan berdasarkan nama semata ✓
- **Tidak** membuka/mengekstrak `cgi-bin.zip`; **tidak** menjalankan payload apa pun ✓

## 5. Ketidakcocokan & kesenjangan bukti
| Item | Status |
|---|---|
| Ukuran tidak cocok | **0** |
| Hash tidak cocok | **0** |
| Metadata tidak dapat diparse | **0** |
| Berkas asli hilang | **0** |
| Turunan hilang | 125 (dinilai tidak kritis) |
| Cakupan `cgi-bin.zip` (1,33 GB) | **tidak tercakup** — belum diinventaris, butuh persetujuan ekstraksi |
| Payload termediah `goodsdetails.php` (>14 lapis) | **tidak** dianalisis di sini (bukan media; tidak dieksekusi) |
| Pemeriksaan malware isi media | **tidak dilakukan** — hash hanya membuktikan identitas byte terhadap pembanding |

## 6. Output (lokal, tidak diunggah)
| Berkas | Ukuran | SHA-256 |
|---|---|---|
| `recovery-work\FASE-6K-C-MANIFEST-MEDIA-SRAGEN-v2.csv` | **222.327 B** | **`0615976c28f36fd990d3f0ca1ced49ec74dbc874bf3cfd7fa8be737dbc4b6e00`** |
| `recovery-work\FASE-6K-C-MANIFEST-MEDIA-SRAGEN.csv` (v1, **cacat**, jangan dipakai) | 203.232 B | `c6e9fcf337ff505d6e9ad396349bb25733608b082dd20e85713ff61ebb250051` |
| `…\recovery-work\FASE-6K-C-HASIL-VALIDASI-SRAGEN.md` (laporan ini) | — | lihat §9 |

**⚠️ Pengungkapan cacat v1:** pada CSV v1, kolom `notes` untuk baris **verified** keliru berisi "tidak ada di inventaris" (warisan logika dari baris `missing`). Data status/hash/size **tidak** terpengaruh. Sesuai aturan "jangan menimpa output", v1 **tidak** dihapus/ditimpa; **v2** diterbitkan dengan `notes` yang benar dan menjadi rujukan resmi.

**Verifikasi output:** CSV v2 dibaca ulang ✓ · **524 baris** (sama dengan jumlah baris manifest) ✓ · 14 kolom sesuai skema ✓ · `hash_validation_status` hanya berisi `verified`/`missing` ✓ · `migration_status` hanya `candidate`/`missing` ✓ · tanpa email/password/token/isi log ✓.

## 7. Risiko yang tersisa & rekomendasi
1. **125 turunan hilang** → regenerasi thumbnail di staging (`wp media regenerate`) setelah impor; tidak perlu menyalin berkas.
2. **2.022 berkas tak dirujuk** → jangan dimigrasikan; simpan sebagai bukti. Bila sekolah mengenali media penting (mis. dokumentasi foto), perlu kurasi manual terpisah.
3. **PDF `KALDIK_2025_2026.pdf`** (2,86 MB) terverifikasi ✓ — kandidat migrasi sah; sebaiknya diperiksa kelayakan konten oleh sekolah.
4. **`fm_backup/`** (2 berkas) dan **log transaksi** dikecualikan; jangan dimigrasikan meski berukuran kecil.
5. Hash **bukan** bukti bebas malware ⇒ sebelum migrasi tetap jalankan validasi V-01..V-12 + pemindaian pola pada berkas yang diimpor.
6. `cgi-bin.zip` dan payload termediah tetap menjadi **kesenjangan bukti** sampai ada persetujuan terpisah.

## 8. Catatan checkpoint (tanpa data rahasia)
| Input | SHA-256 / identitas |
|---|---|
| Dump `smkq8953_sragen10.sql` | `0448259bb0a117636e2e40bcdd11b15ea2f66d1f7615a84769dac0680b46f3af` (80.897.366 B) |
| Inventaris | 1.082.251 B · 7.445 entri |
| Checksum | 1.110.274 B · 7.445 entri · path-set identik dengan inventaris |
| Output v2 | `0615976c28f36fd990d3f0ca1ced49ec74dbc874bf3cfd7fa8be737dbc4b6e00` |

## 9. Keterbatasan
- Hash dihitung dari berkas di tree bukti `D:\smk10november` — bila tree berubah setelah 13-10-2026, kesesuaian dapat berubah (saat ini **cocok** dengan inventaris & checksum ✓).
- Tidak ada pemindaian malware terhadap isi media; penilaian keamanan hanya berdasarkan path, ekstensi, dan bukti forensik sebelumnya.
- `cgi-bin.zip` tidak diperiksa; payload biner tidak dianalisis.
- Tidak ada verifikasi silang terhadap situs produksi (dilarang).

## 10. Konfirmasi
- **Tidak ada** direktori staging atau database `smk10_staging_sragen` dibuat; **tidak ada** impor SQL/WXR; **tidak ada** arsip diekstrak; **tidak ada** berkas dijalankan ✓
- **Produksi tidak disentuh** (tanpa deployment/DNS/hosting/DB produksi) ✓
- Dump, inventaris, checksum, dan artefak sumber **tidak diubah** (hanya dibaca; hash dump diverifikasi ulang) ✓
- **`goly1264_ams` tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah** ✓
- **Proyek AMS tidak disentuh** ✓
- **`C:\Users\sakha\Downloads\admin.php` tidak disentuh** — tidak dibaca, dijalankan, dipindai, diedit, dipindah, diganti nama, atau dihapus ✓
- Tidak ada `git push`/commit/deployment ✓ · output tidak diunggah ke GitHub ✓
