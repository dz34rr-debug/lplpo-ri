# LPLPO Depo — Stok Opname Obat

Aplikasi web untuk menyusun **LPLPO** (Laporan Pemakaian dan Lembar Permintaan Obat) bulanan sebuah depo farmasi
(default: Depo Rawat Inap) langsung dari file laporan Excel yang sudah ada.

Rumus per barang, sama seperti sheet `LPLPO RI`:

```
SISA    = Stok awal (SO bulan lalu) + RET + H + IN − OUT − SALE
SELISIH = SISA − WORKSHEET
```

## Fitur

- **Unggah satu workbook** berisi sheet `Worksheet`, `SO …`, `RET`, `H`, `IN`, `O`, `SALE` — setiap sheet dikenali dari namanya.
- **Atau unggah per sumber** (file terpisah untuk retur, hibah, masuk, keluar, penjualan, worksheet). Kolom dikenali dari judulnya
  (`Nama Obat`/`Jumlah Retur`, `NAMA BARANG`/`JUMLAH`, `NAMA PRODUK`/`QTY`, `NAMA BARANG`/`AKHIR`).
- Pencocokan barang berdasarkan nama (seperti VLOOKUP), tanpa membedakan huruf besar/kecil dan spasi berlebih.
- Ringkasan total, saringan "ada selisih"/"ada mutasi", pencarian, dan urutan berdasarkan selisih.
- Daftar **barang yang tidak ada di master** stok awal, dengan opsi memasukkannya ke LPLPO.
- **Unduh Excel** (`LPLPO RI` dengan rumus SISA/SELISIH, `TIDAK ADA MASTER`, dan `SO <bulan>` siap dipakai bulan depan).
- **Simpan periode** ke database; periode tersimpan bisa dipakai sebagai stok awal bulan berikutnya (berdasarkan worksheet atau sisa LPLPO).

## Teknologi

TanStack Start (React 19) · Tailwind CSS 4 + shadcn/ui · SheetJS (pembacaan/penulisan Excel di browser) ·
Netlify Database (Postgres) dengan Drizzle ORM · di-deploy ke Netlify.

## Menjalankan secara lokal

```bash
pnpm install
netlify dev        # menjalankan aplikasi beserta emulasi Netlify Database
```

Perubahan skema database: ubah `db/schema.ts`, lalu `npx drizzle-kit generate --name <nama_perubahan>`.
Migrasi diterapkan otomatis oleh Netlify saat deploy.
