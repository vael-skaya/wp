# FASE 8D TERPADU — STAGING CLEANUP + AUDIT + QA + PRECHECK PRODUKSI

**Tanggal:** 21 Oktober 2026 · Perubahan hanya di **staging** · Rumahweb/produksi **tidak disentuh** · Tanpa push/commit

## 1. Perubahan yang benar-benar dilakukan

| Situs | Tindakan | Checkpoint (baru, unik) | Hasil |
|---|---|---|---|
| Kebumen | Trash page ID **2** `/laman-contoh/` | `checkpoint-8d-pre-hapus-laman-contoh-20261021-185137.sql` — **2.138.877 B** · SHA `a7c81a80…` | `Success: Trashed post 2` → status **trash**; URL kini **404** ✓ |
| Cibitung | Trash draft **1275, 1281, 1341** | `checkpoint-8d-pre-hapus-3-draft-20261021-185137.sql` — **1.133.788 B** · SHA `f325130d…` | 3× `Trashed post` → status **trash** ✓ |

**Verifikasi referensi sebelum hapus:** keempat ID → `menu=0`, anak=0, konten yang menaut=0 ✓
> Catatan jujur: pemeriksaan "dirujuk Elementor" untuk Kebumen ID 2 memakai pola `LIKE '%"2"%'` yang **longgar** dan mengembalikan 17 hit palsu (angka `2` di JSON mana pun). Bukti kuatnya: isi halaman = teks contoh WordPress, tidak ada di menu, tidak punya anak, tidak ditautkan — dan halaman itu memang menunjukkan teks contoh bawaan ✓

**Tidak ada** checkpoint lama yang ditimpa ✓ · penghapusan memakai **trash** (dapat dipulihkan) ✓ · **tidak** ada perubahan pada `#akademik`/`#`, `elementor-1252`, desain, atau konten editorial ✓

## 2. Audit Elementor #1252 (read-only, TIDAK diubah)
| Properti | Nilai |
|---|---|
| Judul / status / slug | `Elementor #1252` / **publish** / `elementor-1252` |
| Template halaman | `default` |
| Konten | 553 B — container + widget gambar (aset lokal `localhost/staging/cibitung/...`) |
| Di menu | **tidak** (0) |
| Dirujuk `_elementor_data` halaman lain | **tidak** (0) |
| Dipakai sebagai header/footer template | **tidak** (0) |
⇒ **Tidak ada dependensi**; halaman **yatim**. Sesuai instruksi: **dipertahankan**, menunggu keputusan Anda.

## 3. QA pasca-perubahan (dijalankan ulang pada situs yang berubah)
| Situs | Halaman publish diuji | Hasil |
|---|---:|---|
| Kebumen | 11 | **11× 200**, kata 70–773, **0 aset hilang** ✓ |
| Cibitung | 12 | **11× 200** + `/home` → **301** ke `/` ✓, **0 aset hilang** ✓ |
| Tambun / Sragen | — | **tidak diubah** ⇒ tidak diuji ulang (sesuai efisiensi) |

## 4. Fakta lingkungan (precheck)
| Item | Nilai |
|---|---|
| PHP | **8.2.12** (keempat situs) |
| MariaDB | **10.4.32** |
| WordPress | Tambun/Cibitung/Sragen **7.0.7**, Kebumen **7.1.3** |
| Tema | Astra **4.14.0** (Sragen **4.14.1**) |
| `WP_MEMORY_LIMIT` | **40M** ⚠️ (rendah untuk produksi) |
| Unggahan maks | 512 MB |
| Zona waktu | `Asia/Jakarta` ✓ |
| Bahasa | `id_ID` ✓ **kecuali Sragen `en_US`** ⚠️ |
| HTTPS lokal | tidak (wajar) — **wajib di produksi** |
| Plugin aktif | Kebumen 1, lainnya 2 |

