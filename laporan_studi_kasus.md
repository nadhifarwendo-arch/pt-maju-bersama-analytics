# Laporan Studi Kasus

## Analisis Kinerja Keuangan dan Prediksi Keterlambatan Pembayaran Invoice PT Maju Bersama dengan Data Analytics dan AI

### 1. Latar Belakang

Materi kuliah praktisi membahas pentingnya memahami data sebelum mengambil keputusan. Contoh yang disampaikan adalah PT Maju Bersama, perusahaan distributor yang memiliki banyak tabel dan transaksi. Dalam ilustrasi pemateri, pendapatan penjualan meningkat, tetapi laba bersih turun sekitar 14%, piutang naik sekitar 300%, dan arus kas operasi berkurang. Kondisi tersebut menunjukkan bahwa kenaikan penjualan saja belum cukup untuk menyatakan kinerja bisnis membaik.

Studi kasus ini menerapkan alur analitik untuk merangkum indikator keuangan dan membuat prototipe klasifikasi risiko keterlambatan pembayaran invoice. Seluruh data pada proyek adalah data sintetis untuk kebutuhan pembelajaran, bukan data asli perusahaan.

### 2. Rumusan Masalah

1. Bagaimana arah perubahan pendapatan, laba bersih, piutang, dan arus kas operasi selama dua tahun?
2. Indikator apa yang harus diperiksa lebih lanjut ketika pendapatan meningkat tetapi laba dan arus kas melemah?
3. Dapatkah model klasifikasi membantu menandai invoice yang berpotensi terlambat lebih dari 30 hari setelah jatuh tempo?
4. Metrik apa yang layak dipakai untuk menilai model ketika invoice terlambat merupakan kelas minoritas?

### 3. Tujuan

- Menerapkan business understanding, data preparation, analisis deskriptif/diagnostik, pemodelan prediktif, dan evaluasi.
- Menunjukkan bahwa kualitas dan granularitas data perlu diperiksa sebelum tabel digabung atau dipakai untuk analisis.
- Membandingkan Logistic Regression, Decision Tree, dan Random Forest pada data invoice sintetis.
- Menghasilkan rekomendasi awal yang dapat ditinjau oleh pengambil keputusan.

### 4. Data dan Asumsi

Proyek menggunakan dua dataset sintetis. `monthly_financials.csv` berisi 24 bulan, dengan nilai keuangan dalam juta rupiah. Data dirancang untuk menampilkan pola pendapatan yang naik, laba bersih yang menurun, piutang yang meningkat, dan arus kas operasi yang lebih rendah pada tahun kedua. `invoices.csv` berisi data invoice sintetis dan memiliki sejumlah nilai kosong serta baris duplikat yang sengaja disisipkan sebagai contoh masalah kualitas data.

Target prediksi `late_30d` adalah label simulasi: nilai 1 menunjukkan invoice diasumsikan terlambat lebih dari 30 hari setelah tanggal jatuh tempo. Prediktor terbatas pada informasi yang diasumsikan tersedia saat invoice diterbitkan, seperti nilai invoice, termin kredit, segmen pelanggan, metode pembayaran, tingkat keterlambatan historis, dan persentase diskon. Tanggal pembayaran aktual atau status akhir tidak digunakan sebagai prediktor karena dapat menyebabkan kebocoran target (*data leakage*).

### 5. Metode

#### 5.1 Business Understanding

Pertanyaan bisnis diubah menjadi pertanyaan analitik yang bisa dijawab dengan data: perubahan indikator keuangan bulanan dan identifikasi awal invoice berisiko. Definisi indikator dan periode analisis perlu disepakati dengan pihak bisnis sebelum penerapan nyata.

#### 5.2 Data Understanding dan Preparation

