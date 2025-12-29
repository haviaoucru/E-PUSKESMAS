### 1. Ringkasan Proyek

Notebook ini dirancang untuk menarik, mengolah, dan menganalisa data pelayanan kesehatan dari sistem **E-PUSKESMAS**. Fokus utamanya adalah melakukan monitoring kualitas data (kelengkapan/ *missing values*), menghitung statistik kunjungan pasien berdasarkan diagnosis (ICD-10) dan tenaga medis, serta mengekspor hasil pengolahan ke dalam file Excel yang terstandarisasi untuk setiap Puskesmas.

### 2. Dataset yang Digunakan

* **Sumber Data**: Google BigQuery.
* **Tabel Utama**: `stellar-orb-451904-d9.dashboard.epus_pelayanan_assessment`.
* **Rentang Waktu**: Data yang ditarik dalam contoh ini adalah antara tanggal **15 Desember 2025** hingga **21 Desember 2025**.
* **Fitur Utama**: NIK, nama pasien, tanggal lahir, jenis kelamin, nama puskesmas, poli, keluhan, diagnosis (ICD-10), resep, serta tanda-tanda vital (tinggi badan, berat badan, sistole, diastole, dll.).

### 3. Alur Kerja (Workflow) & Logika Kode

Berikut adalah bedah blok kode dari awal hingga akhir:

#### **A. Inisialisasi & Pengambilan Data (Blok 1 - 4)**

* **Logika**: Menyiapkan *environment* dengan menginstal library yang diperlukan (`pandas-gbq`, `google-cloud-bigquery`), melakukan autentikasi Google Colab, dan menginisialisasi BigQuery Client.
* **Proses**: Menjalankan query SQL untuk menarik data mentah. Kode menggunakan `re` (regex) untuk mengekstrak informasi filter tanggal dari string query guna keperluan pencatatan log.

#### **B. Eksplorasi Awal (Blok 5 - 7)**

* **Logika**: Memahami distribusi data berdasarkan nama Puskesmas dan melihat sekilas struktur tabel.
* **Hasil**: Dihasilkan tabel jumlah kunjungan per Puskesmas (misal: Puskesmas Rembang memiliki kunjungan terbanyak dengan 1258 data).

#### **C. Pengolahan & Ekspor Data ke Excel (Blok 8 - 9)**

* **Logika**: Ini adalah bagian paling krusial. Tujuannya adalah merapikan kolom agar sesuai standar laporan dan memisahkan data ke dalam *sheet* Excel yang berbeda berdasarkan nama Puskesmas.
* **Detail Teknis**:
* **Normalisasi**: Nama puskesmas diubah menjadi *uppercase* dan dibersihkan dari spasi berlebih.
* **Fungsi `nik_to_text**`: Mengatasi masalah umum di Excel di mana NIK (angka panjang) sering berubah menjadi format ilmiah (misal: 3.3E+15). Solusinya adalah menambahkan karakter *non-breaking space* (`\u00A0`) di depan NIK agar dibaca sebagai teks utuh oleh Excel.
* **ExcelWriter**: Loop dilakukan untuk setiap puskesmas unik, lalu data disimpan ke sheet masing-masing dengan penamaan sheet maksimal 31 karakter (limitasi Excel).



#### **D. Data Cleaning & Penyesuaian Tipe Data (Blok 10 - 12)**

* **Logika**: Mengubah tipe data kolom tanggal yang semula bertipe *object* menjadi *datetime* dan kolom NIK menjadi numerik agar bisa diolah secara matematis.
* **Penting**: Penggunaan `errors='coerce'` memastikan jika ada format tanggal yang salah, data tersebut akan diubah menjadi `NaT` (Not a Time) daripada merusak jalannya kode.

#### **E. Analisis Statistik Diagnosis & Nakes (Blok 13 - 15)**

* **Logika**: Menghitung prevalensi penyakit berdasarkan kode ICD-10 di setiap puskesmas dan beban kerja setiap Dokter/Tenaga Medis (jumlah pasien yang dilayani).
* **Contoh Hasil**: Menunjukkan jumlah pasien hipertensi (I10) atau asma (J45) di setiap lokasi puskesmas secara mendetail.

#### **F. Analisis Kualitas Data / Completeness (Blok 16 - 21)**

* **Logika**: Mengukur sejauh mana kelengkapan data yang diinput oleh petugas puskesmas. Kolom-kolom vital seperti NIK, Tinggi Badan, Berat Badan, dan Suhu diperiksa jumlah *missing values*-nya.
* **Proses**:
* Dibuat variabel `important_columns` untuk memfokuskan pengecekan pada 14 kolom kritis.
* Dihasilkan tabel persentase kelengkapan per Puskesmas. Sebagai contoh, di Dinas Kesehatan, kolom NIK memiliki *missing value* sebesar 21.79%, sementara diagnosis 1 selalu lengkap (100%).



### 4. Hal-hal Penting untuk Penerus saya

1. **Format NIK**: Jika NIK di file Excel terlihat aneh, jelaskan bahwa itu sudah ditangani dengan prefix spasi tak terlihat agar tidak terpotong oleh Excel.
2. **Missing Values pada Suhu**: Secara sistematis, kolom 'Suhu' seringkali memiliki persentase *missing* yang tinggi (misal: 79% di Karangmoncol), ini bisa menjadi temuan awal untuk koordinasi lapangan terkait kedisiplinan input data tanda vital.
3. **Sheet Excel**: Penamaan sheet otomatis mengikuti nama Puskesmas, namun dipotong jika lebih dari 31 karakter.

Dengan dokumentasi ini, Lu seharusnya dapat menjalankan ulang notebook ini hanya dengan mengubah rentang tanggal di blok pengambilan data BigQuery.
