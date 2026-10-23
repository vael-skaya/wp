# FASE — DATABASE: MINIMUM UNTUK GO-LIVE

**22 Oktober 2026** · **READ-ONLY** — tanpa import/DROP/UPDATE/ALTER/migrasi · tanpa akses hosting · tanpa isi pribadi

## 1. Jumlah baris tabel formulir & data permanen (dihitung dari dump)

| Tabel | Tambun | Cibitung | Kebumen | Sragen | Sifat |
|---|---:|---:|---:|---:|---|
| `e_submissions` | 0 | 0 | 0 | 0 | permanen (kiriman form) |
| `e_submissions_values` | 0 | 0 | 0 | 0 | permanen (nilai isian) |
| `e_submissions_actions_log` | 0 | 0 | 0 | 0 | log |
| `e_notes` | 0 | 0 | 0 | 0 | permanen (catatan form) |
| `frmt_form_entry` | 0 | 0 | 0 | 0 | permanen (kiriman) |
| `frmt_form_entry_meta` | 0 | 0 | 0 | 0 | permanen |
| `wpforms_payments` | 0 | 0 | 0 | 0 | transaksi |
| `wpforms_logs` | 0 | 0 | 0 | 0 | log |
| `ahm_emails` | — | — | 0 | — | permanen (email) |
| `ahm_sessions` | — | — | 0 | — | sesi |
| `ahm_download_stats` | — | — | 0 | — | statistik |
| `mailpoet_subscribers` | 1 | 1 | 1 | 1 | permanen (langganan) |
| `mailpoet_newsletters` | 0 | 0 | 0 | 0 | konten email |
| `actionscheduler_actions` | 3 | 4 | 3 | 3 | operasional |
| `actionscheduler_logs` | 5 | 6 | 5 | 5 | log |

`—` = tabel tidak ada di dump · `0` = tabel ada tetapi **tanpa baris**

## 2. Kesimpulan data
- **Tambun:** tabel formulir ada tetapi **5 kosong**; satu-satunya data non-operasional = `mailpoet_subscribers` (1 baris)
- **Cibitung:** tabel formulir ada tetapi **5 kosong**; satu-satunya data non-operasional = `mailpoet_subscribers` (1 baris)
- **Kebumen:** tabel formulir ada tetapi **5 kosong**; satu-satunya data non-operasional = `mailpoet_subscribers` (1 baris)
- **Sragen:** tabel formulir ada tetapi **5 kosong**; satu-satunya data non-operasional = `mailpoet_subscribers` (1 baris)

⇒ **Tidak ada kiriman formulir, pendaftaran, atau data kontak yang perlu diselamatkan** dari produksi ✓
⇒ `actionscheduler_*` berisi 1–4 baris = **antrean sementara**, dapat dibuat ulang ✓

## 3. Rekomendasi per sekolah
### Tambun · Cibitung · Kebumen · Sragen
| Kategori | Isi |
|---|---|
| **GO-LIVE MINIMUM** ✓ terpenuhi | Semua tabel inti WordPress ada di staging. Tidak ada data formulir produksi yang hilang |
| **PRESERVE BEFORE GO-LIVE** | `mailpoet_subscribers` (**1 baris**) — cek apakah langganan nyata; bila ya, salin **hanya tabel ini** (bukan seluruh DB) |
| **DEFER** | CartFlows, Action Scheduler, Forminator, Fluent Forms, GLA, `ahm_*` — kosong/tidak dipakai; dapat dipasang setelah live bila perlu |
| **JANGAN migrasi** | `users`, `usermeta`, `options` produksi — 163 akun palsu dari serangan |

## 4. Blocker nyata untuk go-live (bukan data)
| # | Blocker | Pemilik |
|---|---|---|
| B-1 | Repo `vael-skaya/wp` **masih publik** | Saka (`gh auth login` → privatkan) |
| B-2 | Checksum inti 3/4 **belum terverifikasi** | Saka (izin jalankan ulang) |
| B-3 | Hosting belum diperiksa: PHP ≥8.1, `memory_limit ≥256M`, SSL, DNS | Pak Eko / Saka |
| B-4 | Email resmi + teks kebijakan privasi | Pak Eko |

## 5. Ketidakpastian yang dinyatakan
- Isi `mailpoet_subscribers` (1 baris) **tidak** saya periksa — bisa jadi baris sistem ❓
- Apakah Forminator/CartFlows/Fluent Forms **pernah** menerima kiriman lalu dihapus — **tidak dapat dipastikan** dari dump ❓
- Kolom/tabel yang strukturnya berbeda antar-versi plugin **tidak** dibandingkan rinci ✗
- **Tidak** meminta ekspor ulang ✓

## 6. Konfirmasi
Tanpa import/DROP/UPDATE/ALTER/migrasi ✓ · tanpa perubahan staging/file/hosting ✓ · tanpa isi pribadi/kredensial di laporan ✓ · `goly1264_ams` & `Downloads\admin.php` tidak disentuh ✓ · tanpa commit/push ✓
