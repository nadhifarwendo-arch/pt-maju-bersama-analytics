# Studi Kasus Data Analytics dan AI — PT Maju Bersama

## Ringkasan

Proyek ini mengadaptasi kasus yang dibahas pada kuliah praktisi **“Make Sense of Data with Analytics and AI”**. Dalam contoh pemateri, PT Maju Bersama adalah perusahaan distributor. Indikator yang dibahas menunjukkan pendapatan penjualan meningkat, laba bersih turun sekitar 14%, piutang meningkat sekitar 300%, dan arus kas operasi berkurang. Kesimpulan bisnis tidak boleh dibuat hanya dari satu indikator; data perlu dipersiapkan, dianalisis sesuai pertanyaan, diuji, lalu diterjemahkan menjadi rekomendasi.

**Catatan penting:** dataset yang disertakan sepenuhnya sintetis untuk demonstrasi akademik. Dataset ini bukan data asli PT Maju Bersama dan tidak dapat dijadikan bukti kondisi perusahaan sungguhan. Angka pada `monthly_financials.csv` sengaja dibuat untuk menirukan pola umum yang dijelaskan dalam transkrip. Dataset invoice menyimulasikan contoh keterlambatan pembayaran.

## Pertanyaan bisnis

1. Bagaimana perubahan pendapatan, laba bersih, piutang, dan arus kas operasi selama 24 bulan?
2. Mengapa kenaikan pendapatan belum tentu berarti kinerja keuangan membaik?
3. Invoice mana yang memiliki risiko terlambat lebih dari 30 hari setelah tanggal jatuh tempo, berdasarkan informasi yang tersedia saat invoice diterbitkan?
4. Bagaimana hasil analisis dapat membantu menentukan prioritas tindak lanjut penagihan?

## Keterkaitan dengan materi pemateri

- **Business understanding:** mendefinisikan pertanyaan bisnis sebelum memilih teknik analisis.
- **Data understanding & preparation:** memeriksa tipe data, nilai kosong, duplikasi, konsistensi, dan unit analisis.
- **Descriptive analytics:** menjelaskan apa yang berubah pada indikator finansial.
- **Diagnostic analytics:** membentuk hipotesis penyebab perubahan; hipotesis harus diperiksa lebih lanjut dan tidak dianggap sebagai penyebab yang sudah terbukti.
- **Predictive analytics:** membandingkan model klasifikasi untuk memperkirakan risiko keterlambatan pembayaran.
- **Evaluation:** tidak hanya melihat akurasi; periksa recall, F1, ROC-AUC dan average precision, terutama karena kasus terlambat relatif lebih sedikit.
- **Decision support:** menyusun daftar invoice berisiko untuk ditinjau manusia, bukan langsung memutuskan pembatasan kredit secara otomatis.
- **Generative AI:** dapat membantu membuat kode, dokumentasi, atau alur kerja, tetapi hasilnya harus diverifikasi dan data sensitif tidak boleh ditempelkan tanpa izin.

## Struktur folder

```text
pt-maju-bersama-analytics/
├── data/
│   ├── monthly_financials.csv
│   └── invoices.csv
├── src/
│   └── analisis.py
├── outputs/                 # hasil dibuat saat program dijalankan
├── laporan_studi_kasus.md
├── requirements.txt
└── .gitignore
```

## Dataset

### `monthly_financials.csv`

Satu baris mewakili satu bulan. Kolom keuangan dinyatakan dalam **juta rupiah**.

- `month`: bulan pencatatan.
- `revenue_million_idr`: pendapatan penjualan.
- `net_profit_million_idr`: laba bersih.
- `accounts_receivable_million_idr`: piutang pada bulan tersebut.
- `operating_cash_flow_million_idr`: arus kas operasi.

### `invoices.csv`

Satu baris mewakili satu invoice, tetapi sengaja disisipkan beberapa baris duplikat dan nilai kosong untuk latihan data preparation. Program menghapus invoice ID duplikat dan mengimputasi nilai fitur yang kosong melalui pipeline.

- `invoice_id`: ID invoice.
- `issue_date`: tanggal invoice diterbitkan.
- `customer_segment`, `payment_method`: informasi yang diketahui pada saat penerbitan.
- `credit_days`: jangka waktu kredit.
- `invoice_amount_idr`: nilai invoice dalam rupiah.
- `historical_late_rate`: tingkat keterlambatan historis pelanggan yang diasumsikan telah tersedia saat itu.
- `discount_pct`: persentase diskon.
- `late_30d`: target simulasi; `1` berarti pembayaran diasumsikan terlambat lebih dari 30 hari setelah tanggal jatuh tempo, `0` selain itu.

