# Movie Recommendations with Spark Collaborative Filtering
Dokumentasi resmi [KNIME](ttps://www.knime.com/blog/movie-recommendations-with-spark-collaborative-filtering "Knime Documentation")

## Overview
1. Knime workflow
![Knime workflow][knimeWorkflow]
[knimeWorkflow]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Knime Workflow"

![Hasil][hasil]
[knimeWorkflow]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Knime Workflow"

## Dokumentasi
### Business Understanding
Collaborative Filtering adalah algoritma berbasis alternating least squares (ALS) yang digunakan untuk membuat sistem rekomendasi untuk user yang didasarkan pada data-data user lain yang memiliki pola yang mirip.

Pada kali ini kita akan mencoba membuat rekomendasi film bagi user film dengan menggunakan Collaborative Filtering.

### Data Understanding

Data yang akan kita gunakan adalah data [ml-20m.zip](../blob/master/LICENSE). Ada dua file utama yang akan kita gunakan dalam workflow knime ini yaitu :

1. [ratings.csv](../blob/master/LICENSE)
Dataset ratings.csv memiliki 20 juta data rating movie dari 130 ribu user circa yang memiliki 4 kolom dengan keterangan :
..- movieID = id dari film
..- userID = id dari user yang merating
..- rating = nilai rating film
..- timestamp = waktu user menginputkan data rating

2. [movies.csv](../blob/master/LICENSE)
Dataset ini memiliki 27 ribu film dari circa yang memiliki 3 kolom dengan keterangan :
..- movieID = id dari film
..- rating = nilai rating film
..- genre = Kategori film

### Data Preparation

Awalnya kita akan memberikan id sebuah user misal : 9999 yang akan kita berikan rekomendasi film. Kita akan membuat tabel list movie yang berisikan sudah ditambahkan data user dan timestamp.

Selanjutnya kita akan meminta user memberikan rating kepada 20 film random yang kita pilih. Dengan ketentuan nilai :
- -1 = film belum ditonton
- 0 - 5 = film jelek - film bagus

Jika user memberikan nilai -1 maka kita tidak akan menggunakan data untuk keperluan training rekomendasi.

### Modelling

Pada tahap ini kita akan membuat modeling yang akan digunakan sebagai dasar rekomendasi. Collaborative Filtering diimplementasikan dalam node Spark Collaborative Filtering Learner. Node menerima input jumlah data, dan fitur-fitur dalam datanya. Dan menghasilkan output model rekomendasi dan prediksi rating untuk semua baris data input. Dataset akan dipecah menjadi data tes dan data training, dimana data training akan digunakan untuk membangun rekomendasi dengan node Spark Collaborative Filtering Learner.

#### Langkah - langkah modelling
1. Create Local Big Data Environment > Membuat semua fungsi local big data environment diantaranya Apache Hive, Apache Spark dan HDFS.
2. CSV to Spark > Membuat spark DataFrame/RDD dari csv yang kita berikan
3. Spark Partitioning > Membagi data menjadi data training dan data test 
4. Table to Spark > Membuat Spark DataFrame/RDD dari table knime. Node ini akan mengubah inputan user menjadi Dataframe/RDD Spark
5. Spark Concatenate > Menggabungkan DataFrame/RDD Spark. Pada node ini inputan user dan partisi data spark akan digabung menjadi satu tabel
6. Spark Collaborative Filtering Learner (MLlib) > Node apache spark untuk melakukan Collaborative Filtering.

Selesai langkah diatas akan dihasilkan model rekomendasi film yang siap diuji cobakan

### Evaluation

Proses selanjutnya kita akan mengevaluasi apakah modeling sudah memberikan hasil yang baik. Data tes yang telah dipartisi akan digunakan untuk mengevaluasi kulaitas dengan node Spark Predictor dan node Spark Numeric Scorer.

#### Langkah-langkah evaluasi
1. Spark Predictor (MLlib) > Memprediksi value berdasarkan model yang telah dibuat pada proses modelling
2. Spark Missing Value > Untuk menghandle jika terdapat prediksi yang error atau membenarkan value data
3. Spark Numeric Scorer > Menghitung score antara value numerik dan value prediksi

Jika value sudah mendekati, maka model sudah benar dan siap untuk digunakan sebagai model rekomendasi

### Deployment

Model yang sudah dievaluasi akan digunakan sebagai prediksi dalam film-film lainya yang belum dirating oleh user. Dengan menggunakan Spark Predictor kita akan memprediksi semua film tersebut dan mengambil 10 film dengan nilai rating tertinggi. 

#### Langkah-langkah deployment
1. Table to Spark > Membuat tabel film yang belum dirating dan menjadikanya dalam bentuk spark RRD
2. Spark Predictor > Menggabungkan model yang telah dibuat untuk memprediksi rating film yang belum dirating
3. Spark to Table > Mengubah kembali bentuk spark RRD menjadi knime table
4. Row filter > Memfilter nilai nan jika terjadi kesalahan prediksi
5. Sorter > Mengurutkan data dari nilai prediksi tinggi ke rendah
6. Row Filter > Mengambil 10 film dengan nilai rekomendasi tertinggi
7. File reader > Mengambil data film dari file movies.csv
8. Joiner > Menggabungkan data movies dan hasil rekomendasi user
9. Display Reccomendation > Menampilkan hasil rekomendasi kepada user
