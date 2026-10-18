# FASE 6K-A — HASIL EKSEKUSI PERBAIKAN BANNER ELEMENTOR

**Tanggal:** 18 Oktober 2026 · **Situs:** Cibitung · Kebumen · Tambun (staging) · **Status akhir: KETIGANYA BERHASIL** ✓
**Ringkas:** dynamic tag `page-title` (Elementor Pro) pada heading banner diganti **judul statis "Struktur Organisasi"**; latar/overlay/padding/container **tidak diubah**; CSS Elementor diregenerasi lewat API; semua viewport lulus **tanpa overflow & tanpa error**.

---

## 1. Tujuan dan ruang lingkup
Memperbaiki banner halaman **Struktur Organisasi** di tiga staging (Cibitung, Kebumen, Tambun) yang tampil sebagai **band kosong** karena judulnya memakai dynamic tag `page-title` milik **Elementor Pro** (tidak terpasang). Ruang lingkup: **hanya** heading banner; tanpa mengubah background, overlay, layout, konten, menu, akun, plugin, atau pengaturan global.

## 2. Identitas target (diverifikasi, bukan diasumsikan)
| Situs | Direktori | Database (dari `wp-config.php`) | Prefix | WP | Elementor | Pro |
|---|---|---|---|---|---|---|
| Cibitung | `C:\xampp\htdocs\staging\cibitung` | `smk10_staging_cibitung` ✓ | `wp_` | 7.0.7 | 4.3.5 | **tidak ada** |
| Kebumen | `C:\xampp\htdocs\staging\kebumen` | `smk10_staging_kebumen` ✓ | `wp_` | **7.1.3** | 4.3.5 | **tidak ada** |
| Tambun | `C:\xampp\htdocs\staging\tambun` | `smk10_staging_tambun` ✓ | `wp_` | 7.0.7 | 4.3.5 | **tidak ada** |

Halaman ditemukan **dari slug** `struktur-organisasi` (bukan dari asumsi ID): ketiganya **post 899 "Struktur Organisasi" (publish)**.

## 3. ID elemen yang benar-benar diperiksa
Sama di ketiga situs (berasal dari template kit yang sama) — **diverifikasi satu per satu per situs**:

| Path | ID | Jenis | Temuan |
|---|---|---|---|
| `[0]` | `7743b359` | container (banner) | `background_background: classic` · `background_image` (foto) · overlay `overlay2.png` (id 444) · `background_overlay_color: #01295199` · padding 7em/5em/3em |
| `[0][0]` | `37b2b453` | container `isInner` | kolom 50%, padding 2em, gap tablet 14px |
| `[0][0][0]` | **`60d69bd2`** | **widget heading** | **`title: ""`** + `__dynamic__: page-title` ⇒ dirender **kosong** ⇒ banner kosong |
| `[1][0][0]` | `434d9ae8` | heading (di bawah banner) | **dynamic tag `page-title` yang sama** (id tag `b71d645`) — **TIDAK diubah** (lihat §8) |

## 4. Kondisi awal (terukur, sebelum perubahan)
| Situs | Banner desktop | Banner mobile 390 | Teks heading | Elemen heading di DOM |
|---|---|---|---|---|
| Cibitung | 1200×288 px | 350×112 px | **kosong** | **tidak ada** (Elementor melewati widget kosong) |
| Kebumen | 1200×288 px | 350×112 px | **kosong** | tidak ada |
| Tambun | 1200×288 px | 350×112 px | **kosong** | tidak ada |

Screenshot kondisi awal: `…\staging\<situs>\qa6k\awal@{1366x900,390x844}.png`.