Target `late_30d` disediakan agar proyek dapat dijalankan tanpa mengakses sistem perusahaan. Untuk penggunaan nyata, target harus dibuat dari riwayat pembayaran dengan definisi waktu yang konsisten. Fitur yang hanya diketahui setelah jatuh tempo, seperti tanggal pembayaran aktual atau status akhir invoice, tidak dimasukkan sebagai prediktor agar tidak terjadi **data leakage**.

## Cara menjalankan

Pastikan Python 3.10+ tersedia. Dari folder proyek jalankan:

```bash
python -m venv .venv
```

Aktifkan lingkungan virtual:

- Windows PowerShell: `\.venv\Scripts\Activate.ps1`
- macOS/Linux: `source .venv/bin/activate`

Install dependensi dan jalankan program:

```bash
pip install -r requirements.txt
python src/analisis.py
```

Hasil tabel dan grafik disimpan ke folder `outputs/`. File hasil meliputi ringkasan KPI, hasil validasi model, perbandingan metrik model di data uji, grafik tren pendapatan/laba/piutang/arus kas, confusion matrix, dan daftar invoice berisiko untuk pemeriksaan.

## Contoh hasil eksekusi

Dari dataset sintetis yang disertakan, ringkasan finansial yang dihitung program adalah:

| Indikator | Tahun 1 | Tahun 2 | Perubahan |
|---|---:|---:|---:|
| Pendapatan | Rp1.401 juta | Rp1.681,2 juta | +20% |
| Laba bersih | Rp190 juta | Rp163,4 juta | -14% |
| Piutang (awal ke akhir periode) | Rp30 juta | Rp120 juta | +300% |
| Arus kas operasi | Rp130 juta | Rp79 juta | -Rp51 juta |

Model dipilih menggunakan average precision di data validasi kronologis. Dalam satu kali run dengan seed tetap, Random Forest terpilih pada validasi, tetapi metrik data uji masih menunjukkan keterbatasan (recall 0,391). Logistic Regression mempunyai average precision data uji lebih tinggi pada run ini; hasil itu tidak digunakan untuk mengganti model terpilih karena data uji semestinya tidak dipakai untuk pemilihan model. Ini contoh mengapa model harus dievaluasi dengan hati-hati dan tidak langsung diterapkan ke keputusan bisnis.

### Grafik tren keuangan

[Tren pendapatan]
<img width="1500" height="825" alt="tren_pendapatan" src="https://github.com/user-attachments/assets/9ca8d020-f3fd-4bae-869d-d1677d39a3b8" />

[Tren laba bersih]
<img width="1500" height="825" alt="tren_laba_bersih" src="https://github.com/user-attachments/assets/ec7fc849-63a4-4eab-9d7e-f0d47fc0e01b" />

[Tren Piutang]
<img width="1500" height="825" alt="tren_piutang" src="https://github.com/user-attachments/assets/37ded3a8-6ff5-40d4-b91e-b63155765f10" />

[Tren arus kas operasi]
<img width="1500" height="825" alt="tren_arus_kas" src="https://github.com/user-attachments/assets/b80c959e-f851-4590-b862-6a09eeeb9a90" />

## Cara membaca hasil

- Jika pendapatan meningkat tetapi laba menurun, periksa beban operasional, harga pokok, diskon, serta komposisi produk/pelanggan. Data ini hanya memberi petunjuk awal, bukan bukti sebab-akibat.
- Piutang yang meningkat dapat menekan kas yang tersedia. Tinjau umur piutang, keterlambatan pelanggan, dan pola pembayaran.
- Akurasi tinggi belum cukup jika recall untuk invoice terlambat rendah. Model dengan recall nol tidak menangkap satu pun invoice terlambat pada data uji sehingga tidak layak langsung digunakan untuk prioritas penagihan.
- Hasil model hanya alat bantu. Tim keuangan perlu memeriksa konteks pelanggan, kebijakan, dan risiko salah prediksi sebelum bertindak.

## Batasan

Data sintetis, volume data kecil, dan pola yang dibangkitkan tidak mencerminkan seluruh kompleksitas bisnis. Hasil model bukan prediksi operasional, bukan nasihat keuangan, dan tidak boleh dianggap sebagai bukti hubungan kausal. Penggunaan nyata memerlukan data yang berizin, definisi target yang disepakati, validasi berdasarkan waktu, pemantauan kinerja, serta tinjauan manusia.
