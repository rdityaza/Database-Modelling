# Ride-Hailing Database Modeling & Complex Querying

![MySQL](https://img.shields.io/badge/MySQL-8.0.39-4479A1?logo=mysql&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-blue)
![Tables](https://img.shields.io/badge/Tables-10-lightgrey)
![Records](https://img.shields.io/badge/Orders-150-brightgreen)

Proyek database relasional end-to-end yang merancang dan mengimplementasikan skema untuk platform ride-hailing multi-layanan. Pipeline mencakup perancangan ERD (BCNF), implementasi schema di MySQL, pengisian data, hingga pembuatan analytical views menggunakan advanced SQL — dirancang untuk mensimulasikan arsitektur database nyata pada platform Grab.

---

## 📋 Table of Contents
- [Project Structure](#-project-structure)
- [Database Schema Overview](#-database-schema-overview)
- [Tech Stack](#-tech-stack)
- [Key Features](#-key-features)
- [Views & Analytical Queries](#-views--analytical-queries)
- [Quick Start](#-quick-start)
- [Author](#-author)

---

## 📁 Project Structure

```text
ride-hailing-database/
│
├── IF2040_Milestone1_K2_G12.pdf        # ERD & requirements documentation
├── IF2040_Milestone2_K02_G12.sql       # Schema + data (Milestone 2)
├── IF2040_Milestone2_K2_G12.pdf        # Milestone 2 report
├── IF2040_Milestone3_K02_G12.sql       # Final schema + views (Milestone 3)
├── IF2040_Milestone3_K02_G12.pdf       # Milestone 3 report
└── README.md
```

---

## 🗄️ Database Schema Overview

Database ini mengelola tiga layanan utama dalam satu platform terintegrasi: **GrabBike**, **GrabCar**, dan **GrabExpress**. Setiap layanan memiliki tabel spesifik yang terhubung ke tabel `orders` sebagai entitas pusat.

**Total: 10 Tabel + 3 Analytical Views**

| Tabel | Deskripsi |
|-------|-----------|
| `user` | Data pelanggan beserta loyalty points |
| `driver` | Data pengemudi beserta informasi SIM dan kontak |
| `vehicle` | Armada kendaraan yang terdaftar per driver |
| `rekening` | Rekening bank driver (relasi one-to-many) |
| `orders` | Tabel pusat transaksi — menghubungkan user, driver, dan lokasi |
| `grab_bike` | Detail order untuk layanan motor |
| `grab_car` | Detail order untuk layanan mobil |
| `grab_express` | Detail order untuk layanan pengiriman barang |
| `payment` | Metode pembayaran per order (Cash/Cashless) |
| `promotion` | Kode promo dan diskon per user |
| `feedback` | Rating dan komentar per order |
| `ulasan` | Junction table yang menghubungkan feedback ke driver dan user |
| `location` | Master data lokasi pickup dan dropoff |

**Normalisasi: BCNF (Boyce-Codd Normal Form)** — memastikan zero redundansi data di seluruh tabel.

---

## 🛠 Tech Stack

| Layer | Tools |
|-------|-------|
| Database | MySQL 8.0.39 |
| Design | ERD (BCNF Normalization) |
| Querying | Advanced SQL — CTE, Subqueries, Multi-table JOIN, CASE WHEN |
| Reporting | MySQL Views |

---

## ✨ Key Features

**Multi-Service Architecture**
Satu database menangani tiga tipe layanan sekaligus (Bike, Car, Express) dengan tabel anak yang terhubung ke `orders` via foreign key — memisahkan atribut spesifik tiap layanan tanpa redundansi.

**Loyalty Tier System**
View `user_tier` mengklasifikasikan pelanggan ke tier Silver, Gold, atau Platinum berdasarkan akumulasi loyalty points — langsung usable untuk segmentasi marketing.

**Promotion & Pricing Engine**
View `total_harga` menghitung harga akhir setelah diskon untuk semua tipe layanan dalam satu query, menggunakan CASE WHEN untuk handle perbedaan atribut harga di tiap layanan.

**Driver Fleet Analytics**
View `statistik_vehicle` menyajikan statistik armada kendaraan (kapasitas, modus bahan bakar, modus jenis kendaraan) menggunakan nested subqueries.

---

## 📊 Views & Analytical Queries

### 1. `total_harga` — Final Price After Discount

Menghitung harga akhir per order dari semua tipe layanan, setelah memperhitungkan kode promo user yang aktif.

```sql
SELECT o.Order_ID, prom.promo_code,
  CASE
    WHEN o.order_type = 'grab_bike'    THEN gb.harga_awal_motor
    WHEN o.order_type = 'grab_express' THEN ge.harga_awal_barang
    WHEN o.order_type = 'grab_car'     THEN gc.harga_awal_mobil
    ELSE 0
  END AS harga_awal,
  IFNULL(prom.diskon, 1) AS diskon,
  ROUND(harga_awal * (1 - IFNULL(prom.diskon, 1)), 2) AS total_harga
FROM orders o
LEFT JOIN grab_bike gb    ON o.Order_ID = gb.Order_ID
LEFT JOIN grab_express ge ON o.Order_ID = ge.Order_ID
LEFT JOIN grab_car gc     ON o.Order_ID = gc.Order_ID
LEFT JOIN promotion prom  ON o.email = prom.email;
```

### 2. `user_tier` — Customer Loyalty Segmentation

```sql
SELECT email, poin_loyalitas,
  CASE
    WHEN poin_loyalitas BETWEEN 0 AND 300   THEN 'SILVER'
    WHEN poin_loyalitas BETWEEN 301 AND 500 THEN 'GOLD'
    WHEN poin_loyalitas > 500               THEN 'PLATINUM'
  END AS tier
FROM user;
```

### 3. `statistik_vehicle` — Fleet Statistics

Menggunakan nested subqueries untuk menghitung modus tipe bahan bakar dan jenis kendaraan dari seluruh armada.

---

## 📈 Key Results

| Metric | Value |
|--------|-------|
| Total Tabel | 10 tables + 3 views |
| Total Orders | 150 transaksi |
| Service Types | 3 (GrabBike, GrabCar, GrabExpress) |
| Normalization Level | BCNF |
| Total Users | 150 akun |
| Total Drivers | 200+ driver |
| Payment Methods | Cash & Cashless |
| Loyalty Tiers | Silver / Gold / Platinum |

---

## 🚀 Quick Start

### Prerequisites
- MySQL 8.0+

### Import Database

```bash
# Login ke MySQL
mysql -u root -p

# Buat database baru
CREATE DATABASE ride_hailing;
USE ride_hailing;

# Import schema dan data
SOURCE IF2040_Milestone3_K02_G12.sql;
```

### Verifikasi

```sql
-- Cek semua tabel
SHOW TABLES;

-- Cek total orders
SELECT COUNT(*) FROM orders;

-- Lihat distribusi tier user
SELECT tier, COUNT(*) FROM user_tier GROUP BY tier;

-- Lihat 5 order dengan harga final tertinggi
SELECT * FROM total_harga ORDER BY total_harga DESC LIMIT 5;
```

---

## 👤 Author

**Kelompok 12**

*Note: Proyek ini merupakan bagian dari tugas kuliah IF2040 (Database Modelling) yang dikerjakan secara berkelompok. Kontribusi personal mencakup perancangan ERD, implementasi schema, pengisian data, dan pembuatan analytical views.*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://linkedin.com/in/raditya-zaki-athaya)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?logo=github)](https://github.com/rdityaza)

---

## 📄 License

Distributed under the MIT License.
