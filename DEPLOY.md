# Deploy ke Vercel

Proyek ini Next.js App Router biasa: tanpa database, tanpa environment variable, tanpa konfigurasi
build khusus. Semua perhitungan berjalan di browser. Jadi deploy-nya termasuk yang paling sederhana.

**Perkiraan waktu: 10–15 menit.**

---

## Sebelum mulai — satu hal soal data

Folder `public/` di Next.js **bisa diakses siapa pun lewat URL**. Kalau file CSV master ditaruh di
sana, siapa pun yang tahu alamatnya bisa mengunduhnya.

Karena itu ketiga file contoh sudah dipindah ke `sample-data/`, yang **tidak** disajikan ke publik,
dan folder itu sudah masuk `.gitignore` supaya tidak ikut ter-commit:

```
sample-data/
  bogor_real_input.csv     data stok / ROP
  mdm_replenish.csv        MDM replenish — Pallet Quantity
  master_data_sku.csv      Master Data SKU — konversi UoM + dimensi
```

File-file itu tetap ada di komputer Anda untuk uji coba, tapi tidak akan ikut ke GitHub maupun ke
Vercel. Pengguna aplikasi mengunggah filenya sendiri lewat halaman — tidak ada data yang tersimpan
di server.

> Kalau Anda memang ingin file contoh ikut ter-deploy, pindahkan ke `public/` dan hapus baris
> `/sample-data/` dari `.gitignore`. Pastikan dulu isinya boleh dilihat publik.

---

## Yang dibutuhkan

- Akun **GitHub** (gratis)
- Akun **Vercel** (gratis, bisa daftar pakai GitHub)
- **Node.js 18.18 atau lebih baru** kalau ingin menjalankan lokal dulu — cek dengan `node -v`
- **Git** terpasang di komputer

---

## Cara A — GitHub + Vercel (disarankan)

Pilih ini kalau ingin setiap perubahan otomatis ter-deploy ulang.

### 1. Coba jalankan lokal dulu

```bash
cd bogor-storage-calculator
npm install
npm run dev
```

Buka `http://localhost:3000`. Unggah ketiga file dari `sample-data/`, klik **Jalankan
perhitungan**, pastikan hasilnya keluar. Kalau di sini jalan, di Vercel juga akan jalan.

Hentikan dengan `Ctrl+C`.

### 2. Naikkan ke GitHub

```bash
git init
git add .
git commit -m "Warehouse calculator: layout, rack, dan pallet"
```

