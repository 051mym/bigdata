# Time Series - electric-production
Source resmi KNIME > https://kni.me/w/W78o4CE7oGCRkf51

## Overview
1. Knime workflow <br>
![Knime Workflow](./dokumentasi/1.PNG)
![Knime Load data](./dokumentasi/2.PNG)
![Knime Extract date time](./dokumentasi/3.PNG)
![Knime Agregation and time series](./dokumentasi/4.PNG)

2. Agregation and Time Series <br>
![Hasil spark CMP](./dokumentasi/5.PNG)
3. Hasil deploy spark <br>
![Hasil deploy1](./dokumentasi/6.PNG)
4. Hasil deploy spark to parquet <br>
![Hasil deploy2](./dokumentasi/7.PNG)

## Dokumentasi
### Business Understanding
Workflow knime diatas mendemonstrasikan penggunaan time series untuk menganalisa rata-rat penggunaan listrik di irlandia. Hasil time series dibagi menjadi 9 yaitu:
1. Total penggunaan listrik
2. Penggunaan listrik per tahun
3. penggunaan listrik per bulan
4. Penggunaan listrik per minggu
5. Penggunaan listrik tiap hari dalam seminggu (tiap senen, tiap selasa dst)
6. Penggunaan listrik per hari
7. Penggunaan listrik per segmen dalam hari (tiap kurun jam tertentu)
8. Penggunaan listrik per hari kerja dan hari libur
9. Penggunaan listrik tiap jam tertentu

### Data Understanding

Data yang akan kita gunakan adalah meters_01_50.csv dengan keterangan sebagai berikut :
![Dataset](./dokumentasi/8.PNG)

Dataset memiliki 44 baris data yang memiliki 3 kolom atribut dengan keterangan :
  - meterID = id meteran listrik
  - enc_datetime = data tanggal yang terenkripsi.
  - reading = nilai meteran pada meteran listrik

### Data Preparation

Tahapan ini kita mengubah dataset ke hive mengubahnya menjadi spark. Dikarenakan data sudah cukup bersih maka kita tidak perlu mencari field null dll. Kita hanya untuk mempersiapkan data untuk tahapan modeling.

![Data preparation](./dokumentasi/9.PNG)

#### Langkah-langkah data preparation
1. File reader > Membaca dataset meter <br>
![Dataset](./dokumentasi/8.PNG)
2. Create Local Big Data Environment > Membuat semua fungsi local big data environment diantaranya Apache Hive, Apache Spark dan HDFS <br>
![Hive Connection](./dokumentasi/10.PNG)
![HDFS Connection](./dokumentasi/11.PNG)
![Spark Context](./dokumentasi/12.PNG)
3. Load data > Meload dataset meter menjadi hive
![Load data](./dokumentasi/2.PNG)
  - DB Table Connector > Membuat tabel database baru dari dataset yang telah dibaca
  ![Table connector](./dokumentasi/13.PNG)
  - DB Loader > Meload data banyak dari database Hive <br>
  ![DB Loader](./dokumentasi/14.PNG)
4. Hive to Spark > Mengimpor hasil dari query Hive inputan menjadi Spark sebagai DataFrame / RDD
![Hasil hive to spark](./dokumentasi/15.PNG)

### Modelling

Pada tahap ini kita akan mengolah DataFrame/RDD Spark hasil preparation dengan macam-macam spark sql node untuk menjabarkan kolom datetime menjadi bagian-bagian sesuai time series yang kita butuhkan.

![Modelling1](./dokumentasi/16.PNG)
![Modelling2](./dokumentasi/17.PNG)
![Modelling3](./dokumentasi/18.PNG)