## 5. Perubahan yang dilakukan (per situs, satu per satu)
Mekanisme: **WordPress + Elementor API**, bukan SQL massal —
1. `get_post_meta(899,'_elementor_data')` → `json_decode` → ubah **satu** elemen `60d69bd2`: `settings.title = "Struktur Organisasi"`, hapus `settings.__dynamic__.title`.
2. Simpan dengan **`wp_slash(wp_json_encode(...))`** (format JSON Elementor tetap valid) via `update_post_meta`.
3. `delete_post_meta('_elementor_element_cache')` + regenerasi CSS khusus post ini via **`\Elementor\Core\Files\CSS\Post::create(899)->update()`** (bukan flush global).
4. Tidak ada perubahan pada container, background, overlay, padding, responsif, konten, menu, akun, plugin, tema.

| Situs | Judul diset | `__dynamic__` sesudah | JSON valid | Elemen target ditemukan | CSS diregenerasi |
|---|---|---|---|---|---|
| Cibitung | `Struktur Organisasi` | `[]` (kosong) | **YA** | tepat **1** | ya (API Elementor) |
| Kebumen | `Struktur Organisasi` | `[]` | **YA** | tepat **1** | ya |
| Tambun | `Struktur Organisasi` | `[]` | **YA** | tepat **1** | ya |

**Container induk tetap utuh** (diverifikasi sesudah): Cibitung `classic` + `#01295199` + foto id 78 · Kebumen idem + foto id **1477** · Tambun idem + foto id 1215.

## 6. Checkpoint & cadangan (tidak berisi rahasia)
| Situs | Checkpoint SQL | Tabel | Cadangan data Elementor | SHA-256 (16 char) |
|---|---|---|---|---|
| Cibitung | `recovery-work\staging\checkpoints\checkpoint-6k-a-pre-cibitung.sql` (1.074.709 B) | 24 | `…\staging\cibitung\backup-6k-a\elementor-data-post899-SEBELUM.json` (4.184 B) | `df916d4789448d99…` |
| Kebumen | `…\checkpoint-6k-a-pre-kebumen.sql` (2.121.558 B) | 13 | `…\staging\kebumen\backup-6k-a\elementor-data-post899-SEBELUM.json` (4.220 B) | `b7b45cd144fe72a3…` |
| Tambun | `…\checkpoint-6k-a-pre-tambun.sql` (971.684 B) | 24 | `…\staging\tambun\backup-6k-a\elementor-data-post899-SEBELUM.json` (4.217 B) | `4e6f1f36d1c56375…` |

Cadangan **dibaca ulang & divalidasi sebagai JSON** sebelum perubahan ✓; disimpan di area staging/kerja, **bukan** repository publik ✓; tidak memuat kredensial ✓.

## 7. Hasil pengujian (sesudah perubahan)
| Situs | 360×800 | 390×844 | 412×915 | 768×1024 | 1366×768/900 |
|---|---|---|---|---|---|
| Cibitung | banner 320×143 · judul ✓ · 0 overflow · 0 error | 350×143 ✓ | 372×143 ✓ | 728×262 ✓ | 1200×355 ✓ |
| Kebumen | 320×143 ✓ | 350×143 ✓ | 372×143 ✓ | 728×262 ✓ | 1200×355 ✓ |
| Tambun | 320×143 ✓ | 350×143 ✓ | 372×143 ✓ | 728×262 ✓ | 1200×355 ✓ |

Pada **semua** situs & viewport: judul **tampil tepat 1×** di teks halaman · **0 sisa** `page-title`/`elementor-tag` di HTML · **overflow 0 px** (desktop −15 px = lebar scrollbar) · **0 elemen meluber** · **0 error JavaScript**.
Bukti visual: `…\staging\<situs>\qa6k\sesudah@{390x844,1366x900}.png`.
- **Kebumen**: banner menampilkan **foto asli + overlay biru + judul putih** — persis desain semestinya ✓
- **Cibitung & Tambun**: band biru-kelabu (overlay) + judul putih jelas ✓; **foto latar tidak tampil** (lihat §8)
- Judul **tidak muncul dua kali** ✓ (heading kedua `434d9ae8` tetap kosong, tidak diubah)