Program membaca CSV, memeriksa nilai kosong dan duplikasi, menghapus duplikasi berdasarkan `invoice_id`, mengubah tanggal ke format yang sesuai, lalu memproses fitur numerik dan kategorikal. Nilai fitur yang hilang diimputasi di dalam pipeline, sehingga strategi preprocessing diterapkan pada data latih dan digunakan kembali pada data uji. Dalam kasus nyata, anomali bisnis tetap perlu diperiksa oleh orang yang memahami proses perusahaan.

#### 5.3 Descriptive Analytics

Indikator bulanan diplot agar perubahan pendapatan, laba bersih, piutang, dan arus kas operasi terlihat. Analisis ini menjawab “apa yang terjadi?”, bukan secara langsung “mengapa hal itu terjadi?”.

#### 5.4 Diagnostic Analytics

Hipotesis yang perlu diuji antara lain peningkatan beban operasional, perubahan margin produk, perubahan komposisi pelanggan, keterlambatan penagihan, atau perubahan metode pembayaran. Hipotesis tersebut tidak dapat dinyatakan sebagai penyebab sebelum ada bukti tambahan, pemeriksaan data yang sesuai, dan konfirmasi dengan pemilik proses.

#### 5.5 Predictive Analytics

Tiga model dibandingkan: Logistic Regression, Decision Tree, dan Random Forest. Pemisahan data dilakukan secara kronologis menjadi data latih, validasi, dan uji (70%/15%/15%) agar periode yang lebih baru tidak dipakai untuk melatih model. Model dipilih berdasarkan average precision pada data validasi; metrik akhir dilaporkan pada data uji. Pipeline menggunakan imputasi fitur, standardisasi untuk fitur numerik pada Logistic Regression, dan one-hot encoding untuk fitur kategorikal.

Evaluasi mencakup accuracy, precision, recall, F1, ROC-AUC, dan average precision. Recall penting karena model dengan akurasi tinggi masih dapat gagal menemukan sebagian besar invoice terlambat. Average precision turut dilaporkan karena kelas terlambat lebih sedikit daripada kelas tidak terlambat.

#### 5.6 Rekomendasi dan Pengawasan

Invoice yang diprediksi berisiko dapat dijadikan daftar prioritas untuk pemeriksaan atau pengingat pembayaran oleh staf. Model tidak boleh otomatis membatasi kredit atau mengambil keputusan penting tanpa validasi manusia, evaluasi biaya salah prediksi, dan kebijakan perusahaan.

### 6. Implementasi

Implementasi dilakukan menggunakan Python, pandas, matplotlib, dan scikit-learn. Jalankan `python src/analisis.py`. Program menghasilkan:

- `outputs/ringkasan_kpi.csv` — perbandingan indikator keuangan;
- `outputs/validasi_model.csv` — evaluasi untuk pemilihan model;
- `outputs/perbandingan_model.csv` — metrik model pada data uji;
- `outputs/invoice_berisiko.csv` — daftar invoice yang perlu ditinjau;
- grafik tren indikator keuangan dan confusion matrix;
- `outputs/laporan_run.txt` — catatan kualitas data dan ringkasan eksekusi.

Pada dataset sintetis yang disediakan, total pendapatan tahun pertama adalah Rp1.401 juta dan tahun kedua Rp1.681,2 juta (+20%). Laba bersih berubah dari Rp190 juta menjadi Rp163,4 juta (-14%); piutang pada titik awal dan akhir berubah dari Rp30 juta menjadi Rp120 juta (+300%); arus kas operasi tahunan berubah dari Rp130 juta menjadi Rp79 juta (-Rp51 juta). Angka-angka ini sengaja disusun untuk mendemonstrasikan pola materi, bukan data perusahaan nyata. Angka metrik model yang keluar juga merupakan hasil dari data sintetis dan seed eksperimen ini, bukan performa di perusahaan nyata. Pada satu kali run, model terpilih berdasarkan average precision validasi adalah Random Forest (validation average precision sekitar 0,335). Pada data uji, Random Forest memiliki accuracy 0,817, precision 0,321, recall 0,391, F1 0,353, ROC-AUC 0,681, dan average precision 0,248. Logistic Regression pada data uji memiliki accuracy 0,683, precision 0,250, recall 0,739, F1 0,374, ROC-AUC 0,725, dan average precision 0,278. Decision Tree memiliki accuracy 0,644, precision 0,185, recall 0,522, F1 0,273, ROC-AUC 0,603, dan average precision 0,229. Angka dapat dibaca langsung pada `outputs/perbandingan_model.csv`. Walau Logistic Regression lebih tinggi pada beberapa metrik data uji, model tidak dipilih ulang berdasarkan hasil data uji, karena hal itu akan menggunakan data uji untuk pemilihan model. Hasil ini menunjukkan kinerja masih terbatas dan perlu data lebih banyak serta validasi lebih lanjut sebelum penggunaan nyata.

