# FASE 8G — QA PENUTUP EMPAT STAGING

**Tanggal:** 22 Oktober 2026 · STAGING SAJA · **0 perubahan data/konten** · Produksi/Rumahweb tidak disentuh · Tanpa push/commit

## 1. Keyboard & fokus — PASS (fokus terlihat) ✓
Tab 6 langkah di beranda desktop, keempat situs:

| Urutan | Elemen | Fokus terlihat |
|---|---|---|
| 1 | `a` "Lewati ke konten" (skip link) | **ya** (outline + shadow) ✓ |
| 2 | `a` (logo) | **ya** (outline) ✓ |
| 3 | `a` "SMK 10 November …" | **ya** ✓ |
| 4 | `a` "Home" | **ya** ✓ |
| 5 | `a` "Profil" | **ya** ✓ |
| 6 | `span` "Menu Toggle"/"Toggle Profil" | **ya** ✓ |

Urutan **masuk akal** (skip-link → identitas → navigasi) ✓ · tidak ada fokus hilang/invisible ✓

## 2. Escape pada menu mobile — **FAIL** ✗
| Situs | Menu terbuka (`aria-expanded`) | Setelah **Escape** |
|---|---|---|
| Tambun | true | **true** ✗ tidak menutup |
| Cibitung | true | **true** ✗ |
| Kebumen | true | **true** ✗ |
| Sragen | true | **true** ✗ |

⇒ Buka/tutup dengan tombol **berfungsi** (Fase 8F ✓), tetapi **Escape tidak menutup menu** ✗. **Tidak diperbaiki** — ini perilaku tema Astra dan perbaikannya butuh JS tambahan (di luar "perbaikan minimal yang terbukti aman"). **Dicatat sebagai temuan aksesibilitas** untuk keputusan Anda.

## 3. Formulir — **TIDAK BERLAKU** (bukan "lulus")
| Situs | `<form>` di DOM |
|---|---|
| Tambun | **0** |
| Cibitung | **0** (6 elemen berkelas mengandung "form" = artefak CSS, bukan form) |
| Kebumen | **0** |
| Sragen | **0** |

⇒ Tidak ada form aktif ⇒ **tidak ada uji form** yang bisa dinyatakan lulus ✓. Tidak ada submit dicoba, tidak ada data dikirim keluar ✓.

## 4. Screenshot & inspeksi visual
**16 PNG** dibuat: `qa-8g/<situs>-<home|kontak>-<desktop|mobile>.png` (1366×900 dan 390×844).

**Inspeksi visual yang benar-benar dilakukan: 1 dari 16** — `sragen-kontak-mobile.png`. Hasilnya:
- **Keempat ikon Kontak TERLIHAT** ✓ — pin lokasi, amplop, dan telepon tampil sebagai bentuk jelas di dalam lingkaran abu-abu ⇒ **perbaikan mask SVG Fase 7.6 TERBUKTI BERHASIL** (verifikasi piksel yang sebelumnya tertunda) ✓
- Header rapi (logo + "SMK 10 November Sragen" + tombol menu) ✓, tidak ada teks terpotong/tumpang tindih ✓
- ⚠️ Terdapat **blok abu-abu kosong** di bawah judul "Kontak" (banner Elementor tanpa isi) — **temuan kosmetik**, bukan bug fungsional ✗

**15 screenshot lain BELUM saya periksa** ✗ — saya tidak mengklaim inspeksi visual lulus untuk halaman itu.

## 5. Cakupan halaman × viewport
| Situs | Terukur sebelumnya (8F: beranda+kontak @390/768/1366) | Seluruh halaman @390/768/1366 |
|---|---|---|
| Tambun | ✓ | **BELUM** ✗ |
| Cibitung | ✓ | **BELUM** ✗ |
| Kebumen | ✓ | **BELUM** ✗ |
| Sragen | ✓ (+ 7 halaman @390/412/768/1366 di Fase 7.4) | n/a |

## 6. Sisa item BLOCKED / NOT TESTED
- Keyboard **Shift+Tab** dan **Enter** pada tautan: **NOT TESTED** ✗ (Tab saja yang diuji)
- Seluruh halaman × 3 viewport untuk Tambun/Cibitung/Kebumen: **NOT TESTED** ✗
- Inspeksi 15 screenshot: **NOT TESTED** ✗
- Escape menutup menu: **FAIL** ✗
- Blok abu-abu kosong di bawah judul halaman: **BELUM DIINVESTIGASI** ✗

## 7. Keputusan
# **BELUM SIAP**
Alasan konkret:
1. Cakupan halaman × viewport belum lengkap untuk 3 situs (hanya beranda + kontak).
2. `Escape` tidak menutup menu mobile (4/4 situs).
3. Inspeksi visual baru 1 dari 16 screenshot.
4. Shift+Tab/Enter belum diuji.
5. (Tetap) hosting: HTTPS, canonical redirect, DNS, PHP & `memory_limit` — **tidak dapat diverifikasi dari staging**.

**Yang sudah terbukti:** HTTP 200 semua halaman publish · 0 aset 4xx · 0 console error · 0 overflow halaman · menu mobile buka/tutup via tombol ✓ · fokus keyboard terlihat ✓ · locale `id_ID` ✓ · **ikon Kontak terverifikasi secara visual** ✓

## 8. Konfirmasi batasan
Staging saja ✓ · 0 perubahan DB/konten ✓ · `elementor-1252`, anchor `#`/`#akademik`, trash 8D, draf privasi tidak disentuh ✓ · `goly1264_ams` & `Downloads\admin.php` tidak disentuh ✓ · dump/evidence/manifest/checksum/checkpoint tidak diubah ✓ · tanpa push/commit/ubah repo ✓ · tanpa rahasia di laporan ✓

## 9. Artefak
`FASE-8G-QA-PENUTUP-HASIL.md` · screenshot: `recovery-work/qa-8g/*.png` (16 berkas) · data: `qa-8g/_8g.json`
