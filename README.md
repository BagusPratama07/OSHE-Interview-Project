# 🏗️ OSHE Mining Accident Analysis Project

**Data Engineering & Analytics Pipeline for Mining Safety Analysis**

---

## 📌 Project Overview

Project ini bertujuan untuk membangun **data pipeline end-to-end** untuk mengolah dan menganalisis data kecelakaan tambang menggunakan Python, SQL Server, dan Power BI.

Fokus utama project:

- Mengelola data mentah → data bersih → data analisis
- Membangun pipeline sederhana
- Melakukan eksplorasi dan clustering
- Menyajikan insight melalui dashboard interaktif

Project ini dibuat sebagai **portfolio dan simulasi implementasi data engineering & data analyst di lingkungan kerja.**

---

## 🏢 Business Context

Industri pertambangan memiliki risiko kecelakaan kerja yang cukup tinggi. Data yang digunakan bersumber dari **MSHA (Mine Safety and Health Administration)**, lembaga pemerintah Amerika Serikat yang bertanggung jawab atas keselamatan tambang. Dataset mencakup data kecelakaan dari seluruh negara bagian di **United States** sepanjang periode **2000–2026**.

Data kecelakaan ini dapat dimanfaatkan untuk:

- Mengidentifikasi pola risiko berdasarkan lokasi dan waktu
- Memahami faktor penyebab kecelakaan yang paling dominan
- Mendukung evaluasi program keselamatan berdasarkan tingkat keparahan
- Membantu manajemen mengambil keputusan berbasis data yang lebih akurat

---

## 🏗️ Technical Architecture

Alur sistem secara umum:

```
Raw Data (ZIP dari MSHA)
        ↓
  Staging CSV
        ↓
  Python Processing (Cleaning, Feature Engineering, K-Means Clustering)
        ↓
  SQL Server (Penyimpanan Terstruktur)
        ↓
  Power BI Dashboard (Visualisasi & Reporting)
```

Penjelasan:

1. Data mentah diunduh dari sumber resmi MSHA dan disimpan di folder `data/raw`
2. Data dibersihkan dan ditransformasi, lalu disimpan di `data/staging`
3. Proses ETL dan feature engineering dilakukan dengan Python
4. Data final disimpan ke SQL Server untuk konsumsi Power BI
5. Visualisasi dan dashboard interaktif dibuat dengan Power BI

---

## 🔄 Data Pipeline

### 1. Data Extraction

