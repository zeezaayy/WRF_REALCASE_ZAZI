# 🌪️ Simulasi Numerik Siklon Tropis Dahlia (2017) Menggunakan Model WRF-ARW v4.3

![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-blue)
![Docker](https://img.shields.io/badge/Docker-dtcenter%2Fwps__wrf-2496ED?logo=docker)
![WRF](https://img.shields.io/badge/WRF-v4.3-orange)
![Data](https://img.shields.io/badge/Data-ERA5-green)
![Status](https://img.shields.io/badge/Status-Completed-success)

---

## 📖 Deskripsi Repository

Repository ini mendokumentasikan proses **simulasi numerik Siklon Tropis Dahlia (2017)** menggunakan **Weather Research and Forecasting (WRF-ARW) versi 4.3** dengan data **ERA5 Reanalysis** sebagai kondisi awal (*initial condition*) dan kondisi batas (*boundary condition*).

Dokumentasi mencakup seluruh tahapan mulai dari persiapan lingkungan kerja menggunakan Docker, konfigurasi domain, preprocessing menggunakan **WPS**, pelaksanaan simulasi **WRF**, hingga visualisasi dan validasi hasil terhadap data observasi **MSWEP**.

Repository ini dibuat sebagai dokumentasi selama kegiatan magang di **Badan Riset dan Inovasi Nasional (BRIN)** sekaligus sebagai referensi bagi pengguna yang ingin melakukan simulasi kasus nyata menggunakan WRF.

---

# 📑 Daftar Isi

- [Latar Belakang](#-latar-belakang)
- [Workflow Simulasi](#-workflow-simulasi)
- [Persyaratan Sistem](#-persyaratan-sistem)
- [Persiapan Docker](#-persiapan-docker)
- [Konfigurasi Domain](#-konfigurasi-domain)
- [Data Input](#-data-input)
- [Menjalankan WPS](#-menjalankan-wps)
- [Menjalankan WRF](#-menjalankan-wrf)
- [Hasil Simulasi](#-hasil-simulasi)
- [Validasi Hasil](#-validasi-hasil)
- [Troubleshooting](#-troubleshooting)
- [Referensi](#-referensi)

---

# 🌎 Latar Belakang

## Weather Research and Forecasting (WRF)

**Weather Research and Forecasting (WRF)** merupakan model numerik atmosfer mesoskal yang banyak digunakan untuk penelitian maupun prakiraan cuaca operasional. Model ini mampu mensimulasikan berbagai fenomena atmosfer, mulai dari sistem cuaca lokal hingga fenomena berskala besar seperti siklon tropis.

Pada repository ini digunakan **WRF-ARW v4.3** dengan data **ERA5 Reanalysis** sebagai kondisi awal dan kondisi batas simulasi.

## Siklon Tropis Dahlia

Siklon Tropis Dahlia merupakan siklon tropis yang terbentuk di Samudra Hindia pada akhir November 2017, tidak lama setelah melemahnya Siklon Tropis Cempaka. Meskipun pusat siklon berada di luar wilayah Indonesia, sistem ini memberikan pengaruh yang cukup signifikan terhadap kondisi cuaca di berbagai wilayah, khususnya Pulau Jawa, Sumatera bagian selatan, serta perairan di Samudra Hindia.

Keberadaan Siklon Tropis Dahlia meningkatkan suplai uap air dan memperkuat konvergensi massa udara sehingga memicu **curah hujan lebat**, **angin kencang**, dan **gelombang laut tinggi** di sejumlah wilayah Indonesia. Dampak hidrometeorologi yang dilaporkan antara lain meliputi **banjir**, **tanah longsor**, **genangan**, pohon tumbang akibat angin kencang, serta terganggunya aktivitas pelayaran karena tinggi gelombang yang meningkat.

Penelitian ini bertujuan merekonstruksi kejadian tersebut menggunakan model WRF sehingga hasil simulasi dapat dibandingkan dengan data observasi satelit.

> 📌 **Referensi BMKG**
>
> - https://www.bmkg.go.id/siaran-pers/cempaka-meluruh-dahlia-lahir-waspada-bencana-hidrometeorologi-menghadang
> - https://bbmkg3.bmkg.go.id/bbmkg3_pdf_files/11122017060450.pdf

---

# 🔄 Workflow Simulasi

```text
ERA5 Reanalysis
        │
        ▼
WRF Domain Wizard
        │
        ▼
WPS
 ├── geogrid.exe
 ├── ungrib.exe
 └── metgrid.exe
        │
        ▼
real.exe
        │
        ▼
wrf.exe
        │
        ▼
wrfout
        │
        ▼
Visualisasi
        │
        ▼
Validasi (MSWEP)
```

---

# 💻 Persyaratan Sistem

## Software yang Digunakan

| Software | Versi |
|----------|--------|
| Docker Desktop | Latest |
| Docker Image | dtcenter/wps_wrf |
| WRF | v4.3 |
| WPS | v4.3 |
| Python | 3.x |
| Google Colab | Latest |
| QGIS | 3.x (Opsional) |

## Hardware yang Disarankan

| Komponen | Minimum |
|----------|---------|
| Processor | Intel Core i5 / Ryzen 5 |
| RAM | 16 GB |
| Storage | ≥100 GB |
| Sistem Operasi | Windows 10 / Windows 11 |

---

# 🐳 Persiapan Docker

## 1. Pull Docker Image

```bash
docker pull dtcenter/wps_wrf
```

```bash
docker images
```

## 2. Menjalankan Docker Container

```bash
docker ps
docker exec -it <container_id> /bin/bash
```

## 3. Direktori Kerja

```text
/comsoftware/wrf
```

```text
comsoftware/
└── wrf/
    ├── WPS-4.3/
    ├── WRF-4.3/
    ├── WPS_GEOG/
    └── data_era5/
```

---

# 🌍 Konfigurasi Domain

| Domain | Resolusi | Grid | Cakupan |
|---------|---------|---------|----------------|
| D01 | 27 km | 170 × 170 | Regional |
| D02 | 9 km | 190 × 175 | Jawa Bagian Tengah |
| D03 | 3 km | 250 × 220 | Wilayah Studi |

## Konfigurasi Domain

<p align="center">
<img src="docs/domain.png" width="900">
</p>

> 📌 **Gambar di atas menunjukkan konfigurasi tiga domain simulasi yang dibuat menggunakan WRF Domain Wizard.**

# 📥 Data Input

Simulasi menggunakan data **ERA5 Reanalysis** sebagai kondisi awal (*Initial Condition*) dan kondisi batas (*Boundary Condition*). Seluruh data diunduh dengan resolusi temporal 1 jam untuk periode simulasi **26–29 November 2017**.

---

## Dataset yang Digunakan

| Jenis Data | Produk | Keterangan |
|------------|--------|------------|
| Pressure Levels | ERA5 Pressure Level | Kondisi atmosfer tiga dimensi |
| Surface Levels | ERA5 Single Level | Variabel permukaan seperti T2, U10, V10, dan PSFC |
| Geographical Data | WPS_GEOG | Data topografi dan penggunaan lahan |

---

## Struktur Folder

```text
/comsoftware/wrf
├── data_era5
│   ├── ERA5_PL/
│   └── ERA5_SFC/
├── WPS-4.3/
└── WPS_GEOG/
```

---

# 🗺️ Data Geografis (WPS_GEOG)

Sebelum menjalankan WPS, pastikan dataset geografis telah tersedia pada direktori **WPS_GEOG**.

Contoh pengecekan:

```bash
cd /comsoftware/wrf/WPS_GEOG

ls
```

Apabila data belum tersedia, ekstrak dataset ke direktori tersebut.

---

# ⚙️ Menjalankan WPS

Masuk ke direktori WPS.

```bash
cd /comsoftware/wrf/WPS-4.3
```

Pastikan file berikut tersedia.

```text
geogrid.exe
ungrib.exe
metgrid.exe
link_grib.csh
```

---

# 📝 Konfigurasi namelist.wps

Sesuaikan parameter simulasi pada file `namelist.wps`.

```bash
nano namelist.wps
```

Contoh konfigurasi utama:

```text
&share
 wrf_core = 'ARW',
 max_dom = 3,
 start_date = '2017-11-26_00:00:00',
 end_date   = '2017-11-29_00:00:00',
 interval_seconds = 3600,
/

&geogrid
 parent_id         =   1, 1, 2,
 parent_grid_ratio =   1, 3, 3,
 dx = 27000,
 dy = 27000,
 ref_lat = -8.5,
 ref_lon = 109.5,
 geog_data_path = '/comsoftware/wrf/WPS_GEOG'
/
```

> 📌 Gunakan konfigurasi `namelist.wps` lengkap sesuai proyek simulasi Anda.

---

# 🌍 Menjalankan geogrid

Program **geogrid.exe** menghasilkan data geografis untuk setiap domain simulasi.

```bash
./geogrid.exe
```

Periksa prosesnya melalui log.

```bash
cat geogrid.log
```

Output yang diharapkan:

```text
Successful completion of program geogrid.exe
```

---

# 🌦️ Menjalankan ungrib

Hubungkan terlebih dahulu data ERA5 dengan WPS.

```bash
./link_grib.csh /comsoftware/wrf/data_era5/ERA5_PL/*
./link_grib.csh /comsoftware/wrf/data_era5/ERA5_SFC/*
```

Pastikan Vtable telah dipilih.

```bash
ln -sf ungrib/Variable_Tables/Vtable.ERA5 Vtable
```

Kemudian jalankan:

```bash
./ungrib.exe
```

Periksa log:

```bash
cat ungrib.log
```

Output yang diharapkan:

```text
Successful completion of program ungrib.exe
```

---

# 🛰️ Menjalankan metgrid

Tahap terakhir preprocessing adalah menggabungkan data meteorologi dengan data geografis.

```bash
./metgrid.exe
```

Periksa hasilnya.

```bash
cat metgrid.log
```

Output yang diharapkan:

```text
Successful completion of program metgrid.exe
```

Setelah selesai akan terbentuk file:

```text
met_em.d01.*
met_em.d02.*
met_em.d03.*
```

File inilah yang selanjutnya digunakan pada proses **real.exe** di WRF.

# 🚀 Menjalankan WRF

Setelah seluruh proses **WPS** selesai dan file `met_em.d0*.nc` berhasil dibuat, tahap berikutnya adalah menjalankan model **WRF-ARW**.

---

# 📂 Struktur Direktori WRF

Masuk ke direktori WRF.

```bash
cd /comsoftware/wrf/WRF-4.3/run
```

Pastikan file berikut tersedia.

```text
real.exe
wrf.exe
namelist.input
met_em.d01.*
met_em.d02.*
met_em.d03.*
```

---

# 📝 Konfigurasi namelist.input

Buka file konfigurasi.

```bash
nano namelist.input
```

Sesuaikan periode simulasi, jumlah domain, interval output, serta parameter fisika sesuai kebutuhan penelitian.

> 📌 Pada repository ini digunakan konfigurasi `namelist.input` untuk simulasi Siklon Tropis Dahlia periode November 2017. Salin konfigurasi lengkap yang digunakan pada proyek ke bagian ini.

---

# ▶️ Menjalankan real.exe

Program `real.exe` akan menghasilkan file kondisi awal (`wrfinput`) dan kondisi batas (`wrfbdy`) berdasarkan output WPS.

```bash
./real.exe
```

Apabila ingin menyimpan log:

```bash
./real.exe 2>&1 | tee LOGS/run_real.log
```

Periksa log:

```bash
tail -50 rsl.error.0000
```

Output yang diharapkan:

```text
SUCCESS COMPLETE REAL_EM INIT
```

File yang dihasilkan:

```text
wrfinput_d01
wrfinput_d02
wrfinput_d03
wrfbdy_d01
```

---

# 🌤️ Menjalankan wrf.exe

Setelah `real.exe` selesai tanpa error, jalankan simulasi utama.

```bash
./wrf.exe
```

Atau simpan log proses:

```bash
./wrf.exe 2>&1 | tee LOGS/run_wrf.log
```

Periksa proses simulasi:

```bash
tail -50 rsl.error.0000
```

Output akhir yang diharapkan:

```text
SUCCESS COMPLETE WRF
```

---

# 📁 Output Simulasi

Apabila simulasi berhasil, akan terbentuk file output berikut.

```text
wrfout_d01_2017-11-26_00:00:00
wrfout_d02_2017-11-26_00:00:00
wrfout_d03_2017-11-26_00:00:00
```

File NetCDF (`*.nc`) tersebut digunakan untuk proses visualisasi dan analisis lebih lanjut menggunakan Python, Google Colab, maupun QGIS.

---

# ⚙️ Skema Fisika

Skema fisika yang digunakan pada simulasi disesuaikan dengan konfigurasi penelitian dan didefinisikan pada `namelist.input`.

| Komponen | Keterangan |
|----------|------------|
| Microphysics | Sesuai konfigurasi penelitian |
| Longwave Radiation | Sesuai konfigurasi penelitian |
| Shortwave Radiation | Sesuai konfigurasi penelitian |
| Planetary Boundary Layer | Sesuai konfigurasi penelitian |
| Surface Layer | Sesuai konfigurasi penelitian |
| Land Surface Model | Noah LSM |
| Cumulus Parameterization | Aktif pada domain beresolusi kasar |

> 📌 Isi tabel di atas dapat diperbarui sesuai parameter fisika yang benar-benar digunakan pada simulasi.

---

# ✅ Pemeriksaan Output

Beberapa file penting yang dapat diperiksa setelah simulasi selesai:

```bash
ls -lh wrfout_*
```

Melihat informasi variabel:

```bash
ncdump -h wrfout_d03_2017-11-26_00:00:00
```

Melihat ukuran file:

```bash
du -sh wrfout_*
```

Tahap berikutnya adalah melakukan visualisasi hasil simulasi dan validasi terhadap data observasi.

# 📊 Hasil Simulasi

Bagian ini menampilkan hasil simulasi WRF-ARW untuk Siklon Tropis Dahlia (2017). Seluruh visualisasi dibuat dari file `wrfout` menggunakan Python pada Google Colab.

---

# 🌡️ Temperatur Permukaan (T2)

<p align="center">
<img src="temperature permukaan.png" width="900">
</p>

Variabel **T2** digunakan untuk merepresentasikan temperatur udara pada ketinggian 2 meter di atas permukaan.

🎥 **Animasi MP4**

https://drive.google.com/file/d/15vKI69PXFYYcw4ojZRbz9CBlmvMUxZ6Z/view?usp=drive_link

---

# 🌧️ Curah Hujan

<p align="center">
<img src="curah hujan.png" width="900">
</p>

Curah hujan dihitung dari akumulasi **RAINC** dan **RAINNC**, kemudian dikonversi menjadi curah hujan per jam.

🎥 **Animasi MP4**

https://drive.google.com/file/d/1FRwf8efv6ijLkFeb3K2WRhwleZ9vJNFD/view?usp=drive_link

---

# 🌬️ Kecepatan dan Arah Angin

<p align="center">
<img src="plot angin.png" width="900">
</p>

Visualisasi memperlihatkan distribusi kecepatan angin beserta arah aliran angin selama simulasi berlangsung.

🎥 **Animasi MP4**

https://drive.google.com/file/d/1Pf5zkxmXw_N2pdeHeVphrIcJ_nNvC4EO/view?usp=drive_link

---

# 🌀 Pergerakan Siklon Tropis Dahlia

| Hasil Simulasi WRF | Referensi BMKG |
|:------------------:|:--------------:|
| <img src="siklon.png" width="420"> | <img src="siklon dahlia.jpg" width="420"> |

Perbandingan lintasan pusat siklon hasil simulasi dengan referensi yang dipublikasikan oleh BMKG.

🎥 **Animasi MP4**

https://drive.google.com/file/d/1xQRibaGs3Ot5KLM9m1vEj2YwOaxjMSCF/view?usp=drive_link

---

# 📈 Validasi Hasil

## Rerata Curah Hujan

<p align="center">
<img src="rerata hujan.png" width="850">
</p>

Rerata curah hujan digunakan untuk membandingkan kecenderungan temporal antara hasil simulasi WRF dan data observasi.

---

## RMSE

<p align="center">
<img src="rmse stasiun pengamatan.png" width="850">
</p>

Nilai **Root Mean Square Error (RMSE)** digunakan untuk mengukur besarnya penyimpangan hasil simulasi terhadap data observasi pada setiap stasiun pengamatan.

---

## Scatter Plot

<p align="center">
<img src="scatter plot.png" width="800">
</p>

Scatter plot digunakan untuk melihat hubungan antara nilai hasil simulasi WRF dan data observasi.

---

# ⚠️ Troubleshooting

| Permasalahan | Penyebab | Solusi |
|--------------|----------|--------|
| `geogrid.exe` gagal | Path WPS_GEOG salah | Periksa `geog_data_path` pada `namelist.wps`. |
| `ungrib.exe` gagal | Vtable tidak sesuai | Gunakan `Vtable.ERA5`. |
| `metgrid.exe` gagal | File intermediate tidak ditemukan | Jalankan `ungrib.exe` terlebih dahulu. |
| `real.exe` berhenti | `met_em` tidak lengkap | Pastikan seluruh file `met_em.d0*.nc` tersedia. |
| `wrf.exe` berhenti | Konfigurasi domain atau input tidak sesuai | Periksa `rsl.error.0000` untuk mengetahui sumber error. |

---

# 📚 Referensi

1. Skamarock, W. C., et al. (2019). *A Description of the Advanced Research WRF Model Version 4.*
2. ECMWF. *ERA5 Reanalysis Dataset.*
3. Beck, H. E., et al. (2019). *MSWEP Version 2.*
4. BMKG. **Cempaka Meluruh, Siklon Tropis Dahlia Lahir.**
5. BMKG Regional III. **Laporan Siklon Tropis Dahlia Tahun 2017.**

---

## 🎥 Google Drive Visualisasi

| Visualisasi | Link |
|-------------|------|
| Temperatur Permukaan | https://drive.google.com/file/d/15vKI69PXFYYcw4ojZRbz9CBlmvMUxZ6Z/view?usp=drive_link |
| Curah Hujan | https://drive.google.com/file/d/1FRwf8efv6ijLkFeb3K2WRhwleZ9vJNFD/view?usp=drive_link |
| Kecepatan & Arah Angin | https://drive.google.com/file/d/1Pf5zkxmXw_N2pdeHeVphrIcJ_nNvC4EO/view?usp=drive_link |
| Pergerakan Siklon | https://drive.google.com/file/d/1xQRibaGs3Ot5KLM9m1vEj2YwOaxjMSCF/view?usp=drive_link |
