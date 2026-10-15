# FASE 6C — PERAPIAN UI, NAVBAR, IKON, DAN PENDAFTARAN: TAMBUN

**Tanggal:** 15 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\tambun` · DB `smk10_staging_tambun` · `http://localhost/staging/tambun/`
**Hasil utama:** **ikon kontak yang hilang dipulihkan** · **lokasi menu Astra ditugaskan** (header berhenti memakai daftar-halaman fallback) · plugin formulir gratis terpasang · logo dibiarkan (bukan berkas milik situs lain)

---

## 1. Checkpoint yang digunakan

| Checkpoint | Ukuran | Waktu |
|---|---|---|
| `checkpoint-6c-pre-tambun.sql` | 926.020 B (13 tabel, uji baca OK) | sebelum perubahan 6C |
| `checkpoint-6b-pre-tambun.sql` | 760.408 B | sebelum 6B |
| `checkpoint-6a-tambun.sql` | 789.727 B | pasca-pemulihan 6A |

## 2. Masalah dan bukti sebelum perubahan

### 2.1 Header memakai daftar-halaman fallback (bukan menu)
- HTML beranda memuat **11 elemen `page_item page-item-<id>`** dan label menu yang urutannya tidak sama dengan menu di DB (Main Menu 12 item, Footer 7 item).
- `nav_menu_locations` **kosong** → Astra menampilkan seluruh halaman sebagai navigasi. (Tambun tidak menampilkan judul tak berjudul seperti Cibitung karena halaman "Laman Contoh"/"Hello world" sudah dihapus di 6A.)

### 2.2 Ikon kontak hilang (masalah inti fase ini)
- **DOM:** `<i class="icon icon-whatsapp-2">`, `icon-phone-handset`, `icon-envelope1`, `icon-map-marker1` — **tidak ada CSS/font ikon yang dimuat** (0 rujukan).
- **Data:** `"selected_icon":{…,"library":"ekiticons"}` = pustaka **ElementsKit Lite** yang tidak dipulihkan.
- **Cakupan:** **7 post** (10 Home, 792 header, 826 Kontak, 895 Sejarah, 897, 903, 1155) ≈ **31 ikon**, 12 nama berbeda (`whatsapp-2`, `phone-handset`, `envelope1`, `map-marker1`, `idea1`, `dart-board`, `graduation-hat`, `bank1`, `team`, `briefcase`, `internet`, `customer-support-2`, `calendar2`).

## 3. Akar masalah yang terverifikasi

1. **Ikon:** pustaka ikon `ekiticons` (ElementsKit) tidak ada di staging → `<i>` dirender tanpa font. **Bukan** ikon Pro, **bukan** ikon tersembunyi CSS, **bukan** widget tidak dirender (ikon ada di DOM).
2. **Menu/header:** lokasi menu Astra kosong → fallback `wp_page_menu` (daftar halaman). `astra-settings` Tambun hanya 92 B (default, sama seperti Cibitung).
3. **Warning kit** (dari 6B): data kit warisan Elementor 3.30.1 tanpa varian tablet/mobile → sudah diperbaiki di 6B dan tidak berulang.

## 4. Perubahan yang dilakukan

| Objek | Perubahan | Nilai lama | Nilai baru |
|---|---|---|---|
| `theme_mods_astra` (`nav_menu_locations`) | penugasan lokasi menu | kosong | `primary`→**3 (Main Menu)**, `footer_menu`→**7 (Footer Menu)**, `mobile_menu`→**3** |
| `wp_postmeta._elementor_data` post **10, 792, 826, 895, 897, 903, 1155** | **31 ikon** `ekiticons` → Font Awesome bawaan Elementor | `{"value":"icon icon-whatsapp-2","library":"ekiticons"}` … | `fab fa-whatsapp`/`fas fa-phone|envelope|map-marker-alt|calendar-alt|lightbulb|bullseye|graduation-cap|university|users|briefcase|globe|headset` dengan `fa-brands`/`fa-solid` |
| Plugin | `wpforms-lite` dipasang & diaktifkan | — | v**2.0.2.2** (repo resmi, `requires_wp=5.5` ✓) |
| Cache | `wp elementor flush-css` | — | CSS diregenerasi |

