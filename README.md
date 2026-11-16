# SIPP SYSTEM

Repository ini berisi export aplikasi Oracle APEX untuk aplikasi `SIPP SYSTEM` dengan ID `58839` yang tersimpan dalam file `f58839.sql`.

## Ringkasan

`SIPP SYSTEM` adalah aplikasi berbasis Oracle APEX untuk menangani pelaporan/whistleblowing (WBS) atau pengaduan internal/eksternal. Aplikasi ini memungkinkan pengguna untuk:

- mengisi formulir laporan baru,
- melampirkan dokumen pendukung,
- melihat riwayat laporan milik pengguna,
- memantau status laporan di sisi admin,
- mengubah status dan penanggung jawab (PIC) serta meninjau dokumen.

Berdasarkan isi export, aplikasi ini dibuat untuk kebutuhan manajemen laporan kejadian, termasuk data pelapor, terlapor, kronologi kejadian, lokasi, nominal, dan lampiran file.

## Metadata aplikasi

- Nama aplikasi: `SIPP SYSTEM`
- Alias aplikasi: `WBS`
- Application ID: `58839`
- Versi APEX: `26.2.1`
- Pemilik schema: `SIPP_SYSTEM`
- Tanggal ekspor: `2026-04-07` (pada file metadata)
- Jumlah halaman aplikasi: `38`
- Tema UI: Redwood / Universal Theme

## Struktur utama

File utama di repositori:

- `f58839.sql` — export aplikasi Oracle APEX lengkap, mencakup tabel pendukung, halaman aplikasi, LOV, form, dan konfigurasi security.
- `README.md` — dokumentasi ini.

## Objek data utama

Pada script instalasi terdapat tabel utama berikut:

### 1) `XTD_WBS`
Tabel utama untuk data laporan.

Kolom penting:

- `ID` — primary key
- `NAMA_PELAPOR` — nama pelapor
- `TELP_PELAPOR` — nomor telepon pelapor
- `EMAIL_PELAPOR` — email pelapor
- `TINDAKAN` — jenis tindakan/kejadian
- `NAMA_TERLAPOR` — nama terlapor
- `JABATAN_TERLAPOR` — jabatan terlapor
- `WAKTU_KEJADIAN` — waktu kejadian
- `LOKASI_KEJADIAN` — lokasi kejadian
- `KRONOLOGIS_KEJADIAN` — kronologi kejadian
- `NOMINAL` — nominal/estimasi kerugian
- `FILE_BLOB` — blob file lampiran utama
- `STATUS` — status laporan
- `NOMOR_LAPORAN` — nomor laporan unik
- `MIME_TYPE` — tipe MIME file
- `LAST_UPDATE_DATE` — tanggal pembaruan terakhir
- `FILE_NAME` — nama file
- `PIC` — penanggung jawab utama

### 2) `XTD_WBS_BLOB`
Tabel pendukung untuk menyimpan lampiran file secara terpisah.

Kolom penting:

- `ID` — primary key
- `FILE_NAME` — nama file
- `FILE_MIME_TYPE` — tipe MIME file
- `FILE_BLOB` — data file binary
- `FILE_BLOB_DATED` — tanggal upload
- `WBS_ID` — foreign key ke `XTD_WBS.ID`

### 3) `XTD_WBS_NO`
Sequence untuk menghasilkan nomor laporan otomatis.

## Status laporan yang umum digunakan

Berdasarkan pilihan LOV di halaman admin, status yang digunakan antara lain:

- `SUBMITTED`
- `ACCEPTED`
- `REJECTED`
- `ON PROGRESS`
- `COMPLETED`

## Fitur aplikasi

Berdasarkan definisi halaman pada export, aplikasi ini memiliki beberapa fitur utama:

- Home page (`page 1`)
- Daftar laporan WBS (`page 2`)
- Formulir pengisian laporan (`page 3`)
- Dashboard laporan per pengguna (`page 4`)
- Viewer lampiran file (`page 5`)
- Admin page status (`page 6`)
- Form update status admin (`page 7`)
- Integrasi upload/preview file
- Proses update data dan penghapusan data pada form admin
- Akses publik untuk formulir input laporan, dengan kontrol admin untuk review dan tindakan lanjutan

