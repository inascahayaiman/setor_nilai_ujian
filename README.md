# Aplikasi Setor Nilai Ujian Essay

Aplikasi web untuk mengumpulkan nilai ujian essay dari guru mapel ke panitia.

- **Database**: Google Spreadsheet
- **Backend/API**: Google Apps Script (`Kode.gs`)
- **Frontend**: satu file statis (`index.html`), di-host di Vercel lewat GitHub

```
Guru / Panitia (browser)  ──►  index.html (Vercel)  ──POST JSON──►  Apps Script Web App  ──►  Spreadsheet
```

## Alamat halaman

| Halaman | Alamat |
|---|---|
| Guru mapel | `https://nama-proyek.vercel.app/` |
| Admin panitia | `https://nama-proyek.vercel.app/#/admin` |

## Langkah pemasangan

### 1. Spreadsheet + Apps Script
1. Buat Google Spreadsheet baru (nama bebas, mis. "Setor Nilai PAS Ganjil").
2. Menu **Ekstensi → Apps Script**. Hapus isi default, tempel seluruh isi `Kode.gs`, simpan.
3. Kembali ke Spreadsheet, muat ulang halaman. Muncul menu **Setor Nilai**.
4. Klik **Setor Nilai → 1. Setup awal (buat sheet)**, izinkan akses saat diminta.
   Akun admin awal: `admin` / `admin123`. Segera ganti lewat **Ubah akun admin panitia**.
5. (Opsional) **Buat data contoh** untuk uji coba. Kosongkan lagi sebelum memasukkan data asli.

### 2. Isi data master (di Spreadsheet)

| Sheet | Kolom | Keterangan |
|---|---|---|
| `Guru` | id, nama, no_wa, program | `no_wa` format `0812…` atau `62812…`. `program` isi `Umum` atau jurusan (Otomotif, Perikanan, …). Kolom `selesai` dan `waktu_selesai` diisi otomatis. |
| `Mapel` | id, nama, program | Mapel umum: program `Umum`. Mapel produktif: isi nama jurusan. |
| `Siswa` | nis, nama, kelas | Nama kelas harus **persis sama** dengan di sheet Penugasan. |
| `Penugasan` | id, guru_id, mapel_id, kelas | **Satu baris = satu guru mengajar satu mapel di satu kelas.** Kolom `status` dan `waktu_setor` diisi otomatis. Kosongkan `id`, lalu jalankan **Setor Nilai → 2. Isi ID Penugasan otomatis**. |
| `Nilai` | (otomatis) | Jangan diedit manual. |
| `Config` | key, value | nama_sekolah, nama_ujian, tahun_ajaran, admin_email (opsional). |

Jangan mengubah nama sheet maupun judul kolom (baris 1). Urutan kolom boleh berubah.

### 3. Deploy Apps Script sebagai Web App
1. Di editor Apps Script: **Deploy → New deployment → Select type: Web app**.
2. *Execute as*: **Me**. *Who has access*: **Anyone**.
3. **Deploy**, izinkan akses, lalu salin **Web app URL** (berakhiran `/exec`).
4. Setiap kali `Kode.gs` diubah: **Deploy → Manage deployments → ✏️ → Version: New version → Deploy** (URL tetap sama).

### 4. Hubungkan frontend
Buka `index.html`, cari bagian:

```js
const CONFIG = {
  API_URL: 'PASTE_URL_WEB_APP_GAS_DI_SINI'
};
```
Ganti dengan URL Web App dari langkah 3.

### 5. GitHub + Vercel
1. Buat repository GitHub, unggah `index.html` (dan `README.md`). `Kode.gs` boleh ikut sebagai arsip.
2. Di [vercel.com](https://vercel.com): **Add New → Project → Import** repository tadi.
3. *Framework Preset*: **Other**. Biarkan Build Command dan Output Directory kosong, lalu **Deploy**.
4. Selesai. Bagikan link utama ke guru, dan simpan link `/#/admin` untuk panitia.

## Alur penggunaan

**Guru**
1. Buka link → pilih nama → **Masuk**.
2. Muncul daftar penugasan (mapel × kelas) milik guru tersebut, dikelompokkan per mapel.
3. **Isi nilai** → ketik manual atau **Unduh template xlsx → isi → Impor**. Siswa tidak hadir ditandai **TH**.
4. **Simpan draft** kapan saja. Bila sudah lengkap, **Periksa & setor** → ringkasan (rata-rata, tertinggi, terendah, peringatan) → **Ya, setor nilai**.
5. Ulangi untuk penugasan lain. Setelah semua disetor, tombol **Selesai** aktif dan panitia mendapat notifikasi.
6. Salah input setelah setor? Buka penugasan → **Revisi nilai** → setor ulang.

**Panitia**
- **Dashboard**: jumlah guru/mapel/kelas/siswa, progres keseluruhan, aktivitas setoran terbaru, link guru (salin).
- **Progres guru**: progres tiap guru, detail per mapel-kelas, tombol **Ingatkan** (membuka WhatsApp dengan pesan siap kirim berisi daftar yang belum disetor).
- **Rekap kelas**: semua mapel dalam satu kelas beserta guru, rata-rata, standar deviasi, nilai tertinggi/terendah, unduh Excel.
- **Data**: pintasan ke Spreadsheet dan daftar penugasan.

Dashboard otomatis menyegarkan diri tiap 60 detik. Jika `admin_email` di sheet `Config` diisi, panitia juga menerima email setiap guru menekan **Selesai**.

## Catatan teknis
- NIS dan nomor WA disimpan sebagai teks agar angka 0 di depan tidak hilang. Saat impor, NIS dicocokkan dengan mengabaikan angka 0 di depan.
- Standar deviasi memakai rumus populasi. Siswa TH tidak masuk perhitungan statistik.
- Penyimpanan memakai kunci (lock) agar banyak guru yang menyetor bersamaan tidak saling menimpa.
- Keamanan sengaja sederhana: halaman admin dilindungi username/password (disimpan sebagai hash SHA-256 di `Config`, sesi 6 jam). Halaman guru tanpa login, sesuai kebutuhan.
- Kuota gratis Apps Script cukup untuk satu sekolah. Jika banyak guru menyetor di menit yang sama dan terasa lambat, minta mereka menunggu beberapa detik setelah menekan tombol.
- Persiapan ujian berikutnya: **Setor Nilai → Kosongkan semua nilai & status**, lalu perbarui data Penugasan.