**Regresi halaman lain:** tidak ada — `/`, `/kontak`, `/home`, `/berita-artikel`, `/struktur-organisasi` = **200** di ketiga situs; overflow beranda 0 px. (Catatan: `/profil` & `/akademik` yang sempat 404 adalah **slug uji saya yang salah**, bukan halaman yang ada.)

## 8. Temuan yang belum diselesaikan (di luar lingkup 6K-A, tidak diubah)
1. **Lampiran background hilang di database** ⇒ foto banner tidak dapat dirender oleh Elementor:
   | Situs | `background_image.id` | Status di DB |
   |---|---|---|
   | Cibitung | 78 | **tidak ada** ✗ (berkas `2025/12/header_001.webp` ada di disk) |
   | Kebumen | 1477 | **ada** ✓ (foto tampil) |
   | Tambun | 1215 | **tidak ada** ✗ |
   | ketiga | `background_overlay_image` id **444** | **tidak ada** ✗ (overlay **warna** `#01295199` tetap tampil) |
   → Penyebab: ID lampiran pada data Elementor masih merujuk ID **sumber**, tidak dipetakan ke ID staging. Perbaikan memerlukan pemetaan/ pembuatan lampiran (perubahan data terpisah) — **butuh persetujuan**.
2. **Heading kedua `434d9ae8`** memakai dynamic tag `page-title` yang sama dan tetap kosong. Tidak diubah karena mengisinya akan membuat judul tampil **dua kali**. Perlu keputusan: isi dengan teks lain, sembunyikan, atau biarkan.
3. Tinggi banner 355 px di desktop (dari 288 px) akibat judul kini mengisi ruang 7em padding. Bila dianggap terlalu tinggi, opsi: kurangi `padding` (mis. 7em→4em) — **belum dilakukan** (perubahan desain, butuh persetujuan).

## 9. Rollback (tidak diperlukan — semua lulus)
Bila perlu per situs: pulihkan `_elementor_data` post 899 dari `backup-6k-a\…-SEBELUM.json` (via `update_post_meta` + `wp_slash`), hapus `_elementor_css` & `_elementor_element_cache`, lalu `Post::create(899)->update()`; atau restore `checkpoint-6k-a-pre-<situs>.sql`. **Tidak ada rollback yang dijalankan.**

## 10. Hal yang belum diuji / keterbatasan
- Pengujian memakai **emulasi viewport CDP (Chrome headless)** — **bukan perangkat Android/fisik**.
- Console diperiksa melalui hook `window.onerror` + `console.error`, bukan DevTools interaktif.
- Tidak ada pengujian dengan pembaca layar, mode gelap, atau browser selain Chrome.
- Foto banner Cibitung/Tambun belum dapat diuji karena lampiran tidak ada (§8.1).
- 768×1024 hanya diuji untuk halaman Struktur Organisasi.

## 11. Status akhir
| Situs | Status |
|---|---|
| **Cibitung** | **BERHASIL** ✓ (judul tampil jelas; foto latar menunggu perbaikan lampiran) |
| **Kebumen** | **BERHASIL** ✓ (judul + foto + overlay tampil sempurna) |
| **Tambun** | **BERHASIL** ✓ (judul tampil jelas; foto latar menunggu perbaikan lampiran) |

## 12. Konfirmasi
- **Situs produksi tidak disentuh**; tidak ada deployment, DNS, hosting, atau database produksi.
- **Database `goly1264_ams`: tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah** ✓
- **`C:\Users\sakha\Downloads\admin.php`: tidak disentuh** — tidak dibaca, dijalankan, dipindai, diedit, dipindah, diganti nama, atau dihapus ✓
- Tidak ada impor dump sumber; tidak ada unduhan media produksi; tidak ada Elementor Pro/plugin dipasang ✓
- Tidak ada `git push`/deployment ✓ · Laporan **tanpa** password/hash/token/cookie/rahasia ✓
- Catatan: clone lokal repository `vael-skaya/wp` **tidak ditemukan** di host ini ⇒ laporan disimpan di direktori kerja terverifikasi `recovery-work\`.