#### Langkah - langkah modelling
1. Extract date-time atribut > Menjabarkan date time menjadi time series yg kita butuhkan. 
Node ini merupakan kumpulan Spark SQL Query dimana tiap querynya adalah sebagai berikut :

  - Initial datetime conversion
  ![Initial date](./dokumentasi/19.PNG)
    Pada query diatas kolom reading akan di-rename menjadi kw30. Kemudian menambahkan kolom eventDate dan my_time. Berikut penjelasannya:
    - eventDate: hasil dari fungsi date_add (mengambil nilai interval antara tanggal 31-12-2008 dengan tiga digit pertama kolom enc_datetime).
    - my_time: mengambil nilai hh:mm, hh didapatkan dari ((digit ke-4 enc_datetime * 30) / 60 ) % 24. Diambil 2 digit dari hasil tersebut. sedangkan value mm, didapatkan dari ((digit ke-4 enc_datetime * 30) % 60 ) dan diambil 2 digit dari hasil tersebut.
  ![Hasil initial date](./dokumentasi/20.PNG)

  - Extract new datetime features
  ![Extract new datetime features](./dokumentasi/21.PNG)
    Pada query diatas akan dihasilkan 5 kolom baru yaitu:
    - year: mendapatkan nilai tahun dari kolom eventDate
    - month: mendapatkan nilai bulan dari kolom eventDate
    - week: mendapatkan nilai minggu dari kolom eventDate
    - dayOfWeek: mendapatkan nilai nama hari dari kolom eventDate.
    - hour: mendapatkan nilai jam dari value pada kolom my_time.
  ![Hasil Extract new datetime features](./dokumentasi/22.PNG)

  - Assign weekend / weekday
  ![Hasil initial date](./dokumentasi/23.PNG)
    Pada query diatas kita akan mendapatkan 1 kolom baru yaitu:
    - dayClassifier : Didapatkan ketika value dayOfWeek saturday / sunday maka dayClassifiernya `WE` selain itu dayClassifiernya `BD`
  ![Hasil initial date](./dokumentasi/24.PNG)

  - Assign hourly bins (daysegment)
  ![Hasil initial date](./dokumentasi/25.PNG)
    Pada query diatas kita akan mendapatkan 1 kolom baru yaitu:
    - daySegment : Didapatkan dari value hour dengan ketentuan:
      - Nilai hour >= 7 dan hour < 9, maka nilai daySegment 7-9
      - Nilai hour >= 9 dan hour < 13, maka nilai daySegment 9-13
      - Nilai hour >= 13 dan hour < 17, maka nilai daySegment 13-17
      - Nilai hour >= 17 dan hour < 21, maka nilai daySegment 17-21
      - Nilai hour >= 21 atau hour < 7, maka nilai daySegment 21-7
  ![Hasil initial date](./dokumentasi/26.PNG)

2. Aggregation and Time Series > Mendapatkan nilai rata-rata penggunaan listrik tiap time series yang kita inginkan
Dalam node ini dilakukan proses agregasi dengan menggunakan node sparks. Node-node yang digunakan antara lain :
  - Presist Spark DataFrame/RDD > Untuk mencache spark DataFrame agar mempercepat operasi-operasi yang menggunakan dataframe yang sama
  - Spark GroupBy > Melakukan group data sesuai dengan index yang kita inginkan. Selain itu kita juga bisa melakukan agregasi pada node ini
  - Spark Pivot > Melakukan pivoting pada Spark DataFrame / RDD yang diberikan menggunakan jumlah kolom yang dipilih untuk pengelompokan dan satu kolom untuk pivoting
  - Spark Coloumn Rename > Mengubah nama suatu kolom dari table
  - Spark Joiner > Menjoin 2 spark DataFrame seperti join pada database

![Knime Agregation and time series](./dokumentasi/4.PNG)