- **Sumber:** MSHA Open Government Data — [Accidents.zip](https://arlweb.msha.gov/OpenGovernmentData/DataSets/Accidents.zip)
- **Scope:** Seluruh negara bagian United States, tahun 2000–2026
- **Format:** ZIP / TXT
- **Output:** File CSV di folder staging

### 2. Data Cleaning

Proses pembersihan data mencakup:

- Menghapus data kosong dan duplikat
- Mengatur tipe data sesuai kebutuhan analisis
- Standarisasi teks dan nilai kategori
- Handling missing value pada kolom kritis seperti `CLUSTER_LABEL` — rows dengan `IS_FATAL = 1` secara alami memiliki `CLUSTER_LABEL = null` dan dilabel sebagai **"CRITICAL - Fatal Risk"** melalui custom column di Power Query

### 3. Feature Engineering

Fitur tambahan yang dibuat:

- **Bulan & Hari kejadian** — untuk analisis temporal
- **SEVERITY_SCORE** — skor keparahan kecelakaan
- **IS_FATAL** — flag binary (1 = fatal, 0 = non-fatal)
- **TOT_EXPER** — total pengalaman kerja pekerja yang terlibat
- **State_Name** — mapping dari kode FIPS ke nama negara bagian menggunakan DAX SWITCH + VALUE() conversion
- **CLUSTER_LABEL** — hasil dari K-Means clustering yang dikombinasikan dengan logika conditional untuk fatal cases

### 4. Data Storage

- **Database:** SQL Server
- **Tujuan:** Menyimpan data terstruktur yang siap dikonsumsi oleh Power BI
- **Koneksi:** Power BI terhubung langsung ke SQL Server sebagai data source

### 5. Analytics & Clustering

- **Metode:** K-Means Clustering
- **Pre-processing:** Data distandardisasi dengan StandardScaler sebelum clustering
- **Jumlah cluster:** Ditentukan melalui evaluasi elbow method dan silhouette score
- **Output:** 4 cluster utama yang mengidentifikasi pola risiko berbeda

### 6. Visualization

- **Tools:** Power BI Desktop
- **Output:** Dashboard interaktif dengan filter tahun, tingkat risiko, dan kluster label
- **Visuals:** KPI Cards, Area Chart, Donut Chart, Scatter Plot, Bar Chart, dan Map (United States)

---

## 🤖 Clustering Method

- **Algoritma:** K-Means
- **Pre-processing:** StandardScaler untuk normalisasi fitur
- **Evaluasi:** Elbow method dan silhouette score untuk menentukan jumlah cluster optimal
- **Hasil:** 4 cluster dengan distribusi sebagai berikut:

| Cluster Label | Persentase | Deskripsi |
|---|---|---|
| Low Risk - Maintenance | 47.09% | Kluster terbesar, kecelakaan dengan tingkat risiko rendah di area maintenance |
| Moderate Severity | 28.31% | Kecelakaan dengan tingkat keparahan sedang |
| Low Risk - Operational | 23.74% | Kecelakaan risiko rendah di area operasional |
| CRITICAL - Fatal Risk | 0.86% | Kecelakaan fatal — jumlah kecil namun dampak paling signifikan |

> **Catatan penting:** Rows dengan `IS_FATAL = 1` secara alami tidak mendapatkan cluster label dari K-Means. Label "CRITICAL - Fatal Risk" diberikan secara manual melalui conditional logic di Power Query berdasarkan nilai `IS_FATAL = 1`.

Tujuan utama clustering:

> Bukan untuk prediksi, tetapi untuk memahami dan mengelompokkan pola kecelakaan berdasarkan karakteristiknya.

---

## 📊 Dashboard KPI Summary

Berikut metrik utama yang ditampilkan di dashboard berdasarkan data terkini:

| Metrik | Nilai | Interpretasi |
|---|---|---|
| Total Kecelakaan | 270,921 | Total kasus tercatat sepanjang 2000–2026 |
| Kasus Fatal | 1,194 | Kecelakaan yang mengakibatkan kematian |
| Tingkat Fatalitas | 0.44% | Rasio kasus fatal terhadap total kecelakaan |
| Tingkat Keparahan | 75.7 | Skor keparahan rata-rata |
| Kasus Risiko Tinggi | 2,318 | Kecelakaan yang dikategorikan HIGH risk |
| Avg Pengalaman (Kasus Fatal) | 13.06 tahun | Rata-rata pengalaman kerja pekerja pada kasus fatal |

---

## 📈 Key Findings

Berdasarkan analisis data dan visualisasi dashboard:

### 1. Kecelakaan Fatal Jumlahnya Kecil Namun Memiliki Pola Khusus
Dari 270,921 total kecelakaan, hanya **1,194 kasus (0.44%)** yang bersifat fatal. Meskipun persentasenya sangat kecil, fatal cases membentuk cluster tersendiri (**CRITICAL - Fatal Risk, 0.86%**) dan membutuhkan penanganan prioritas yang berbeda dari kecelakaan non-fatal.

### 2. Pekerja Berpengalaman Tetap Bisa Mengalami Kecelakaan Fatal
Rata-rata pengalaman kerja pada kasus fatal adalah **13.06 tahun**. Ini menunjukkan bahwa pengalaman kerja yang tinggi **tidak menjamin** bebas dari kecelakaan fatal. Faktor lain seperti jenis pekerjaan dan kondisi operasional memainkan peran penting.

### 3. Maintenance Adalah Area Risiko Terdominasi
Cluster **Low Risk - Maintenance** mencakup **47.09%** dari seluruh kecelakaan — menjadi kluster terbesar. Meskipun dikategorikan "low risk", volume yang sangat tinggi menunjukkan bahwa area maintenance membutuhkan perhatian dari sisi frekuensi kejadian.

### 4. Tren Kecelakaan Menurun Signifikan Sejak 2010
Area chart menunjukkan bahwa puncak kecelakaan terjadi sekitar **2005–2008**, diikuti penurunan yang konsisten dari **2010 hingga 2025**. Tren ini menunjukkan efektivitas kebijakan keselamatan yang diimplementasikan selama periode tersebut.

### 5. Penyebab Kecelakaan Didominasi Oleh Kondisi Operasional
Top 5 penyebab kecelakaan teratas adalah:
- **Accident type without [injuries]** — ~30K kasus (tertinggi)
- **Over-exertion NEC** — ~26K kasus
- **Struck by NEC** — ~25K kasus
- **Struck by falling object** — ~20K kasus
- **Over-exertion in lifting** — ~14K kasus

Kelima penyebab ini semuanya terkait dengan aktivitas fisik dan kondisi operasional di lapangan.

### 6. Distribusi Risiko Terkonsentrasi di Wilayah Tertentu
Peta distribusi risiko menunjukkan bahwa kecelakaan **HIGH risk** terkonsentrasi di beberapa negara bagian tertentu di United States, menunjukkan bahwa ada perbedaan signifikan antar wilayah operasi.

---

## 💡 Business Recommendations

### 1. Prioritaskan Investigasi Mendalam untuk Kasus Fatal
**Dasar data:** Tingkat fatalitas 0.44% dengan rata-rata pengalaman pekerja 13.06 tahun menunjukkan bahwa kasus fatal tidak hanya menimpa pekerja baru.

**Rekomendasi:**
- Lakukan root cause analysis pada setiap kasus fatal secara individual
- Jangan hanya fokus pada pekerja baru — evaluasi juga pekerja senior
- Dokumentasikan pola dan bagikan sebagai "lesson learned" antar site

**Manfaat:** Pencegahan kasus fatal yang lebih efektif dan targeted.

---

### 2. Tingkatkan Monitoring di Area Maintenance
**Dasar data:** Kluster Low Risk - Maintenance mencakup 47.09% dari total kecelakaan.

**Rekomendasi:**
- Meskipun dikategorikan "low risk", tingginya volume menunjukkan pola yang perlu diperhatikan
- Implementasikan safety control yang lebih ketat di area maintenance
- Lakukan audit rutin pada prosedur maintenance

**Manfaat:** Pengurangan frekuensi kecelakaan di area yang paling sering terjadi.

---

### 3. Fokuskan Pelatihan Pada Aktivitas Fisik & Operasional
**Dasar data:** 5 penyebab kecelakaan teratas semuanya terkait aktivitas fisik (over-exertion, struck by, falling object).

**Rekomendasi:**
- Perkuat pelatihan teknik kerja yang aman untuk aktivitas fisik berat
- Sediakan peralatan ergonomis dan protective equipment yang sesuai
- Edukasi berkelanjutan tentang prosedur penanganan beban dan benda berat

**Manfaat:** Penurunan kecelakaan pada kategori penyebab tertinggi.

---

### 4. Manfaatkan Tren Penurunan Sebagai Benchmark
**Dasar data:** Kecelakaan menurun signifikan dari 2010 ke 2025.

**Rekomendasi:**
- Identifikasi kebijakan atau program apa yang diterapkan sejak 2010 dan evaluasi efektivitasnya
- Gunakan tren positif ini sebagai benchmark untuk target ke depan
- Jangan berhenti pada pencapaian saat ini — tetapkan target penurunan lebih lanjut

**Manfaat:** Optimalisasi kebijakan keselamatan berbasis bukti historis.

---

### 5. Pemanfaatan Dashboard untuk Evaluasi Rutin
**Rekomendasi:**
- Gunakan dashboard untuk monitoring bulanan dan quarterly review
- Filter berdasarkan tahun dan risk level untuk analisis per periode
- Integrasikan dashboard ke dalam proses reporting manajemen

**Manfaat:** Pengambilan keputusan keselamatan yang lebih cepat dan berbasis data.

> **Catatan:** Rekomendasi bersifat pendukung analisis berdasarkan data yang tersedia. Implementasi kebijakan seharusnya mempertimbangkan konteks operasional dan regulasi yang berlaku.

---

## 📁 Project Structure

```
OSHE-INTERVIEW-PROJECT/
│
├── data/
│   ├── raw/
│   │   └── accidents_raw.zip          # Source data dari MSHA
│   └── staging/
│       └── accidents_staging.csv      # Data setelah cleaning
│
├── python/
│   ├── data_pipeline.ipynb            # ETL & data cleaning
│   └── master_sql_pipeline.ipynb      # Pipeline ke SQL Server + Clustering
│
├── powerbi/
│   └── dashboard.pbix                 # Power BI Dashboard
│
└── README.md                          # File ini
```

---

## 🛠️ Technologies Used

- **Python** — pandas, numpy, scikit-learn (K-Means, StandardScaler)
- **SQL Server** — Data storage dan query
- **Power BI Desktop** — Dashboard dan visualisasi
- **Power Query** — Data transformation dan custom columns
- **DAX** — Calculated columns dan measures
- **Jupyter Notebook** — Eksekusi pipeline
- **VS Code** — Development environment
- **Git** — Version control

---

## ⚠️ Technical Notes

### FIPS State Code Mapping
Kolom `FIPS_STATE_CD` di sumber data berisi kode numerik US FIPS. Power BI tidak menggenali angka ini sebagai kode negara bagian secara otomatis. Solusi yang diimplementasikan adalah membuat calculated column `State_Name` menggunakan **DAX SWITCH** dengan **VALUE()** conversion:

```dax
State_Name = SWITCH(TRUE(),
VALUE([FIPS_STATE_CD])=1,"Alabama",
VALUE([FIPS_STATE_CD])=2,"Alaska",
...
VALUE([FIPS_STATE_CD])=56,"Wyoming",
"Unknown")
```

### CLUSTER_LABEL untuk Fatal Cases
Rows dengan `IS_FATAL = 1` secara alami memiliki `CLUSTER_LABEL = null` dari hasil K-Means clustering. Label ini diisi melalui **Custom Column di Power Query**:

```
if [IS_FATAL] = 1 then "CRITICAL - Fatal Risk"
else [CLUSTER_LABEL]
```

---

## ▶️ How to Run

1. Clone repository dan install dependencies Python
2. Jalankan `data_pipeline.ipynb` untuk extraction dan cleaning
3. Jalankan `master_sql_pipeline.ipynb` untuk load data ke SQL Server dan jalankan clustering
4. Buka `dashboard.pbix` di Power BI Desktop
5. Update data source connection ke SQL Server Anda
6. Klik **Refresh** untuk memuat data terbaru

---

## 🎯 Project Purpose

Project ini dibuat untuk:

- Melatih implementasi **data pipeline end-to-end** dari raw data hingga dashboard
- Mempraktikkan **ETL process** dengan Python dan SQL Server
- Menghubungkan ekosistem **Python → SQL Server → Power BI**
- Mengaplikasikan **machine learning (K-Means)** untuk analisis pola data
- Mengembangkan **portfolio profesional** di bidang data engineering dan analytics

---

## 👤 Author

**Bagus Diaz Pratama**
Data Analyst / Data Engineer Enthusiast

---

## 📞 Contact & Support

**Project Owner:** Bagus Diaz Pratama
**Email:** bagusdiazp@gmail.com
**LinkedIn:** www.linkedin.com/in/bagus-diaz-pratama

---

*Last Update: February 2026*
