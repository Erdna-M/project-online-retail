# Data Model — Online Retail Batch Pipeline

> Dokumen desain data warehouse untuk final project.

## 1. Problem Statement
Data transaksi ritel mentah (Online Retail 541.909 baris) tidak bisa langsung di analisa karena banyak masalah kualitas :
24,6% transaksi tanpa CustomerID, ~2,4% transaksi retur yang bercampur dengan penjualan, ~0,5% harga tidak valid, dan 4.879 baris duplikat.
Pipeline ini membersihkan data tersebut lalu memodelkannya menjadi data warehouse bentuk star schema, siap dipakai menjawab pertanyaan bisnis seperti tren penjualan, produk terlaris, dan penjualan per negara.

## 2. Dataset Overview
- Sumber: Online Retail dataset (CSV) — aslinya dari UCI Machine Learning Repository
- Volume: ~541.909 baris transaksi
- Kolom (8): InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country

## 3. Data Quality Findings & Cleaning Rules
CustomerID kosong: Transaksi tetap sah walau tanpa identitas pelanggan, membuang 24,6% data = kehilangan hampir seperempat penjualan valid. Karena itu tetap disimpan, dan di dim_customer nilai kosong dipetakan ke satu kategori "Unknown".
UnitPrice ≤ 0: Harga nol/negatif tidak valid dan tidak bisa dihitung sebagai revenue, sehingga baris ini dibuang.
Duplikat penuh: Baris yang identik 100% akan terhitung ganda dan menggelembungkan total penjualan (over-count), sehingga dibuang.

| # | Temuan | Jumlah | Keputusan | Alasan |
|---|--------|--------|-----------|--------|
| 1 | Quantity negatif (retur) | 13.269 (~2,4%) | Keep + tandai is_return | Retur = transaksi bisnis yang sah (barang dikembalikan), bukan data rusak; disimpan & ditandai agar mudah dipisah saat analisis. (contoh) |
| 2 | CustomerID kosong | 133.391 (~24,6%) | Keep + petakan -> "Unknown" | [TULIS SENDIRI] |
| 3 | UnitPrice <= 0 | 2.534 (~0,5%) | Drop | [TULIS SENDIRI] |
| 4 | Duplikat penuh | 4.879 | Drop | [TULIS SENDIRI] |

## 4. Star Schema Design
Detail deskriptif (nama produk, negara pelanggan) disimpan satu kali di dimension table, sedangkan fact_sales hanya menyimpan angka terukur + foreign key ringkas ke dimension. Ini menghemat storage (detail tidak diulang di ratusan ribu baris) dan menjaga konsistensi: kalau satu detail perlu diperbaiki, cukup diubah di satu baris dimension, bukan di ribuan baris fact.

### fact_sales
- InvoiceNo
- product_key   (FK -> dim_product)
- customer_key  (FK -> dim_customer)
- date_key      (FK -> dim_date)
- Quantity, UnitPrice
- total_amount  (= Quantity x UnitPrice)
- is_return

### dim_product
- product_key (PK)
- StockCode, Description

### dim_customer
- customer_key (PK)
- CustomerID (null -> "Unknown"), Country

### dim_date
- date_key (PK, mis. 20101201)
- full_date, year, month, day

## 5. Kenapa dim_date terpisah?
Dengan memecah tanggal menjadi year/month/day di dim_date, analisis berbasis waktu jadi cepat tanpa mengolah ulang kolom tanggal setiap saat. Contoh: pertanyaan "berapa total penjualan per bulan di 2011?" cukup dijawab dengan GROUP BY month, bukan mem-parse InvoiceDate berulang-ulang.

## 6. Catatan
- Arsitektur platform (Airflow + Spark + Postgres) didokumentasikan terpisah.
- Cleaning rules di atas menjadi acuan tahap Transform (PySpark).