Buat repository kosong di [github.com/new](https://github.com/new) — **jangan** centang "Add a
README" atau "Add .gitignore", proyek ini sudah punya sendiri. Lalu:

```bash
git remote add origin https://github.com/<username-anda>/bogor-storage-calculator.git
git branch -M main
git push -u origin main
```

Sebelum lanjut, buka repository-nya di browser dan **pastikan folder `sample-data/` tidak ada di
sana**. Kalau ternyata ikut ter-upload, berarti `.gitignore` belum terbaca — jalankan:

```bash
git rm -r --cached sample-data
git commit -m "Keluarkan data master dari repository"
git push
```

### 3. Import ke Vercel

1. Buka [vercel.com/new](https://vercel.com/new), masuk pakai akun GitHub.
2. Pilih repository `bogor-storage-calculator`.
3. Vercel akan mendeteksi sendiri: Framework **Next.js**, build command `next build`, output
   `.next`. **Biarkan semua apa adanya** — tidak ada yang perlu diubah.
4. Environment Variables: **kosongkan**. Aplikasi ini tidak memakai satu pun.
5. Klik **Deploy**.

Build memakan waktu sekitar 1–2 menit. Setelah selesai Anda dapat URL seperti
`https://bogor-storage-calculator.vercel.app`.

### 4. Setiap update berikutnya

```bash
git add .
git commit -m "penjelasan singkat perubahannya"
git push
```

Vercel otomatis build ulang dan menerbitkan versi baru. Tidak perlu buka dashboard.

---

## Cara B — Vercel CLI (tanpa GitHub)

Cocok kalau hanya ingin cepat memasang tanpa repository.

```bash
npm install -g vercel
cd bogor-storage-calculator
vercel
```

CLI akan bertanya beberapa hal — jawab default semua (tekan Enter). Untuk menerbitkan ke alamat
produksi:

```bash
vercel --prod
```

Kekurangannya: tidak ada deploy otomatis. Setiap ada perubahan, `vercel --prod` harus dijalankan
lagi secara manual.

---

## Setelah live

**Ganti nama domain.** Di dashboard Vercel: Settings → Domains. Anda bisa memakai subdomain
`.vercel.app` yang lain, atau memasang domain perusahaan sendiri.

**Batasi akses kalau perlu.** Halaman ini bersifat publik. Kalau tidak boleh diakses umum, Vercel
punya fitur Password Protection dan Vercel Authentication (Settings → Deployment Protection) —
sebagian butuh paket berbayar.

**Data pengguna tidak ke mana-mana.** Semua perhitungan berjalan di browser; file CSV yang diunggah
tidak pernah dikirim ke server. Layout yang disimpan tersimpan di browser masing-masing pengguna
(localStorage), jadi tidak saling terlihat antar orang dan tidak ikut kalau ganti perangkat.

---

## Kalau build gagal

| Gejala | Sebabnya | Perbaikannya |
|---|---|---|
| `Module not found` | ada file yang belum ter-commit | `git status`, lalu `git add` file yang tertinggal |
| Error TypeScript | tipe tidak cocok | jalankan `npx tsc --noEmit` lokal, perbaiki, commit ulang |
| `next: command not found` | dependency belum terpasang | pastikan `package.json` dan `package-lock.json` ikut ter-commit |
| Build sukses tapi halaman kosong | error JavaScript di browser | buka Console di DevTools, lihat pesannya |
| Peringatan soal Google Fonts saat build | jaringan build tidak bisa mengambil font | **abaikan** — hanya peringatan, halaman tetap jalan dan font fallback dipakai |

Cek juga dengan menjalankan build yang sama persis seperti Vercel di komputer sendiri:

```bash
rm -rf .next node_modules
npm ci
npm run build
npm start
```

Kalau perintah itu berhasil, Vercel hampir pasti juga berhasil.

---

## Perintah yang tersedia

| Perintah | Fungsi |
|---|---|
| `npm run dev` | jalankan lokal dengan auto-reload di `localhost:3000` |
| `npm run build` | build produksi, sama seperti yang dipakai Vercel |
| `npm start` | jalankan hasil build produksi secara lokal |
| `npm test` | 263 unit test untuk mesin perhitungan |
| `npx tsc --noEmit` | cek tipe TypeScript tanpa build |

Biasakan menjalankan `npm test` sebelum `git push` — semua rumus inti ada tesnya, jadi kalau ada
yang rusak akan ketahuan sebelum ter-deploy.

---

## Isi proyek

```
app/            halaman dan style global
components/     tampilan React — canvas denah, panel konfigurasi, hasil
lib/            mesin perhitungan (tanpa React, bisa diuji sendiri)
  calc.ts       alokasi rack, kebutuhan pallet, aturan berbagi
  layout.ts     geometri penempatan pallet, aisle, dock
  masterdata.ts pembaca file SAP (MDM + Master Data SKU)
  csv.ts        pembaca CSV dan deteksi jenis file
  site.ts       konfigurasi site: rack type, pallet type, aisle, volume
  layouts.ts    simpan/muat layout bernama
  calc.test.ts  263 test
sample-data/    file contoh, TIDAK ikut ter-commit maupun ter-deploy
```

Folder `lib/` sengaja bebas dari React: seluruh rumus bisa dijalankan dan diuji tanpa browser.