### 7. Pembahasan

Kasus ini menegaskan alur *read, transform, analyze, visualize* yang dibahas pemateri. Analisis dimulai dengan pertanyaan yang jelas, bukan dengan memasukkan semua kolom dari seluruh tabel secara membabi buta. Perbedaan granularitas data dan join yang tidak tepat dapat menggandakan baris, sehingga jumlah transaksi atau nilai keuangan menjadi bias. Kualitas data juga perlu diperiksa karena “garbage in, garbage out”: keluaran analisis tidak akan dapat diandalkan jika input salah atau tidak lengkap.

Hasil deskriptif digunakan untuk menemukan pola. Setelah itu, analisis diagnostik mengajukan dan menguji kemungkinan penjelasan. Model prediktif membantu memperkirakan risiko, tetapi harus dievaluasi dengan metrik yang tepat. Jika recall rendah, banyak invoice yang benar-benar terlambat bisa terlewat meskipun akurasi terlihat tinggi.

Generative AI dapat membantu menyusun kerangka kode, menjelaskan workflow, atau membuat dokumentasi. Namun, analis tetap perlu memeriksa kode, memastikan definisi bisnis benar, menguji hasil, dan tidak memasukkan data pelanggan sensitif ke layanan AI tanpa izin.

### 8. Kesimpulan

Analisis data membutuhkan gabungan pemahaman bisnis, kualitas data, statistik, pemodelan, visualisasi, dan interpretasi. Pada contoh PT Maju Bersama, pendapatan yang meningkat tidak cukup untuk menyatakan keadaan perusahaan sehat ketika laba bersih, piutang, dan arus kas menunjukkan sinyal berbeda. Perlu pemeriksaan tambahan atas biaya, margin, perilaku pelanggan, dan proses penagihan. Klasifikasi invoice dapat digunakan sebagai prototipe pendukung prioritas kerja, tetapi hasil model harus dinilai dengan metrik yang sesuai serta diverifikasi secara manual.

### 9. Saran

Gunakan data transaksi yang sah dan lengkap jika proyek akan diterapkan pada bisnis nyata. Pastikan definisi pendapatan, laba, piutang, periode pengukuran, serta label keterlambatan disepakati lebih dahulu. Periksa duplikasi, nilai kosong, kategori tidak konsisten, anomali bisnis, dan granularitas setiap tabel sebelum melakukan penggabungan data. Lakukan validasi temporal, uji ulang saat pola bisnis berubah, pantau recall dan metrik lain dari waktu ke waktu, serta jadikan model sebagai alat bantu keputusan. Setiap rekomendasi perlu disampaikan bersama keterbatasan data dan asumsi yang digunakan.

### 10. Referensi Materi

Transkrip kegiatan *Informatics Practitioner Lecture Series*, materi “Make Sense of Data with Analytics and AI” oleh narasumber yang tercantum pada transkrip pengguna. Dataset dan hasil implementasi yang menyertai laporan ini dibuat secara sintetis untuk demonstrasi, bukan diambil dari data perusahaan asli.
