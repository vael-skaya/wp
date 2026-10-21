# FASE 8B — QA LENGKAP EMPAT STAGING

**Tanggal:** 21 Oktober 2026 · Read-only (0 perubahan) · Hosting Rumahweb & produksi tidak disentuh

## 1. Hasil uji URL — semua halaman

| Situs | Halaman di DB | Status hasil |
|---|---:|---|
| Tambun | 12 | 10× **200**, 1× 301 (`/home`), 1× draft `/kebijakan-privasi` |
| Cibitung | 15 | 12× **200** (termasuk `elementor-1252`, 2 halaman slug kosong), 1× 301 (`/home`) |
| Kebumen | 13 | 11× **200**, 1× 404 (draft), 1× 301 (`/home`) |
| Sragen | 12 | 10× **200**, 1× 404 (draft), 1× 301 (`/home`) |

### Uji ulang URL yang sebelumnya 000
| URL | Hasil ulang | Final URL | Catatan |
|---|---|---|---|
| `tambun/kebijakan-privasi/` | **404** | `~/kebijakan-privasi/` | tetap gagal, perlu investigasi |
| `tambun/home/` | **200** | `~/` | koneksi sementara — kini normal |
| `tambun/berita-artikel/` | **200** | `~/berita-artikel/` | koneksi sementara — kini normal |
| `cibitung/home/` | **200** | `~/` | koneksi sementara — kini normal |
| `cibitung/berita-artikel/` | **200** | `~/berita-artikel/` | koneksi sementara — kini normal |
| `cibitung/kontak/` | **200** | `~/kontak/` | koneksi sementara — kini normal |
| `kebumen/laman-contoh/` | **200** | `~/laman-contoh/` | koneksi sementara — kini normal |

### Rincian Kebumen & Sragen (kata = indikasi isi nyata, bukan soft-404)

| Kebumen URL | HTTP | Judul | Kata | Aset | Aset hilang | Produksi |
|---|---|---|---:|---:|---:|---:|
| `/laman-contoh/` | **000** | - | 0 | 0 | 0 | 0 |
| `/kebijakan-privasi/` | **404** | Laman tidak ditemukan &#8211; SMK 10 Novem | 90 | 3 | 0 | 0 |
| `/home/` | **200** | SMK 10 November Kebumen &#8211; Merajut Mi | 546 | 28 | 0 | 0 |
| `/berita-artikel/` | **200** | Berita/Artikel &#8211; SMK 10 November Keb | 373 | 12 | 0 | 0 |
| `/kontak/` | **200** | Kontak &#8211; SMK 10 November Kebumen | 111 | 4 | 0 | 3 |
| `/sejarah/` | **200** | Sejarah &#8211; SMK 10 November Kebumen | 775 | 3 | 0 | 0 |
| `/visi-misi/` | **200** | Visi &#038; Misi &#8211; SMK 10 November K | 243 | 3 | 0 | 0 |
| `/struktur-organisasi/` | **200** | Struktur Organisasi &#8211; SMK 10 Novembe | 75 | 4 | 0 | 0 |
| `/jurusan-konsentrasi-keahlian/` | **200** | Jurusan/Konsentrasi Keahlian &#8211; SMK 1 | 73 | 7 | 0 | 0 |
| `/kalender-akademik/` | **200** | Kalender Akademik &#8211; SMK 10 November  | 84 | 3 | 0 | 0 |
| `/fasilitas/` | **200** | Fasilitas Sekolah &#8211; SMK 10 November  | 87 | 9 | 0 | 0 |
| `/prestasi/` | **200** | Prestasi &#8211; SMK 10 November Kebumen | 72 | 7 | 0 | 0 |
| `/pendaftaran/` | **200** | Pendaftaran &#8211; SMK 10 November Kebume | 79 | 3 | 0 | 0 |

| Sragen URL | HTTP | Judul | Kata | Aset | Aset hilang | Produksi |
|---|---|---|---:|---:|---:|---:|
| `/kebijakan-privasi/` | **404** | Page not found &#8211; SMK 10 November Sra | 92 | 1 | 0 | 0 |
| `/home/` | **200** | SMK 10 November Sragen | 507 | 16 | 0 | 0 |
| `/berita-artikel/` | **200** | Berita/Artikel &#8211; SMK 10 November Sra | 69 | 1 | 0 | 0 |
| `/kontak/` | **200** | Kontak &#8211; SMK 10 November Sragen | 103 | 2 | 0 | 0 |
| `/sejarah/` | **200** | Sejarah &#8211; SMK 10 November Sragen | 73 | 1 | 0 | 0 |
| `/visi-misi/` | **200** | Visi &#038; Misi &#8211; SMK 10 November S | 163 | 1 | 0 | 0 |
| `/struktur-organisasi/` | **200** | Struktur Organisasi &#8211; SMK 10 Novembe | 72 | 2 | 0 | 0 |
| `/jurusan-konsentrasi-keahlian/` | **200** | Jurusan/Konsentrasi Keahlian &#8211; SMK 1 | 90 | 6 | 0 | 0 |
| `/kalender-akademik/` | **200** | Kalender Akademik &#8211; SMK 10 November  | 88 | 1 | 0 | 0 |
| `/fasilitas/` | **200** | Fasilitas Sekolah &#8211; SMK 10 November  | 95 | 10 | 0 | 0 |
| `/prestasi/` | **200** | Prestasi &#8211; SMK 10 November Sragen | 74 | 5 | 0 | 0 |
| `/pendaftaran/` | **200** | Pendaftaran &#8211; SMK 10 November Sragen | 73 | 1 | 0 | 0 |

