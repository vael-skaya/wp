# FASE 6K-D — AUDIT PAPARAN REPOSITORY DOKUMENTASI

**Tanggal:** 19 Oktober 2026 · **Metode:** pembacaan **read-only** repo publik `vael-skaya/wp` via GitHub API + raw (tanpa clone, tanpa unduh berkas untuk dijalankan, **tanpa** perubahan apa pun pada repo)
**Ringkas:** repo **dapat diakses publik** dan memuat **46 berkas** — termasuk **ekspor WXR staging Kebumen (1,6 MB)**, laporan forensik rinci, tiga CSV manifest, dan tree `wp-content` ✗ ⇒ **paparan informasi nyata**, perlu tindakan

---

## 1. Artefak publik yang diperiksa (46 berkas, semua di root, tanpa subdirektori)
| Kategori | Berkas | Catatan |
|---|---|---|
| **Ekspor data staging** | `kebumen-staging-pilot.wxr` (**1.599.301 B**) · `kebumen-staging-pilot.wxr.dryrun.txt` (21.033 B) · `kebumen-media-manifest.tsv` (7.385 B) | **WXR = isi situs + blok `<wp:author>`** ✗ |
| **Manifest media (fase 6K-C)** | `FASE-6K-C-MANIFEST-MEDIA-SRAGEN.csv` (203.232 B) · `-v2.csv` (222.327 B) · `-v3.csv` (178.797 B) | path media + SHA-256 ✗ |
| **Laporan forensik rinci** | `LAPORAN-FASE-2.md` · `LAPORAN-FASE-3A.md` · `LAPORAN-FASE-3A-ADDENDUM.md` · `FASE-4A-INVENTARIS.md` · `FASE-4C-1/2/3` · `FASE-4D` | hash malware, path artefak, **audit akun** ✗ |
| **Laporan pemulihan** | `FASE-5A…6K-C` (30 berkas) | sebagian memuat nama database, path, dan hasil uji |
| **Dump struktur berkas** | `wp-content.txt` (200.789 B) | daftar berkas lengkap (termasuk nama plugin/tema) ✗ |
| **Berkas lokal-sensitif** | `recovery-work\sensitive\6j-akun\*` (akun-rinci.tsv, akun-6j.json, laporan rinci) | **TIDAK dipublikasikan** ✓ |

## 2. Kategori risiko — bukti terukur (pemindaian pola, **isi tidak dicetak**)
| Kategori | Jumlah kemunculan | Berkas terbanyak | Penilaian |
|---|---:|---|---|
| Rujukan domain C2/IoC (`wp2shell`, `expansor`, dll.) | 205 | `FASE-4C-2` (130) | **Sedang** — mengungkap indikator & taktik; berguna bagi penyerang untuk menguji apakah mereka masih terdeteksi |
| **Lokasi berkas berbahaya** (`admin.php`, `cgi-bin.zip`) | 76 | `FASE-4A` (17), `3A-ADDENDUM` (10) | **Sedang** — mengungkap keberadaan & nama berkas webshell di mesin analis |
| Nama kredensial/DB/cPanel (`DB_NAME`, `smkq8953`, `cpanel`) | 64 | `FASE-4A`/`4C-1` (18 masing-masing) | **Rendah–sedang** — umumnya **nama** database/user, bukan sandi |
| **Alamat email** | 43 | **`kebumen-staging-pilot.wxr` (18)** · `wp-content.txt` (10) | **TINGGI** ✗ — data pribadi pada berkas yang dapat diunduh siapa pun |
| Token/API key (pola kata) | 6 | `4A`, `4C-1`, `6D` | **`needs_manual_review`** — belum dipastikan kunci nyata vs istilah teknis |
| Pola `password:` | 1 | (satu laporan) | **`needs_manual_review`** — kemungkinan besar prosa/label, tidak dapat dipastikan tanpa menampilkan isi |
| Blok `<wp:author>` | 1 | `kebumen-staging-pilot.wxr` | **TINGGI** ✗ — WXR biasanya memuat login + email penulis |

## 3. Temuan (dipisah: terbukti / dugaan / perlu tinjauan)
### 3.1 Terbukti
1. **WXR staging Kebumen dipublikasikan** (1,6 MB, ada blok `<wp:author>`, 18 kemunculan pola email) ⇒ **isi situs + identitas penulis/komentator dapat diunduh publik** ✗ **PRIORITAS 1**.
2. **Audit akun `FASE-4C-2-AUDIT-AKUN-KONTEN.md` (48 KB) dipublikasikan** ⇒ mengungkap pola akun admin palsu, username berpola (`wp2shell_*`, `w2s_*`), metrik 163 akun ✗ **PRIORITAS 2** (berguna bagi penyerang untuk mengetahui apa yang sudah diketahui).
3. **Tiga CSV manifest media + tree `wp-content`** dipublikasikan ⇒ struktur berkas & checksum media sekolah ✗ **PRIORITAS 3**.
4. **Laporan memuat lokasi webshell** (`Downloads\admin.php`, `cgi-bin.zip`) dan nama database ✗ **PRIORITAS 2** — mengarahkan perhatian ke artefak yang masih ada di mesin analis.
5. **Klarifikasi penting:** `FASE-6J-AKUN-WORDPRESS.md` (versi non-sensitif) memang dipublikasikan ✓ dan **berkas rinci akun 6J tidak** ✓ — pernyataan "rincian hanya lokal" **benar untuk artefak 6J** ✓, tetapi **tidak berlaku untuk keseluruhan proyek** karena `FASE-4C-2` (audit akun fase sebelumnya) sudah publik ✗ ⇒ **ketidaksesuaian yang wajib dicatat**.

