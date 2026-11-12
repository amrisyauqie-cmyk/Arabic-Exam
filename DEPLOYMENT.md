# Catatan perbaikan deployment Arabic CBT SMP

## Perbaikan log Vercel

Log yang dikirim menunjukkan Vercel menggunakan Node.js `26.13.1`, lalu gagal mengompilasi native addon `better-sqlite3` `11.10.0` terhadap API V8 Node 26.

Perbaikan build:
- `package.json` sekarang membatasi Node.js ke `22.x`.
- `better-sqlite3` dipatok ke `11.10.0`.
- `multer` diperbarui ke jalur 2.x untuk menghindari peringatan kerentanan versi 1.x.

Jika tetap mencoba build di Vercel:
1. Buka **Project Settings → General → Node.js Version**.
2. Pilih **22.x**.
3. Pastikan `SESSION_SECRET` diatur sebagai environment variable.
4. Jalankan redeploy dengan **Clear Build Cache**.

**Penting:** langkah di atas menargetkan kegagalan `npm install` yang terlihat di log. Itu belum menjadikan SQLite lokal cocok untuk Vercel. Aplikasi ini menyimpan database ke file SQLite. Lingkungan fungsi serverless Vercel tidak menyediakan penyimpanan file lokal permanen yang dapat diandalkan untuk data ujian. Data dapat hilang atau berbeda antar-instance. Jangan gunakan konfigurasi SQLite lokal ini untuk ujian sekolah production di Vercel.

Untuk tetap memakai Vercel, database perlu dimigrasikan ke layanan database terkelola seperti PostgreSQL (misalnya Neon/Supabase) atau layanan SQLite remote seperti Turso, dan backend perlu disesuaikan.

## Cara deployment yang mempertahankan SQLite

Proyek ini juga menyediakan `render.yaml` untuk menjalankan aplikasi sebagai layanan Node.js dengan persistent disk. Blueprint tersebut menggunakan paket **Starter** karena persistent disk memerlukan layanan berbayar di Render.

1. Push folder proyek ke repository GitHub.
2. Di Render, pilih **New → Blueprint** lalu pilih repository.
3. Tinjau konfigurasi dan biaya layanan/persistent disk sebelum membuat layanan.
4. Render akan memasang `SESSION_SECRET` otomatis.
5. Database disimpan di `/opt/render/project/src/data/arabic-cbt.sqlite`.

Untuk menjalankan lokal:
```bash
npm ci
npm start
```

Buka `http://localhost:3000`.

## Pemeriksaan keamanan sebelum ujian sungguhan

- Ganti kata sandi admin awal.
- Pastikan `SESSION_SECRET` unik dan rahasia.
- Aktifkan HTTPS.
- Siapkan backup database berkala dan uji pemulihannya.
- Uji banyak siswa mengerjakan ujian bersamaan.
- Uji autosave, timer, dan pemulihan sesi.
- Tambahkan penyimpanan sesi persisten dan perlindungan CSRF/rate limiting sebelum pemakaian production.