Pada workflow diatas terdapat proses-proses :

  - Mencari total penggunaan listrik dengan cara:
    - SUM kolom kw30 dan groupby meterid
    ![Total listrik1](./dokumentasi/27.PNG)
    ![Total listrik2](./dokumentasi/28.PNG)
    - Rename kolom hasil sum menjadi totalKW
    ![Rename kolom](./dokumentasi/29.PNG) <br>
    Hasilnya menjadi <br>
    ![Hasil total](./dokumentasi/30.PNG)

  - Mencari rata-rata pengunaan listrik per tahun dengan cara:
    - SUM kolom kw30 dan groupby meterid, year
    ![Rata tahun1](./dokumentasi/31.PNG)
    ![Rata tahun2](./dokumentasi/32.PNG)
    - MEAN kolom SUM(kw30) dan groupby meterid
    ![Rata tahun3](./dokumentasi/33.PNG)
    ![Rata tahun4](./dokumentasi/34.PNG)
    - Rename kolom hasil MEAN menjadi avgYearlyKW
    ![Rata tahun5](./dokumentasi/35.PNG) <br>
    Hasilnya menjadi <br>
    ![Rata tahun6](./dokumentasi/36.PNG)

  - Mencari rata-rata pengunaan listrik per bulan dengan cara:
    - SUM kolom kw30 dan groupby meterid, year, month
    ![Rata bulan1](./dokumentasi/37.PNG)
    ![Rata bulan2](./dokumentasi/38.PNG)
    - MEAN kolom SUM(kw30) dan groupby meterid
    ![Rata bulan3](./dokumentasi/39.PNG)
    ![Rata bulan4](./dokumentasi/40.PNG)
    - Rename kolom hasil MEAN menjadi avgMonthlyKW
    ![Rata bulan5](./dokumentasi/41.PNG) <br>
    Hasilnya menjadi <br>
    ![Rata bulan6](./dokumentasi/42.PNG)

  - Mencari rata-rata pengunaan listrik per minggu dengan cara:
    - SUM kolom kw30 dan groupby meterid, year, week
    ![Rata minggu1](./dokumentasi/43.PNG)
    ![Rata minggu2](./dokumentasi/44.PNG)
    - MEAN kolom SUM(kw30) dan groupby meterid
    ![Rata minggu3](./dokumentasi/45.PNG)
    ![Rata minggu4](./dokumentasi/46.PNG)
    - Rename kolom hasil MEAN menjadi avgWeeklyKW
    ![Rata minggu5](./dokumentasi/47.PNG) <br>
    Hasilnya menjadi <br>
    ![Rata minggu6](./dokumentasi/48.PNG)

  - Mencari rata-rata pengunaan listrik per dayofweek dengan cara:
    - SUM kolom kw30 dan groupby meterid, year, week, dayOfWeek
    ![Rata dayminggu1](./dokumentasi/49.PNG)
    ![Rata dayminggu2](./dokumentasi/50.PNG)
    - MEAN kolom SUM(kw30) dengan pivot dayOfWeek dan groupby meterid
    ![Rata dayminggu3](./dokumentasi/51.PNG)
    ![Rata dayminggu4](./dokumentasi/52.PNG)
    ![Rata dayminggu4](./dokumentasi/53.PNG)
    - Rename kolom hasil ditas menjadi avg[day].
    ![Rata dayminggu5](./dokumentasi/54.PNG) <br>
    Hasilnya menjadi <br>
    ![Rata dayminggu6](./dokumentasi/55.PNG)

  - Penggabungan semua DataFrame/RDDs dengan menggunakan Spark joiner. Karena spark joiner hanya bisa menggabungkan 2 DataFrame/RDDs maka hasil workflow penggabungan menjadi seperti dibawah.
  ![Spark joiner](./dokumentasi/56.PNG) <br>
  Kita menggabungkan dengan inner join, dengan meterid sebagai key joinya. Serta menginclude semua kolomnya
  ![Spark joiner](./dokumentasi/57.PNG)
  ![Spark joiner](./dokumentasi/58.PNG)

3. Spark SQL Query > Melakukan sql query pada spark
Disini kita akan menghitung persentase dari penggunaan listrik per hari, dan pada hari saat periode jam tertentu. Dengan setinggan seperti dibawah
![Compute %dayly1](./dokumentasi/59.PNG)
Maka hasilnya menjadi
![Compute %dayly2](./dokumentasi/60.PNG)

### Evaluation

Proses selanjutnya kita akan mengevaluasi hasil time series kita. Kita menggunakan PCA, K-means, Scatter plot untuk melakukan visualisasi dari data yang telah dlklusterkan.
Hasil visualisasinya

![Evaluation1](./dokumentasi/61.PNG)
![Evaluation2](./dokumentasi/62.PNG)
![Evaluation3](./dokumentasi/63.PNG)

### Deployment

![Deploy1](./dokumentasi/64.PNG) <br>
Tahapan ini kita akan mendeploy hasil spark DataFrame dari proses wvaluation menjadi tabel hive dan parquet file menggunakan node dibawah

- Spark to Hive > Mengkonversi Spark DataFrame menjadi tabel hive
Konfigurasi
![Deploy2](./dokumentasi/65.PNG) <br>
Hasil <br>
![Hasil deploy1](./dokumentasi/6.PNG)
- Spatk to Parquet > Mengkonversi Spark DataFame menjadi parquet file
Konfigurasi
![Hasil deploy1](./dokumentasi/66.PNG) <br>
Hasil <br>
![Hasil deploy2](./dokumentasi/7.PNG)