## 5. Precheck produksi (read-only, belum ada tindakan)
**Yang dibutuhkan sebelum migrasi:**
1. **Domain/HTTPS**: 4 domain sekolah + sertifikat SSL aktif; `siteurl`/`home` diganti **setelah** masuk hosting, dengan `wp search-replace` terkontrol (bukan sebelum).
2. **PHP**: minimal 8.1, **`memory_limit` ≥ 256M** (staging 40M ⚠️).
3. **Database**: MySQL 8/MariaDB 10.4+, buat DB + user terpisah per situs; **jangan** impor dump lama — gunakan dump **staging** yang sudah bersih.
4. **Akun admin**: buat **akun admin baru per situs** dengan email sekolah; **jangan** migrasikan akun lama (163 akun hasil serangan sudah tidak ada di staging ✓).
5. **Email**: konfirmasi alamat resmi yang akan dipakai (`info@…`) — **alamat pada halaman Kontak belum dikonfirmasi pemilik** ⚠️.
6. **Plugin/theme**: Astra + Elementor (gratis) + WPForms Lite (Cibitung/Tambun) — semua dari repo resmi ✓; **Elementor Pro tidak tersedia** (keputusan tertunda).
7. **Rollback**: checkpoint SQL lokal sudah ada (6G, 6I, 6K-A, 7-01, 7-1-pre, **8D ×2**) + paket WordPress resmi; siapkan juga backup harian hosting sebelum go-live.
8. **Bahasa Sragen** `en_US` → perlu diubah ke `id_ID` **sebelum** produksi (belum diubah — menunggu persetujuan).

**Tidak ada** login/migrasi/perubahan hosting yang dilakukan ✓

## 6. Yang LULUS / BELUM DIUJI (dipisah tegas)
**LULUS (terukur):** HTTP semua halaman publish Kebumen & Cibitung ✓ · aset lokal resolvable (0 hilang) ✓ · trash terverifikasi ✓ · checkpoint terverifikasi ✓ · dependensi #1252 = nol ✓
**BELUM DIUJI:** console JS, warning PHP runtime, responsif 360/390/412/768/1366 **setelah perubahan ini**, overflow, menu mobile interaktif, keyboard/focus, formulir, `tel:`/WhatsApp, **verifikasi visual** ✗ — browser automation tidak stabil di mesin ini; saya **tidak** mengklaim lulus untuk hal-hal ini.

## 7. Bug tersisa
| # | Temuan | Status |
|---|---|---|
| B-1 | Sragen bahasa `en_US` | **TERBUKA** — butuh persetujuan ubah opsi |
| B-2 | `WP_MEMORY_LIMIT` 40M | **PRECHECK** — set 256M di produksi |
| B-3 | Kebijakan privasi masih template | **DRAF SIAP** → `DRAF-KEBIJAKAN-PRIVASI-4-SITUS.md` |
| B-4 | `elementor-1252` publish yatim | **DIPERTAHANKAN** sesuai instruksi |
| B-5 | Item menu `#` / `#akademik` | **TIDAK DIUBAH** sesuai instruksi |
| B-6 | Elementor Pro | **KEPUTUSAN TERTUNDA** |
| B-7 | WXR Kebumen di repo publik | **BELUM PRIVAT** (butuh `gh auth login`) |

## 8. Status kesiapan
| Situs | Status |
|---|---|
| Kebumen | **SIAP dengan catatan** (privasi, memory, bahasa ✓) |
| Cibitung | **SIAP dengan catatan** (privasi, `#1252`, memory) |
| Tambun | **SIAP dengan catatan** (privasi; QA belum diulang fase ini) |
| Sragen | **SIAP dengan catatan** — **bahasa `en_US`** harus diperbaiki dulu |

## 9. Artefak
- Laporan ini: `FASE-8D-TERPADU-HASIL.md`
- Draf privasi: `DRAF-KEBIJAKAN-PRIVASI-4-SITUS.md`
- Checkpoint: `staging/checkpoints/checkpoint-8d-pre-hapus-laman-contoh-20261021-185137.sql`, `…pre-hapus-3-draft-20261021-185137.sql`
- Data uji: `sragen/export/_8d.json`, `_8dqa.json`

## 10. Konfirmasi batas
`goly1264_ams` & `Downloads\admin.php` **tidak disentuh** ✓ · dump/backup/pohon bukti/inventaris/checksum/manifest **tidak diubah** ✓ · tanpa impor dump ke produksi ✓ · payload tidak dijalankan, `cgi-bin.zip` tidak diekstrak ✓ · tanpa akun/kredensial lama ✓ · tanpa push/commit/perubahan remote ✓ · tanpa layanan berbayar ✓ · tanpa secrets di laporan ✓