## 2. Aset

- **Kebumen & Sragen:** seluruh aset yang dirujuk (CSS/JS/gambar/font) yang diperiksa (maks. 12 unik per halaman) **ADA di disk** → **0 hilang** ✓
- **Tambun:** 0 berkas dirujuk tapi tidak ada di disk
- **Cibitung:** 0 berkas dirujuk tapi tidak ada di disk

## 3. E-05 — rujukan domain produksi

- **Kebumen `/kontak/`: 3 rujukan**; **Cibitung `/kontak/`: 2 rujukan** (dikonfirmasi ulang). Konteks tercetak di atas.
- **Tidak** ada aset/gambar yang diambil dari produksi ✓ · **tidak** ada URL yang saya ubah ✓
- Status: **BUTUH KEPUTUSAN SAKA** — saya tidak tahu tujuan pengganti yang benar.

## 4. Masalah tersisa
| # | Situs | Masalah | Status |
|---|---|---|---|

| E-01 | Tambun | `/kebijakan-privasi/` → **404** setelah uji ulang (000 sebelumnya **transien**) | **draft** — 404 untuk pengunjung anonim adalah perilaku benar; keputusan publikasi = Saka |
| E-02 | Kebumen | `/kebijakan-privasi/` 404 | **draft** — keputusan konten |
| E-03 | Sragen | `/kebijakan-privasi/` 404 | **draft** — keputusan konten |
| E-04 | Kebumen | `/laman-contoh/` masih terbit | Halaman contoh bawaan WP — keputusan konten |
| E-05 | Cibitung (2), Kebumen (3) | rujukan domain produksi di Kontak | **Butuh keputusan Saka** |
| **E-08** | Cibitung | 2 halaman `page` dengan **slug kosong** (draft) + `elementor-1252` | Perlu keputusan: hapus/ganti slug/publikasi |

## 5. Belum diverifikasi
- Navigasi desktop/mobile interaktif, formulir kontak (kirim), tombol `tel:`/`mailto:`/WhatsApp: **BELUM** ✗ (browser/CDP tidak stabil)
- Tautan internal klik-per-klik: **BELUM** (aset sudah dicek per file ✓)
- Visual per halaman: **BELUM**
- Redirect `/home/` tujuan akhir: tercatat di tabel uji ulang ✓

## 6. Status per situs
| Situs | Status |
|---|---|
| Tambun | **LULUS DENGAN CATATAN** |
| Cibitung | **LULUS DENGAN CATATAN** (E-05, E-08) |
| Kebumen | **LULUS DENGAN CATATAN** (E-02, E-04, E-05) |
| Sragen | **LULUS DENGAN CATATAN** (E-03 draft) |

Tidak ada situs yang **LULUS QA LOKAL penuh** — navigasi interaktif, formulir, dan verifikasi visual belum dijalankan.

## 8. Koreksi penting terhadap Fase 8A
Seluruh **HTTP 000** pada Fase 8A (`Tambun /kebijakan-privasi`, `/home`, `/berita-artikel`; `Cibitung /home`, `/berita-artikel`, `/kontak`) **tidak terulang** pada uji ulang — semuanya mengembalikan **200/404 sesuai status**. Penyebabnya **koneksi lokal sementara** saat pengujian beruntun, **bukan** kerusakan situs. `/home/` mengarah ke beranda (`~/`) tanpa loop di keempat situs ✓

## 7. Konfirmasi
Checkpoint: **tidak dibuat** (0 perubahan) ✓ · hosting Rumahweb **tidak diakses** ✓ · produksi **tidak disentuh** ✓ · `goly1264_ams` & `Downloads\admin.php` **tidak disentuh** ✓ · dump/backup/manifest/checksum/pohon bukti/laporan lama **tidak diubah** ✓ · tanpa push/commit ✓ · tanpa rahasia di laporan ✓
