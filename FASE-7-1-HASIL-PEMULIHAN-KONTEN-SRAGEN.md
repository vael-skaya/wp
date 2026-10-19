# FASE 7.1 — HASIL PEMULIHAN KONTEN, LAMPIRAN, UI & QA SRAGEN

**Tanggal:** 19 Oktober 2026 · Staging tidak dibuat ulang ✓ · DB tidak dikosongkan ✓ · Tanpa push/sinkronisasi ✓

## A. Checkpoint (baru, tidak menimpa)
`recovery-work\staging\checkpoints\checkpoint-7-1-pre-perbaikan-sragen.sql` — 677.987 B, SHA-256 `b94bb24788eb55572468d829…` ✓ (checkpoint-7-01 sebelumnya tetap utuh ✓)

## B. Triage kredensial (isi TIDAK dibaca/ditampilkan)
| Berkas | Ada | Ukuran | Mode | Status Git |
|---|---|---:|---|---|
| `.dbpass` | ya | 32 B | 644 | **tidak ada repo Git lokal** ⇒ tidak terlacak |
| `.adminpass` | ya | 28 B | 644 | idem |
| `.cibitungadminpass` | ya | 32 B | 644 | idem |
| `.tambunadminpass` | ya | 32 B | 644 | idem |
| `.my_staging.cnf` | ya | 89 B | 644 | idem |

- Repo **publik** (`vael-skaya/wp`, 46 berkas per audit 6K-D) **tidak** memuat berkas-berkas tersebut ✓ (dotfile tidak dipublikasikan).
- **Riwayat Git tidak dapat diperiksa** (tidak ada clone lokal) ✗ ⇒ `needs_manual_review` bila Anda pernah meng-commit dari mesin lain.
- **Tidak** dilakukan: push, perubahan remote, penghapusan berkas, atau perubahan riwayat ✓.
- Rekomendasi: pindahkan kelima berkas ke lokasi luar repository (mis. `C:\Users\sakha\.smk10\`) **sebelum** sinkronisasi berikutnya; rotasi kredensial hanya jika terbukti pernah ter-commit.

## C. Artikel — **belum berhasil** ✗ (blocker)
- Terimpor: **1 `post`** (sumber ~96 publish) ✗ · halaman 13 ✓
- Konverter: **149 dipilih / 135 dikeluarkan**, **147 postmeta serialized ditunda** ✗
- **Penyebab belum dipastikan**: kriteria seleksi konverter belum diaudit per-item ⇒ `PERLU AUDIT` (lihat `FASE-7-1-REKONSILIASI-ARTIKEL-SRAGEN.csv`)
- Tidak menggunakan `unserialize()` PHP, tidak menjalankan SQL sumber ✓

## D. Lampiran — **BERHASIL** ✓
| Tahap | Hasil |
|---|---|
| Kandidat lampiran asli (dump × manifest `original`) | **107** |
| Dibuat via `wp_insert_attachment` + `_wp_attached_file` + metadata (lokal) | **106** dibuat, 1 sudah ada, 0 gagal, 0 berkas hilang |
| Lampiran lama yang salah (`_wp_attached_file` menunjuk **berkas turunan**, 60 record) | **dihapus** (setelah verifikasi bukan bagian 107 asli) |
| **Lampiran akhir** | **107** ✓ (satu per lampiran sumber, tanpa duplikat) |
Jumlah `attachment` di staging kini **107** ✓ — sesuai sumber.

## E. Elementor JSON — **BERHASIL (koreksi laporan)**
**18 dokumen `_elementor_data` → 18 valid, 0 invalid** ✓. Angka "1 tidak valid" pada Fase 7 berasal dari metrik internal `step_verify`, **bukan** dari validasi JSON dokumen — dikoreksi di sini.

## F. UI & fungsi
| Item | Status |
|---|---|
| Label `(STAGING pemulihan)` pada judul situs | **DIPERBAIKI** → `blogname = "SMK 10 November Sragen"`; HTML: `STAGING`=0, `pemulihan`=0 ✓ |
| Header/logo/banner/menu/footer | **BELUM** ✗ |
| Menu mobile | **BELUM diuji** ✗ |
| Container/banner Elementor, tipografi, lebar | **BELUM** ✗ |

## G. QA (yang benar-benar dijalankan)
| Uji | Hasil |
|---|---|
| 7 halaman (`/`, sejarah, kontak, struktur-organisasi, visi-misi, fasilitas, prestasi) | **200 semua** ✓ |
| Judul situs | `SMK 10 November Sragen` ✓ |
| URL domain produksi di beranda | **0** ✓ |
| Gambar lokal di beranda | 14 tautan `localhost/staging/sragen/...` ✓ |
| Akun | hanya `sragen_admin` ✓ |
| Lampiran | 107 ✓ |
| Elementor JSON | 18/18 valid ✓ |
| **Responsif 360/390/412/768/1366** | **BELUM DIUJI** ✗ |
| **Konsol JavaScript / warning PHP** | **BELUM** ✗ |

## H. Blocker & keputusan
1. **Artikel**: butuh audit kriteria seleksi konverter (`convert_dump_to_wxr.py`) + daftar 135 item dikeluarkan → keputusan Anda: pulihkan semua artikel publish, atau subset.
2. **UI**: pekerjaan terbesar yang tersisa (header/logo/banner/menu/responsif).
3. **Kredensial**: izin memindahkan 5 berkas sandi ke luar repository.
4. **Publikasi**: tetap **lokal** (repo publik + laporan menyebut path), tanpa push ✓.

## I. Konfirmasi
Produksi tidak disentuh ✓ · dump/tree/inventaris/checksum/manifest tidak diubah ✓ · payload tidak dijalankan, `cgi-bin.zip` tidak diekstrak ✓ · `goly1264_ams`, proyek AMS, `C:\Users\sakha\Downloads\admin.php` tidak disentuh ✓ · tanpa push/perubahan remote ✓.
