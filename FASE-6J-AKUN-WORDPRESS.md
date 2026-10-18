# FASE 6J — AKUN WORDPRESS KEEMPAT SITUS SMK 10 NOVEMBER (RINGKASAN NON-SENSITIF)

**Tanggal:** 18 Oktober 2026 · **Metode:** parsing **offline** dump sumber (tanpa impor ke database, tanpa login) · **Perubahan: 0**
**Catatan penyimpanan:** laporan ini **tidak memuat** username, email, atau hash. Daftar akun rinci disimpan **lokal** di `recovery-work\sensitive\6j-akun\` (folder ini **bukan** repositori Git) dan **tidak boleh** diunggah ke GitHub.

---

## 1. Ringkasan angka
| Situs | Dump / prefix | Total akun | Administrator | Akun dibuat ≥2025-11-01 | Kandidat akun asli (2025-10-30) |
|---|---|---:|---:|---:|---:|
| Tambun | `smkq8953_utama1.sql` / `smk_` | **41** | **41 (100%)** | 40 | **1** |
| Cibitung | `smkq8953_cbitung.sql` / `cbt_` | **90** | **90 (100%)** | 89 | **1** |
| Kebumen | `smkq8953_kebumen.sql` / `kbm_` | **2** | **2 (100%)** | 1 | **1** |
| Sragen | `smkq8953_sragen10.sql` / `srg_` | **30** | **30 (100%)** | 29 | **1** |
| **Total** | — | **163** | **163** | **159** | **4** |

**Role diperiksa dari metadata `{prefix}capabilities` per user ID** (bukan dari nama pengguna): **seluruh 163 akun = `administrator`** dan **`user_level` = 10**. Tidak ditemukan satu pun akun editor, penulis, kontributor, atau subscriber.

## 2. Temuan utama
1. **163 akun administrator untuk empat sekolah** — tidak wajar. Empat situs ini semestinya memiliki 1–3 admin masing-masing.
2. **Hanya 4 akun berusia wajar**: satu per situs, dibuat **2025-10-30** (tanggal pembuatan situs), memakai email **domain sekolah resmi**.
3. **159 akun dibuat antara 2026-07-25 dan 2026-09-11** — seluruhnya administrator. Puncaknya **2026-07-26 (47 akun dalam satu hari)**, bertepatan dengan jendela aktivitas `wp2shell`.
4. Pola pembuatan akun konsisten dengan **otomatisasi penyerang**: login acak/berpola (`wp2_*`, `w2s_*`, `wp_service_*`, `Nx_*`, `Bunk_*`, `admins<hex>`), nama tampilan meniru sistem (impersonasi "WordPress support/admin backup/root"), dan email pada domain **fiktif** (`.invalid`, `*.localhost`, `local.host`, `service.localhost`).
5. **Tiga akun `wp2shell_*`** (satu di Tambun, Cibitung, Sragen) — persis nama yang tercatat sebagai indikator kompromi pada laporan fase sebelumnya.
6. Ditemukan juga domain email penyerang lain di luar daftar C2 lama: **`shellcode.lol`** (2 akun, Cibitung) dan pola email `dumplink*`/`malkomal*` — indikasi aktivitas pengambilan data.
7. **Semua 163 hash password berformat `$wp$2y$10$`** (bcrypt gaya WordPress 6.8+) — termasuk akun asli 2025-10-30. Artinya hash telah **ditulis ulang** pada suatu titik setelah Oktober 2025 (fakta); siapa yang menulis ulang **tidak dapat dibuktikan** dari dump.
8. **`user_activation_key` kosong** dan **`session_tokens` tidak ada** untuk semua akun ⇒ tidak ada aktivasi tertunda atau sesi aktif yang tersimpan di dump.
9. Dua akun **tanpa email sama sekali** (Tambun, Sragen) — keduanya berpola nama "support agent".
10. Tidak ada akun dengan `user_status` ≠ 0 (tidak ada akun spam yang dinonaktifkan).

## 3. Sebaran domain email (hitungan **eksak**, tanpa alamat lengkap)
| Kelompok domain | Jumlah |
|---|---:|
| domain **fiktif** `*.invalid` + `shellcode.lol` | **111** |
| `wp2shell.local` / `*.localhost` / `local.host` | **18** |
| `wp2shell.research` | **3** |
| `example.com` | **3** |
| `wordpress.org` / `wordpress.com` (impersonasi) | **6** |
| gmail.com / outlook.com (generik, semuanya terdaftar pada masa serangan) | **10** |
| `goole.com` (domain salah ketik, muncul di **keempat** situs) | **4** |
| domain sekolah resmi | **6** (4 akun asli + 2 akun login-acak Agustus 2026) |
| **tanpa email sama sekali** | **2** |
| **Total** | **163** |

⇒ **135 akun** memakai domain yang jelas fiktif; **hanya 6** memakai domain sekolah.

## 4. Status password (wajib dibaca)
- **Password asli: `tidak dapat diketahui dari dump`** — yang tersedia hanya hash satu arah.
- Tidak ada upaya membalik hash, menebak, atau memakai kredensial lama.
- **Tidak ada hash yang ditulis ke laporan ini, berkas audit, atau JSON/TSV lokal** (hanya formatnya).
- ⚠️ **Pengungkapan diri:** pada satu langkah *debug* di terminal (bukan laporan), potongan beberapa hash sempat tercetak — satu di antaranya utuh. Ini pelanggaran prosedur saya sendiri. Dampaknya terbatas (akun-akun itu milik penyerang), tetapi hash akun asli **harus dianggap perlu dirotasi**; rotasi semua password admin produksi memang sudah masuk daftar tindak lanjut keamanan.
- **Tidak ada** reset atau perubahan password yang dilakukan.

## 5. Rekomendasi
| Item | Rekomendasi |
|---|---|
| Migrasi ke staging | **Jangan migrasikan satu pun akun lama.** Buat **1 admin baru per staging** (kebijakan "tanpa akun lama" sudah dipakai di Fase 5E1). |
| Akun asli (4) | **Pertahankan idenya, bukan hash-nya**: minta pengelola sekolah mengonfirmasi apakah `admin@<domain sekolah>` masih dipakai; bila perlu, buat ulang akunnya di produksi dengan password baru, bukan memulihkan hash lama. |
| 159 akun lain | **Hapus dari produksi** setelah inventaris disetujui; sebelum penghapusan, ekspor bukti (sudah tersimpan lokal) untuk keperluan forensik. |
| Produksi | Rotasi password seluruh administrator + cabut sesi; tindak lanjut 27 opsi sensitif (laporan Fase 5D/5E) tetap berlaku. |
| Guru/staf | Kumpulkan daftar resmi staf per sekolah, lalu buat akun dengan role **editor/author** sesuai kebutuhan — bukan administrator. |

## 6. Daftar akun yang membutuhkan konfirmasi pengelola sekolah
- **4 akun asli** (satu per situs, dibuat 2025-10-30, email domain sekolah) — perlu konfirmasi siapa pemiliknya dan apakah masih aktif.
- **6 akun** yang memakai email domain sekolah tetapi login acak dan dibuat Agustus 2026 (2 di Cibitung) — **tidak** dapat dianggap akun sah; perlu keputusan pengelola.
- **Seluruh 159 akun lain** — diusulkan **tidak dipertahankan** (bukti rinci tersedia lokal).

## 7. Yang akan dilakukan pada staging (**setelah disetujui**, belum dilakukan)
Untuk setiap situs: catat situs, URL lokal, username baru, role, dan status verifikasi login. **Password ditanyakan lebih dulu kepada Anda**: (a) Anda menentukan sendiri, atau (b) dibuatkan password acak kuat. Password **tidak** akan ditulis di laporan/Markdown/Git/log; bila belum ada penyimpanan rahasia yang aman (password manager), saya **berhenti dan minta arahan**.

## 8. Kepatuhan
- **Tidak ada impor dump**, tidak ada koneksi/login ke WordPress, tidak ada perubahan database/akun.
- **`goly1264_ams` tidak diakses** sama sekali ✓ · **`Downloads\admin.php` tidak disentuh/dibaca/dipindai** ✓.
- Produksi tidak disentuh; tidak ada reset password; tidak ada akun dibuat.
- Dump sumber hanya **dibaca**; file rinci lokal berada di luar repositori Git.

## 9. Keterbatasan
- Klasifikasi "penyerang" berbasis **pola** (domain email fiktif, login berpola, tanggal), bukan bukti atribusi; 159 akun "tidak dapat dibuktikan sebagai staf sekolah" **bukan** berarti "terbukti penyerang" — diperlukan konfirmasi pengelola.
- Hanya tabel `users` + `usermeta` yang diperiksa; jejak lain (log akses, tabel plugin, file) tidak dianalisis pada fase ini.
- Username/email rinci **tidak** disertakan di sini sesuai instruksi; lihat berkas lokal.
