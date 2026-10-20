# Forecasting Store Item Demand

Proyek ini memprediksi penjualan mingguan 500 kombinasi toko dan item untuk tahun 2018, lalu menguji apakah model Holt-Winters benar-benar lebih akurat daripada baseline yang sederhana. Hasil prediksi bisa dipakai sebagai dasar perencanaan stok, misalnya saat tim marketing menanyakan apakah suatu barang akan cukup di bulan tertentu.

## Hasil Utama

| Model | WMAPE (500 deret, uji 2017) | MAE (unit/minggu) | Bias |
|---|---|---|---|
| Seasonal naive | 6,2% | 25,5 | -3,7% |
| Holt-Winters | 5,5% | 22,7 | +1,9% |

- Holt-Winters lebih akurat di seluruh 10 toko, dengan selisih sekitar 0,5 sampai 0,9 poin persen.
- Total forecast 2018 sekitar 11,24 juta unit, naik 4,7% dari penjualan 2017 (10,73 juta).
- Di kuartal pertama Holt-Winters justru kalah dari seasonal naive (error 8,6% vs 6,6%).

## Data

[Store Item Demand Forecasting Challenge](https://www.kaggle.com/c/demand-forecasting-kernels-only) dari Kaggle.

- 913.000 baris: `date`, `store`, `item`, `sales`
- 10 toko × 50 item = 500 deret waktu harian
- Periode 1 Januari 2013 sampai 31 Desember 2017
- Tidak ada nilai kosong dan tidak ada baris ganda

File data tidak disertakan di repositori ini. Unduh `train.csv` dari Kaggle lalu letakkan di folder yang sama dengan notebook.

Data ini tampaknya sintetis (beberapa item punya total penjualan yang hampir sama persis) dan tidak memuat informasi stok. Penjualan dianggap sama dengan permintaan.

## Alur Analisis

1. **Load & check data.** Cek ukuran, tipe kolom, nilai kosong, duplikat, dan kelengkapan tanggal.
2. **EDA.** Penjualan punya tiga pola: tren naik tahunan, musiman bulanan (puncak sekitar Juli), dan pola mingguan (puncak di akhir pekan). Skala antar toko dan antar item berbeda cukup jauh.
3. **Train/test split dan baseline.** Data 2013 sampai 2016 untuk latihan, 2017 untuk uji, dibagi berdasarkan waktu supaya tidak ada kebocoran data. Tiga baseline: naive, rata-rata 28 hari, dan seasonal naive (menyalin 364 hari sebelumnya).
4. **Holt-Winters.** Data diubah ke mingguan, lalu dipakai `ExponentialSmoothing` dengan tren aditif dan musiman aditif 52 minggu. Dicoba dulu di satu deret, kemudian di seluruh 500 deret.
5. **Perbandingan.** Metrik MAE, WMAPE, dan bias, dipecah per toko dan per kuartal.
6. **Forecast 2018.** Model dilatih ulang dengan seluruh data 2013 sampai 2017 (260 minggu), lalu meramal 52 minggu ke depan. Hasil disimpan ke `forecast_2018.csv`.

## Metrik

- **MAE**: rata-rata selisih absolut antara aktual dan ramalan, dalam unit.
- **WMAPE**: total selisih absolut dibagi total aktual. Lebih stabil daripada MAPE biasa untuk deret dengan nilai kecil.
- **Bias**: selisih total ramalan terhadap aktual. Negatif berarti cenderung meramal terlalu rendah (risiko kehabisan barang), positif berarti terlalu tinggi (risiko menumpuk).

## Keterbatasan

- Selisih akurasi Holt-Winters terhadap seasonal naive relatif kecil, sekitar 0,7 poin persen.
- Performa Holt-Winters lebih lemah di kuartal pertama. Penyebabnya belum diselidiki.
- Forecast 2018 tidak bisa dicocokkan dengan kenyataan. Kepercayaan pada angka ini bersandar pada hasil uji 2017.
- Horizon 52 minggu lebih sulit daripada kebutuhan operasional yang biasanya 2 sampai 8 minggu.
- Data sintetis, jadi hasil di data nyata bisa berbeda.
- Proyek ini berhenti di permintaan. Safety stock dan reorder point tidak dihitung karena dataset tidak memuat stok maupun lead time.

## Tools

Python, pandas, NumPy, Matplotlib, statsmodels.
