# Warehouse Storage Calculator

Menghitung kebutuhan ruang penyimpanan dari data stok: berapa **rack** dan berapa **pallet**, dan
apakah muat di lantai yang tersedia.

Next.js (App Router, TypeScript). Tanpa database, tanpa environment variable — semua perhitungan
berjalan di browser, dan file yang diunggah tidak pernah dikirim ke server.

Cara deploy ada di [DEPLOY.md](./DEPLOY.md).

---

## Alurnya empat langkah

**1 · Layout lantai** — susun denah langsung dengan tangan. Geser block untuk memindahkan, geser
sisinya untuk mengubah ukuran. Tiap block bisa punya area **inbound** dan **outbound** yang dipotong
dari lantai terpakai, plus aisle tambahan yang bisa digeser panjang dan posisinya.

**2 · Konfigurasi site** — rack type (bisa lebih dari satu), pallet type, jarak aisle, dan setelan
untuk perhitungan berbasis volume. Semua bisa diubah, dengan rekomendasi di samping tiap kolom.

**3 · Data SKU** — tiga file, urutan bebas, jenisnya dikenali otomatis dari header:

| File | Isinya | Dipakai untuk |
|---|---|---|
| Data stok / ROP | jumlah per SKU | apa yang harus disimpan |
| MDM replenish | kolom **Pallet Quantity** | kebutuhan pallet = stok ÷ angka ini |
| Master Data SKU | numerator PAL/CAR/PC + dimensi | kebutuhan rack, pcs per carton |

**4 · Aturan berbagi pallet** — seberapa rapat SKU boleh berbagi satu pallet. Ini menggeser hasil
lebih jauh daripada input mana pun:

- **Satu SKU per pallet** — tiap SKU pakai pallet sendiri, sisa satu carton pun makan satu pallet
- **Satu SKU per stack** *(disarankan)* — sisa tiap SKU tidak terpecah, tapi berbagi pallet
- **Aturan 50%** — yang mengisi ≥ setengah pallet pakai pallet sendiri, sisanya berbagi
- **Luas murni** — batas bawah teoretis, membolehkan satu lot terpecah dua pallet

---

## Yang menentukan hasilnya

**Kebutuhan pallet** datang dari kolom **Pallet Quantity** di MDM: berapa pcs muat di satu pallet.
Stok dibagi angka itu — terukur, bukan diperkirakan dari dimensi. SKU yang tidak ada di file itu
jatuh ke perhitungan geometri sebagai cadangan.

**Kebutuhan rack** dari dimensi carton di Master Data SKU: tumpuk ke belakang sedalam rack, ke atas
setinggi shelf, dan satu stack tambahan ke samping (maksimal dua kolom). Carton yang tidak punya
L×W×H dihitung dari volumenya, pada 80% isi dan dibagi 5 SKU type per shelf — keduanya bisa diubah.

**Kapasitas lantai** dari denah: pallet berdiri dua baris saling membelakangi tanpa celah, aisle
hanya di antar block. Tiap block bisa dikunci arah aisle-nya dan arah putar pallet-nya, atau
dibiarkan Optimal — pencarinya mencoba keempat kombinasi plus pergeseran grid untuk menghindari
dock, lalu memilih yang memuat paling banyak.

---

## Struktur

```
app/            halaman dan style global
components/     tampilan React — canvas denah, panel konfigurasi, hasil
lib/            mesin perhitungan, bebas React
  calc.ts         alokasi rack, kebutuhan pallet, empat aturan berbagi
  layout.ts       geometri pallet, aisle, dock, optimasi penempatan
  masterdata.ts   pembaca dua file SAP
  csv.ts          parser CSV (RFC 4180) + deteksi jenis file
  site.ts         konfigurasi site
  layouts.ts      simpan/muat layout bernama
  estimate.ts     estimasi carton untuk SKU tanpa dimensi terukur
  calc.test.ts    263 test
sample-data/    file contoh — TIDAK ikut ter-commit (lihat .gitignore)
```

`lib/` sengaja tidak mengimpor React sama sekali, jadi seluruh rumus bisa dijalankan dan diuji
tanpa browser.

---

## Menjalankan

```bash
npm install
npm run dev      # localhost:3000
npm test         # 263 test
npm run build    # build produksi
```

---

## Catatan penting

**Data master jangan ditaruh di `public/`.** Folder itu disajikan terbuka oleh Next.js — siapa pun
yang tahu URL-nya bisa mengunduh isinya. File contoh disimpan di `sample-data/` yang sudah masuk
`.gitignore`.

**Layout tersimpan di browser masing-masing.** Fitur simpan layout memakai localStorage, jadi tidak
terlihat antar pengguna dan tidak ikut kalau ganti perangkat atau browser.

**Rumus yang pernah direvisi dicatat, bukan ditimpa.** Beberapa keputusan sudah dibalik selama
pengembangan (aturan aisle, kapasitas shelf, sumber angka pallet). Riwayatnya ada di dokumen proyek
supaya perubahan bisa ditelusuri, bukan hilang begitu saja.
