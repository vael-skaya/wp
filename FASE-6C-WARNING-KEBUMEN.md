# FASE 6C — WARNING ELEMENTOR DI KEBUMEN (perbaikan terbatas)

**Tanggal:** 15 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\kebumen` · DB `smk10_staging_kebumen` · `http://localhost/staging/kebumen/`
**Kesimpulan:** akar masalah **terverifikasi** dan **diperbaiki** dengan perubahan data minimal; **tidak ada berkas inti Elementor yang diedit** dan **pelaporan error tidak dimatikan**.

---

## 1. Checkpoint

| Checkpoint | Ukuran | Keterangan |
|---|---|---|
| `recovery-work\staging\checkpoints\checkpoint-6c-pre-kebumen.sql` | **1.933.266 B** (13 tabel, uji baca OK) | checkpoint khusus sebelum perubahan 6C di Kebumen |
| `checkpoint-05-pre-5e1.sql` | 2.293.789 B | checkpoint fase 5E.1 (tidak ditimpa) |
| `checkpoint-04-pre-5d.sql`, `checkpoint-03…`, `checkpoint-02…` | — | tidak ditimpa |

## 2. Masalah dan bukti sebelum perubahan

- Warning yang dilaporkan: `Undefined array key "body_background_image_tablet"` pada `wp-content/plugins/elementor/includes/base/controls-stack.php` baris 1499, muncul pada halaman depan/`/kontak/`.
- **Bukti sebelum perbaikan (query read-only):** kit aktif Kebumen = **post 433** (`global-kit-styles`), `_elementor_page_settings` panjang **23.576 B** dengan **9 key `body_background_*`**, dan **10 varian tablet/mobile HILANG**:
  `body_background_background_tablet/mobile`, `body_background_color_tablet/mobile`, `body_background_image_tablet/mobile`, `body_background_position_tablet/mobile`, `body_background_repeat_tablet/mobile`.
- Bandingan: kondisi ini **identik** dengan Cibitung/Tambun sebelum diperbaiki (bukan masalah khas Kebumen), dan berasal dari data kit era **Elementor 3.30.1**.
- **Yang TIDAK dilakukan:** tidak mengedit `controls-stack.php`, tidak mematikan `display_errors`, tidak mengubah metadata kit secara massal, tidak menganggap otomatis bahwa masalah sama tanpa verifikasi (diverifikasi: key yang hilang memang ada 10 dan nama warning cocok).

## 3. Akar masalah (terverifikasi)

Kit aktif menyimpan pengaturan background **desktop** tetapi tidak menyimpan varian **tablet/mobile**. Elementor 4.3.5 mengakses key varian itu saat menggenerate CSS kit tanpa pemeriksaan → PHP warning (non-fatal, tetapi tercetak bila `display_errors` aktif). **Penyebabnya data kit yang tidak lengkap + selisih versi Elementor**, bukan tema, bukan migrasi konten.

## 4. Perubahan yang dilakukan (minimal, terarah)

| Objek | Perubahan | Sebelum | Sesudah |
|---|---|---|---|
| `wp_postmeta._elementor_page_settings`, post **433** | melengkapi varian tablet/mobile dengan **menyalin nilai desktop** (semantik "tanpa override terpisah" tetap sama) | 9 key `body_background_*`, 10 varian hilang | **17 key**, 12 varian tablet/mobile ada; **8 key ditambahkan** oleh skrip ("key ditambahkan: 8") |
| Cache CSS Elementor (`uploads/elementor/css/*`) | `wp elementor flush-css` | — | CSS diregenerasi |

Penulisan dilakukan lewat API WordPress (`update_post_meta` dengan **array**, bukan SQL langsung) sehingga **tidak terjadi serialisasi ganda**. Tidak ada berkas inti plugin/tema yang disentuh.

## 5. Hasil sebelum dan sesudah

| Uji | Sebelum | Sesudah |
|---|---|---|
| Warning di HTML `/kontak/` | terjadi (`body_background_image_tablet`) | **0 kemunculan** |
| Warning baru di log Apache setelah `flush-css` + 7 halaman diuji (beranda, kontak, sejarah, visi-misi, fasilitas, berita-artikel, prestasi) | — | **0 tambahan** (13 → 13) |
| Beranda HTTP | 200 | **200** |
| JSON Elementor invalid | 3 (revisi lama, sudah ada sejak fase 5) | **3** (tidak berubah — bukan akibat perubahan ini) |

## 6. Perubahan lain di Kebumen pada fase ini (dan yang dikembalikan)

- **Ikon kontak** (bagian §4 fase): 22 ikon `library:"ekiticons"` pada post 10, 792, 799, 826, 1155 diganti ke **Font Awesome bawaan Elementor** → halaman `/kontak/` kini menampilkan **7 SVG** `e-fab-whatsapp`, `e-fas-phone`, `e-fas-envelope`, `e-fas-map-marker-alt`, dst; sisa `ekiticons` di DB = **0**. Dibuat pada checkpoint yang sama (`checkpoint-6c-pre-kebumen.sql`).
- **Dikembalikan ke keadaan semula:** saat menguji perbaikan lokasi menu, saya sempat menugaskan menu "Footer Menu" ke lokasi `footer_menu` di Kebumen — **sudah dicabut kembali**, sehingga Kebumen tetap pada penugasan aslinya (`primary`=11 Main Menu, `secondary_menu`=12 Footer Menu). *(Ini saya laporkan karena sempat berubah, meski akhirnya kembali.)*
- **Catatan (tidak diubah):** navigasi Kebumen masih memuat **13 elemen `page_item`** fallback (mis. "Laman Contoh" ikut tampil) karena lokasi `mobile_menu`/`footer_menu` tidak ditugaskan. Perbaikan analog Cibitung/Tambun bisa diterapkan bila Anda setujui.

## 7. Pengujian desktop dan mobile
- Breakpoint sempit (pane): teruji, tidak ada overflow.
- Desktop lebar penuh: belum teruji (tanpa kontrol viewport).

## 8. Keterbatasan
- Screenshot "sesudah" tidak diandalkan: preview pane menampilkan kanvas lama ketika tab/URL bertukar → verifikasi memakai HTML (`curl`) dan query DB.
- Warning diuji pada 7 halaman; halaman lain tidak dimuat satu per satu. Karena sumbernya adalah **generate CSS kit** (satu kali per kit), cakupan 7 halaman dianggap memadai, namun tidak saya klaim sebagai bukti menyeluruh.

## 9. Keputusan yang diperlukan
1. Setujui (atau tolak) penugasan menu Kebumen (`mobile_menu` → Main Menu) untuk menghilangkan sisa daftar-halaman fallback.
2. Apakah 3 nilai `_elementor_data` yang invalid (revisi lama, tidak dirender) perlu ditangani — **tidak** termasuk masalah warning ini.

## 10. Prosedur rollback

```
C:\xampp\mysql\bin\mysql.exe -u root smk10_staging_kebumen < recovery-work\staging\checkpoints\checkpoint-6c-pre-kebumen.sql
```
Mengembalikan kit (varian tablet/mobile) **dan** penggantian ikon 6C di Kebumen sekaligus. Untuk mengembalikan hanya CSS: hapus `uploads/elementor/css/*` lalu `wp elementor flush-css`.