## Arsitektur dan pola kerja

Aplikasi ini bukan script SQL biasa; ini adalah export aplikasi Oracle APEX yang berisi:

- definisi aplikasi,
- halaman/region/pages,
- item, LOV, proses, dynamic action,
- konfigurasi autentikasi/authorization,
- script instalasi tabel pendukung,
- trigger/sequence/index yang dibutuhkan.

Dengan demikian, untuk menjalankannya, biasanya cukup diimpor ke environment Oracle APEX yang sesuai, lalu menjalankan aplikasi di workspace yang dimaksud.

## Fitur offline mode (PWA)

Script export juga mengaktifkan fitur Progressive Web App (PWA) untuk aplikasi ini. Hal ini terlihat dari konfigurasi aplikasi:

- `p_pwa_is_installable => 'Y'`
- `p_pwa_manifest_display => 'standalone'`
- `p_pwa_service_worker_hooks` mengarah ke `sipp_service_worker.js`
- halaman global memuat file `sipp_pwa_offline.js` saat aplikasi dimuat

### Cara kerja offline mode

Kode JavaScript yang di-embed pada aplikasi menciptakan controller `window.SIPP_PWA` yang bekerja seperti berikut:

- membuka/menjalankan IndexedDB dengan nama `SIPP_PWA_DB`
- membuat object store `wbs_queue` untuk menampung data laporan yang masih tertunda
- mengumpulkan data formulir dari halaman laporan (`P3_*`)
- saat koneksi internet terputus, submit laporan tidak langsung gagal, tetapi disimpan ke queue lokal
- setelah koneksi pulih, aplikasi secara otomatis memproses queued item dan mengirimnya ke server
- menampilkan banner notifikasi bahwa laporan sudah aman disimpan dan akan dikirim otomatis
- membersihkan form setelah data berhasil ditahan agar user bisa melanjutkan input berikutnya

### Tujuan fitur ini

Fitur ini dibuat agar formulir pelaporan tetap bisa digunakan meskipun pengguna sedang offline atau koneksi terputus sementara. Data yang sudah dimasukkan tidak hilang, dan akan diteruskan ke backend saat jaringan kembali normal. Ini sangat penting untuk skenario pengaduan/kasus darurat yang perlu dicatat tanpa tergantung koneksi internet stabil.

### Objek yang terkait

- `sipp_pwa_offline.js` — controller offline logic di browser
- `sipp_service_worker.js` — worker service untuk PWA
- `wbs_queue` — object store di IndexedDB untuk antrian laporan offline

## Cara import

1. Pastikan environment database Oracle sudah siap dan Oracle APEX terinstal.
2. Gunakan SQL Workshop / APEX Application Builder untuk mengimpor `f58839.sql`.
3. Jalankan sebagai schema pemilik aplikasi (`SIPP_SYSTEM`) atau user dengan hak yang cukup.
4. Setelah import selesai, buka aplikasi `SIPP SYSTEM` dari APEX.

Catatan: file export adalah aplikasi APEX, bukan dump database mandiri. Untuk menjalankan fungsionalitas penuh, environment APEX dan database target harus sudah disiapkan dengan benar.

## Catatan penting

- Script instalasi membuat tabel `XTD_WBS`, `XTD_WBS_BLOB`, dan sequence `XTD_WBS_NO`.
- Lampiran file disimpan sebagai BLOB untuk kebutuhan preview dan download.
- Aplikasi mendukung status kerja admin dan pemantauan laporan secara terstruktur.

## Kesimpulan

`SIPP SYSTEM` adalah aplikasi manajemen laporan berbasis web yang dirancang untuk menerima, meninjau, dan mengelola laporan kejadian secara terstruktur dengan dukungan lampiran file dan alur kerja admin. Export file `f58839.sql` berisi seluruh konfigurasi aplikasi beserta tabel serta artefak pendukungnya.