### 3.2 Dugaan (belum terbukti)
- Email pada WXR berasal dari **komentar** (`wp:comment_author_email`) dan/atau **penulis** — tidak dapat dipastikan tanpa menampilkan isi.
- 6 kemunculan pola token kemungkinan besar istilah teknis, bukan kunci aktif.
- `wp-content.txt` dapat mengungkap plugin rentan yang pernah dipakai (berguna bagi penyerang).

### 3.3 `needs_manual_review`
- Berkas dengan pola `token`/`password` (5 berkas) — perlu pemeriksaan manual **oleh Anda** (bukan dicetak di sini).
- Isi lengkap `kebumen-staging-pilot.wxr` — perlu dipastikan apakah memuat **email komentator** (PII pihak ketiga).

## 4. Rekomendasi mitigasi (tanpa melakukan perubahan remote)
| Prioritas | Tindakan | Alasan |
|---|---|---|
| **P1** | **Hapus/ privatkan `kebumen-staging-pilot.wxr` + `.dryrun.txt`** dari repo publik | PII + isi situs |
| **P1** | **Rotasi** kredensial apa pun yang pernah muncul di laporan (nama DB/user boleh tetap, sandi wajib dianggap terpapar bila pernah tertulis) | Prinsip least exposure |
| **P2** | Hapus/ privatkan `FASE-4C-2-AUDIT-AKUN-KONTEN.md`, `FASE-4A-INVENTARIS.md`, `LAPORAN-FASE-3A*.md`, `FASE-4C-1/4C-3` | Menyingkap taktik & artefak |
| **P2** | Ganti rujukan path nyata (`Downloads\admin.php`, `cgi-bin.zip`) dengan label netral (`<artefak-kuarantina>`) di laporan publik | Mengurangi penargetan |
| **P3** | Pindahkan manifest CSV & `wp-content.txt` ke repo privat atau lampiran lokal | Struktur internal |
| **P3** | Pertahankan laporan versi **non-sensitif** (seperti `FASE-6J-AKUN-WORDPRESS.md`) sebagai publik; versi rinci tetap lokal | Model yang sudah benar |
| **P4** | Tambahkan `SECURITY.md`/kebijakan: laporan forensik tidak dipublikasikan tanpa redaksi | Pencegahan |
| **P4** | Audit git history (file yang dihapus tetap ada di riwayat) | Penghapusan file **tidak** menghapus riwayat |

**Catatan:** semua tindakan di atas **tidak** saya lakukan — perubahan repo (hapus/commit/push/privatkan) di luar izin fase ini.

## 5. Ketidaksesuaian versi lokal vs repo (tidak diunduh/diganti)
Beberapa berkas publik **berukuran berbeda** dari salinan lokal, mis. `FASE-4D-RENCANA-MIGRASI.md` (repo 24.240 B vs lokal 24.031 B), `FASE-5A-HASIL-EKSEKUSI.md` (22.389 vs 22.067), `LAPORAN-FASE-2.md` (19.690 vs 19.526).

**⚠️ Implikasi penting:** file `FASE-6K-C-MANIFEST-MEDIA-SRAGEN-v3.csv` (178.797 B) dan laporan 6K-B/6K-C **sudah ada di repo** — jadi repo kemungkinan disinkronkan otomatis/manual dari `recovery-work\`. Karena itu, **artefak 6K-D ini pun berpotensi ikut terpublikasi** bila sinkronisasi berjalan ⇒ rekomendasi P1/P2 harus diputuskan **sebelum** sinkronisasi berikutnya.

## 6. Batasan audit
- Hanya membaca **HEAD** repo publik (46 berkas); riwayat commit **tidak** diperiksa.
- Pemindaian berbasis pola — bukan verifikasi kriptografis kunci/rahasia; nilai rahasia **tidak** ditampilkan/diuji.
- Tidak dapat memastikan isi PII tanpa menampilkan data (dihindari secara sengaja).
- Tidak memeriksa fork/mirror/cache pihak ketiga.

## 7. Konfirmasi
Tidak ada perubahan pada repo (tanpa commit/push/privatkan/hapus) ✓ · tidak mencetak rahasia/kredensial/hash kata sandi/data pribadi ✓ · tidak mengunduh berkas untuk dijalankan ✓ · tidak menyentuh `goly1264_ams`, proyek AMS, `C:\Users\sakha\Downloads\admin.php`, produksi, atau tree bukti ✓.
