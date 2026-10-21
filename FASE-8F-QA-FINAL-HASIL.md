# FASE 8F — QA FINAL STAGING EMPAT SITUS

**Tanggal:** 22 Oktober 2026 · STAGING SAJA · produksi/Rumahweb tidak disentuh · tanpa push/commit · **0 perubahan data/config fase ini**

## 1. QA browser — BERHASIL dijalankan (24 pemuatan)
Chrome headless/CDP, 4 situs × 3 viewport (390/768/1366) × 2 halaman (beranda, kontak). Gambar diperiksa **setelah scroll ke bawah + tunggu 1,8 s** (lazy-load diberi waktu).

| Situs | Overflow 390/768/1366 | Gambar rusak | Resource 4xx | Console error | `lang` | Logo (390/768/1366) |
|---|---|---|---|---|---|---|
| Tambun | 0 / 0 / −15 | **0** | 0 | [] | id ✓ | 48/48/72 ✓ |
| Cibitung | 0 / 0 / −15 | **0** | 0 | [] | id ✓ | 48/48/72 ✓ |
| **Kebumen** | 0 / 0 / −14 | **2 (390)** ⚠️ | 0 | [] | id ✓ | 44/48/60 ✓ |
| Sragen | 0 / 0 / −15 | **0** | 0 | [] | id ✓ | 48/48/72 ✓ |

(−15/−14 = scrollbar vertikal, bukan overflow ✓)

**Gambar Kebumen @390:** 2 gambar `naturalWidth=0` **setelah** lazy-load diberi waktu → **bukan** sekadar lazy (perlu dicatat sebagai temuan).

**Sragen 14/17 `naturalWidth=0` dari Fase 8E: TERBANTAH** ✓ — setelah tunggu + scroll, **0 gambar rusak**; sisanya `lazyPending` normal ✓

## 2. Menu mobile — PASS ✓ (koreksi temuan sebelumnya)
| Situs | `aria-expanded` sebelum → sesudah klik |
|---|---|
| Tambun | **false → true** ✓ |
| Cibitung | **false → true** ✓ |
| Kebumen | **false → true** ✓ |
| Sragen | **false → true** ✓ |

⇒ Menu **berfungsi**. Laporan sebelumnya ("tidak terbuka") adalah **kesalahan pengukuran saya** (selector panel salah), bukan bug situs ✓

## 3. Kebumen — "overflow" terbukti **false positive** ✓
Diagnosis lanjutan menunjukkan **seluruh** elemen yang "meluber" adalah **slide carousel Swiper**:
`div.swiper-slide.swiper-slide-next` (x=693 @390 · x=2446 @1366), `figure.swiper-slide-inner`, `img.swiper-slide-image`, dan `swiper-slide-duplicate`.

- Slide di sebelah kanan viewport **memang berada di luar layar** — itu perilaku normal carousel ✓
- `document.scrollWidth - innerWidth` = **0** ⇒ **tidak ada overflow halaman** ✓
- Gambar `naturalWidth=0` di Kebumen = `src=""` + `loading=lazy` + `swiper-slide-image` ⇒ **placeholder slide duplikat**, bukan aset rusak ✓

⇒ **Tidak ada bug layout Kebumen.** Tidak ada perbaikan yang diperlukan, dan **tidak ada perubahan** yang saya lakukan.

## 4. Interaksi lain
| Tes | Status |
|---|---|
| Menu mobile buka/tutup | **PASS** (aria-expanded) |
| Keyboard Tab/Enter/Escape + focus visible | **NOT TESTED** ✗ |
| Formulir (validasi) | **NOT TESTED** ✗ (tidak ada form aktif terdeteksi; submit tidak dicoba) |
| `mailto:`/`tel:`/WhatsApp | **PASS statis** (8C); Cibitung/Kebumen Kontak menampilkan 2–3 `mailto:` resmi ✓ |
| URL produksi pada render | **0** di semua situs ✓ (kecuali domain email `mailto:` ✓) |
| Locale Sragen | **id_ID** ✓ (`lang="id"` di keempat situs) |

## 5. Gerbang kesiapan produksi
### Status: **SIAP DITINJAU** (bukan "siap produksi")
**Terbukti di staging ✓**
- HTTP semua halaman publish **200** (Tambun 11 · Cibitung 12 · Kebumen 11 · Sragen 11) + `/home` → 301 ke `/` ✓
- **0 gambar rusak** (setelah lazy-load diberi waktu) · **0 resource 4xx** · **0 console error** di 24 pemuatan
- Overflow halaman **0** di 390/768/1366 · logo 48/72 px benar (Kebumen 44/48/60 sesuai CSS-nya) ✓
- **Menu mobile berfungsi**: `aria-expanded` false → **true** di keempat situs ✓
- Locale **id_ID** di keempat situs ✓ · aset lokal resolvable ✓ · checkpoint lengkap ✓

**Belum diuji ✗ (harus ditutup sebelum migrasi)**
- Keyboard Tab/Enter/Escape + focus visible
- Formulir (validasi & pesan) — tidak ada form aktif terdeteksi
- Halaman selain beranda & kontak pada 390/768/1366 untuk Tambun/Cibitung/Kebumen (Sragen sudah 7 halaman @390/412/768/1366 di Fase 7.4 ✓)
- Screenshot visual (tidak diambil)

**Tidak dapat diverifikasi dari staging ⚠️:** HTTPS, canonical redirect, DNS, versi PHP & `memory_limit` di hosting (staging **40M**, target ≥**256M**), pengiriman email.

## 6. Konfirmasi batasan
Staging saja ✓ · `goly1264_ams` & `Downloads\admin.php` tidak disentuh ✓ · dump/evidence/manifest/checksum/checkpoint lama tidak diubah ✓ · tanpa hapus permanen & tanpa mengulang 8D ✓ · `elementor-1252` & anchor `#`/`#akademik` tidak diubah ✓ · tanpa push/commit/perubahan repo ✓ · tanpa rahasia di laporan ✓

## 7. Artefak
`FASE-8F-QA-FINAL-HASIL.md` · data: `sragen/export/_8f.json`, `_8f_diag.json` · **screenshot: tidak diambil** ✗