**Tidak diubah:** logo (`site_logo` 1316 dibiarkan — bukan berkas milik situs lain), konten, halaman, item menu, `default-kit-2`, draft, tema, berkas inti.

`JSON_VALID` seluruh `_elementor_data` = **0 invalid** setelah penulisan.

## 5. Hasil sebelum dan sesudah

| Pemeriksaan | Sebelum | Sesudah |
|---|---|---|
| Ikon kontak | `<i>` tanpa font | **7 SVG** `e-fab-whatsapp`, `e-fas-phone`, `e-fas-envelope`, `e-fas-map-marker-alt`, `e-fab-facebook-f/instagram/tiktok` |
| `ekiticons` di DB | 7 post | **0** |
| Navigasi header | daftar halaman fallback (11 `page_item`) | menu asli yang ditugaskan (`primary`=Main Menu) |
| Beranda HTTP | 200 | **200** |
| Tautan internal (15) | — | **0 non-200** |
| Warning kit | (6B) tampil di beranda → hilang | **0 baru** |
| JSON Elementor invalid | 0 | **0** |
| IoC | 0 | **0** |

## 6. Screenshot
- **Sebelum (6B):** warning tercetak di beranda (bukti visual tersimpan).
- **Sesudah:** tombol toggle menu, hero, WELCOME tampil normal (screenshot 6B). Untuk 6C, preview pane kembali menampilkan kanvas lama saat berpindah tab/URL → **tidak dipakai sebagai bukti**; verifikasi dilakukan pada DOM/HTML (`curl`) dan DB.

## 7. Pengujian desktop dan mobile
- **Mobile/sempit: teruji** — toggle menu berfungsi, tidak ada overflow.
- **Desktop lebar penuh: belum teruji** (tanpa kontrol viewport).

## 8. Pengujian formulir dan tautan kontak
- **WPForms Lite** aktif; **formulir belum dibuat/ditempel** (`<form>` = 0 di halaman Kontak). Pembuatan form butuh builder WPForms (UI) atau API internal versi-spesifik → tidak saya tebak.
- **Email tidak dapat diuji** (tanpa SMTP di XAMPP) → belum dinyatakan siap produksi.
- Alamat email sumber: `email@smk10novembertambun.sch.id` (bentuk lokal "email@" mencurigakan/pengisi contoh) → **perlu konfirmasi sekolah** sebelum dipakai sebagai penerima.
- Semua tautan menu internal 200.

## 9. Informasi resmi / keputusan yang masih dibutuhkan
1. **Formulir Kontak:** alamat penerima yang sah + persetujuan membuat & menempelkan formulir.
2. **Halaman Pendaftaran** berisi teks **"Halaman dalam pengembangan…"** yang **memang tersimpan** di sumber (konten 45 B) dan tidak rusak layout → informasi resmi PPDB harus dari sekolah (**tidak boleh dikarang**).
3. **Draft "Kebijakan Privasi"** — diperiksa read-only: kontennya praktis kosong (19 B, hanya spasi). Belum dipublikasikan/dihapus.
4. **`default-kit-2`** (ID 7): **bukan** kit aktif (`elementor_active_kit=433`), **tidak dipakai** oleh postmeta mana pun (0), kondisi Theme Builder 0 → inert. **Tidak dihapus** menunggu keputusan.
5. **Widget Pro** yang belum tergantikan: `posts` ×3, `testimonial-carousel` ×2, `archive-posts`, `post-info`, `share-buttons`, `search-form`, `form` (+`nav-menu` di templat header/footer yang tidak aktif). Penggantian hanya akan dilakukan bila konten sumbernya sah dan penggantinya benar-benar berfungsi (belum dilakukan).

## 10. Prosedur rollback

```
C:\xampp\mysql\bin\mysql.exe -u root smk10_staging_tambun < recovery-work\staging\checkpoints\checkpoint-6c-pre-tambun.sql
```
Mengembalikan seluruh perubahan 6C. Plugin: `wp plugin deactivate wpforms-lite`. Tidak ada berkas tema/plugin/int WordPress yang diubah, jadi rollback berkas tidak diperlukan.
