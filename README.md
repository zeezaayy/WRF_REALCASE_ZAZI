# Rekonstruksi Numerik Siklon Tropis Dahlia dengan WRF-ARW

> Catatan riset pribadi — simulasi ulang kejadian Siklon Tropis Dahlia (29 November – 2 Desember 2017) memakai WRF-ARW v4.3, dijalankan di dalam container Docker `dtcenter/wps_wrf`.

**Stack:** WPS 4.3 · WRF-ARW 4.3 · ERA5 Reanalysis · Docker · WRF Domain Wizard

---

## Ringkasan Isi

- [Latar Belakang Kasus](#latar-belakang-kasus)
- [Persiapan Lingkungan Kerja](#persiapan-lingkungan-kerja)
- [Desain Domain Simulasi](#desain-domain-simulasi)
- [Data Input](#data-input)
- [Menjalankan WPS](#menjalankan-wps)
- [Menjalankan WRF](#menjalankan-wrf)
- [Hasil Simulasi (Visualisasi)](#hasil-simulasi-visualisasi)
- [Evaluasi Statistik (WRF vs MSWEP)](#evaluasi-statistik-wrf-vs-mswep)
- [Troubleshooting](#troubleshooting)
- [Struktur Folder](#struktur-folder)
- [Referensi](#referensi)

---

## Latar Belakang Kasus

Siklon Tropis **Dahlia** lahir di Samudra Hindia sebelah selatan barat daya Bengkulu pada **29 November 2017**, tak lama setelah Siklon Tropis Cempaka melemah dan meninggalkan wilayah Indonesia. Berbeda dari Cempaka yang tumbuh dekat pantai selatan Jawa, Dahlia terbentuk lebih ke arah barat dan sepanjang hidupnya bergerak menjauh dari daratan Indonesia (ke arah tenggara/timur tenggara), hingga akhirnya melemah menjadi Depresi Tropis pada **2–3 Desember 2017**.

Walau bergerak menjauh, sirkulasi tepi (*outer band*) Dahlia tetap memicu:

- Hujan sedang–lebat di pesisir barat Bengkulu, Lampung, Banten, DKI Jakarta, dan Jawa Barat
- Angin kencang > 20 knot di pesisir barat Sumatera bagian selatan
- Gelombang laut 2,5–6,0 m di perairan barat Sumatera hingga Selat Sunda

Simulasi ini disusun untuk melihat bagaimana model menangkap dampak tidak langsung tersebut, khususnya penjalaran sabuk hujan tepi saat sistem menjauhi daratan.

## Persiapan Lingkungan Kerja

Container dijalankan lewat **Docker Desktop** tanpa volume mounting, jadi transfer file keluar-masuk dilakukan lewat `docker cp`.

```bash
docker pull dtcenter/wps_wrf
```

Setelah image ditarik, jalankan via Docker Desktop (tab *Images* → **Run**), lalu masuk ke terminalnya lewat tab **Exec** pada container yang aktif. Direktori kerja default:

```bash
comsoftware/wrf/
```

berisi folder `WPS_GEOG`, `WPS-4.3`, dan `WRF-4.3` yang sudah terkompilasi.

Kalau lebih suka terminal biasa:

```bash
docker ps
docker exec -it <container_id> /bin/bash
```

## Desain Domain Simulasi

Domain ditentukan pakai **WRF Domain Wizard**, dipusatkan di sekitar Laut Jawa bagian barat (ref_lat -6.252, ref_lon 115.525) supaya domain terluar tetap mencakup posisi kelahiran Dahlia di barat daya Bengkulu sekaligus wilayah yang terdampak di selatan Jawa.

| Domain | Resolusi | Grid (e_we × e_sn) | Cakupan |
| :--- | :--- | :--- | :--- |
| **D01** | 30 km | 165 × 93 | Regional — Samudra Hindia, Sumatera–Jawa, Kalimantan bagian selatan |
| **D02** | 10 km | 202 × 127 | Bengkulu, Lampung, Banten, Jawa bagian barat |
| **D03** | 3,33 km | 250 × 163 | Jawa bagian barat hingga tengah dan perairan selatan Jawa |

Rasio pembesaran antar-domain konsisten **1:3** (`parent_grid_ratio = 1, 3, 3`), dengan posisi domain anak:

```ini
i_parent_start = 1, 33, 36
j_parent_start = 1, 11, 63
```

> Sudut domain D01 hasil Domain Wizard: N (6.190, 93.303) – (6.190, 137.747), S (-18.406, 93.303) – (-18.406, 137.747).

## Data Input

### Data Statis (WPS_GEOG)

```bash
wget -c https://www2.mmm.ucar.edu/wrf/src/wps_files/geog_complete.tar.gz
tar -xzvf geog_complete.tar.gz -C ./WPS_GEOG --strip-components=1
docker cp ./WPS_GEOG <container_id>:comsoftware/wrf/WPS_GEOG
```

### Data Meteorologi (ERA5)

Diunduh dari **Copernicus Climate Data Store**, dipisah menjadi dua berkas untuk rentang **29 November – 2 Desember 2017**:

- **Pressure levels** → variabel 3D (geopotential, temperature, u/v wind, RH)
- **Single/surface levels** → variabel 2D (SST, soil temperature/moisture, mslp, 2m temp, 10m wind)

```bash
docker cp ./data_era5 <container_id>:comsoftware/wrf/WPS-4.3/data_era5
```

## Menjalankan WPS

```bash
cd comsoftware/wrf/WPS-4.3
```

### `namelist.wps` yang dipakai

```ini
&share
 wrf_core             = 'ARW'
 max_dom              = 3
 start_date           = '2017-11-29_00:00:00','2017-11-29_00:00:00','2017-11-29_00:00:00'
 end_date             = '2017-12-02_23:00:00','2017-12-02_23:00:00','2017-12-02_23:00:00'
 interval_seconds     = 3600
 io_form_geogrid      = 2
 debug_level          = 0
/
&geogrid
 parent_id            = 1, 1, 2
 parent_grid_ratio    = 1, 3, 3
 i_parent_start       = 1, 33, 36
 j_parent_start       = 1, 11, 63
 e_we                 = 165, 202, 250
 e_sn                 = 93, 127, 163
 geog_data_res        = 'default', 'default', 'default'
 dx                   = 30000
 dy                   = 30000
 map_proj             = 'mercator'
 ref_lat              = -6.252
 ref_lon              = 115.525
 truelat1             = -5.309
 pole_lat             = 90
 pole_lon             = 0
 geog_data_path       = '/comsoftware/wrf/WPS_GEOG'
 opt_geogrid_tbl_path = './geogrid/'
/
&ungrib
 out_format           = 'WPS'
 prefix               = 'ERA5_sfc'
/
&metgrid
 fg_name              = 'ERA5_p','ERA5_sfc'
 io_form_metgrid      = 2
 opt_metgrid_tbl_path = './metgrid'
/
```

### Geogrid

```bash
./geogrid.exe >& log.geogrid
tail -n 5 log.geogrid
ls geo_em.d0*.nc
```

### Ungrib — dua kali jalan, terpisah pressure level & surface

ERA5 pressure level dan surface **wajib diproses terpisah** lewat `ungrib.exe`, masing-masing dengan `prefix` sendiri, karena keduanya punya struktur variabel berbeda dan `metgrid` butuh dua set file intermediate (`ERA5_p:*` dan `ERA5_sfc:*`) sekaligus.

**Tahap pressure level:**

```bash
./link_grib.csh /path/to/ERA5/PL/*
./ungrib.exe
```

Lalu hapus symbolic link GRIB agar tidak tercampur dengan tahap berikutnya:

```bash
rm -f GRIBFILE.*
```

**Ubah prefix di `&ungrib` menjadi:**

```ini
prefix = 'ERA5_sfc'
```

**Tahap surface, jalankan ulang:**

```bash
./link_grib.csh /path/to/ERA5/SFC/*
./ungrib.exe
```

Pastikan file `ERA5_p:YYYY-MM-DD_HH` **dan** `ERA5_sfc:YYYY-MM-DD_HH` sudah tersedia sebelum lanjut ke `metgrid.exe`.

### Metgrid

```bash
./metgrid.exe >& log.metgrid
tail -n 5 log.metgrid
ls met_em.d0*.nc
```

## Menjalankan WRF

```bash
cd comsoftware/wrf/WRF-4.3/test/em_real
ln -sf ../../../WPS-4.3/met_em.d0* .
```

### `namelist.input` yang dipakai

```ini
&time_control
 run_days                            = 0,
 run_hours                           = 0,
 run_minutes                         = 0,
 run_seconds                         = 0,
 start_year                          = 2017, 2017, 2017,
 start_month                         = 11,   11,   11,
 start_day                           = 29,   29,   29,
 start_hour                          = 08,   08,   08,
 end_year                            = 2017, 2017, 2017,
 end_month                           = 12,   12,   12,
 end_day                             = 02,   02,   02,
 end_hour                            = 23,   23,   23,
 interval_seconds                    = 3600,
 input_from_file                     = .true., .true., .true.,
 history_interval                    = 60, 60, 60,
 frames_per_outfile                  = 1, 1, 1,
 restart                             = .true.,
 restart_interval                    = 60,
 io_form_history                     = 2,
 io_form_restart                     = 2,
 io_form_input                       = 2,
 io_form_boundary                    = 2,
 auxinput4_inname                    = "wrflowinp_d<domain>",
 auxinput4_interval                  = 60,60,60,
 io_form_auxinput4                   = 2,
/

&namelist_quilt
 nio_tasks_per_group = 0,
 nio_groups = 1,
 /

&domains
 time_step                           = 180,
 time_step_fract_num                 = 0,
 time_step_fract_den                 = 1,
 max_dom                             = 3,
 e_we                                = 165,202,250,
 e_sn                                = 93,127,163,
 e_vert                              = 60,60,60,
 p_top_requested                     = 10000,
 num_metgrid_levels                  = 38,
 num_metgrid_soil_levels             = 4,
 dx                                  = 30000,
 dy                                  = 30000,
 grid_id                             = 1,2,3,
 parent_id                           = 1,1,2,
 i_parent_start                      = 1,33,36,
 j_parent_start                      = 1,11,63,
 parent_grid_ratio                   = 1,3,3,
 parent_time_step_ratio              = 1,3,3,
 feedback                            = 1,
 smooth_option                       = 0,
/

&physics
 mp_physics                          = 2,     2,     2,     ! Purdue-Lin
 ra_lw_physics                       = 4,     4,     4,     ! RRTMG
 ra_sw_physics                       = 4,     4,     4,     ! RRTMG
 radt                                = 27,    9,     3,
 sf_sfclay_physics                   = 1,     1,     1,     ! Revised MM5
 sf_surface_physics                  = 2,     2,     2,     ! Noah LSM
 bl_pbl_physics                      = 1,     1,     1,     ! YSU
 bldt                                = 0,     0,     0,
 cu_physics                          = 2,     2,     0,     ! BMJ (D01, D02), tanpa skema kumulus di D03
 cudt                                = 5,     5,     0,
 isfflx                              = 1,
 ifsnow                              = 1,
 icloud                              = 1,
 surface_input_source                = 3,
 num_soil_layers                     = 4,
 sst_update                          = 1,     ! SST harian diperbarui sepanjang simulasi
 sf_urban_physics                    = 0,     0,     0,
 /

&dynamics
 w_damping                           = 1,
 diff_opt                            = 1,      1,      1,
 km_opt                              = 4,      4,      4,
 diff_6th_opt                        = 0,      0,      0,
 diff_6th_factor                     = 0.12,   0.12,   0.12,
 base_temp                           = 290.,
 damp_opt                            = 3,
 zdamp                               = 5000.,  5000.,  5000.,
 dampcoef                            = 0.2,    0.2,    0.2,
 khdif                               = 0,      0,      0,
 kvdif                               = 0,      0,      0,
 non_hydrostatic                     = .true., .true., .true.,
 moist_adv_opt                       = 1,      1,      1,
 scalar_adv_opt                      = 1,      1,      1,
 gwd_opt                             = 1,      0,      0,
 /

&bdy_control
 spec_bdy_width                      = 5,
 specified                           = .true., .false.,.false.,
 nested                              = .false.,.true., .true.,
 /
```

**Kenapa `sst_update = 1`?** Dahlia menghabiskan sebagian besar siklus hidupnya di atas laut sambil menjauhi daratan. Tanpa update SST harian, model memakai SST tunggal dari kondisi awal sepanjang simulasi — berisiko membuat intensitas dan fase pelemahan siklon meleset dari observasi.

### Eksekusi

```bash
./real.exe >& log.real
tail -n 5 log.real
ls wrfbdy_d01 wrfinput_d0* wrflowinp_d0*
```

```bash
mpirun -np 8 ./wrf.exe >& log.wrf
```

## Hasil Simulasi (Visualisasi)

Berikut adalah visualisasi hasil running model WRF-ARW untuk kejadian Siklon Tropis Dahlia, yang mencakup tiga parameter utama di ketiga domain (D01, D02, dan D03).

### 1. Suhu Permukaan (Surface Temperature)
Visualisasi ini menunjukkan dinamika suhu permukaan daratan dan laut (SST). Pembaruan SST (*SST update*) harian diaktifkan untuk menjaga akurasi interaksi laut-atmosfer karena pergerakan Siklon Dahlia mayoritas terjadi di Samudra Hindia.

<video src="temp.mp4" controls="controls" width="100%"></video>

### 2. Perbandingan Curah Hujan (Hourly Rainfall: WRF vs MSWEP)
Visualisasi ini sangat penting untuk mengevaluasi performa model. Di sini, curah hujan per jam dari output WRF disandingkan dengan data observasi satelit MSWEP. Terlihat bagaimana model menangkap pergerakan sabuk hujan tepi (*outer band*) dari sistem siklon yang memicu hujan lebat di wilayah pesisir barat dan selatan Jawa.

<video src="hujan.mp4" controls="controls" width="100%"></video>

### 3. Kecepatan dan Arah Angin 10 Meter (10 m Wind Speed and Direction)
Memperlihatkan dengan jelas struktur sirkulasi angin siklonik di sekitar pusat tekanan rendah. Angin kencang terlihat menjalar dan berdampak pada pesisir barat daya Sumatera dan selatan Jawa, walau pusat siklon bergerak perlahan menjauhi daratan ke arah tenggara.

<video src="angin.mp4" controls="controls" width="100%"></video>

## Evaluasi Statistik (WRF vs MSWEP)

Untuk mengukur akurasi model secara kuantitatif, dilakukan evaluasi statistik antara output curah hujan WRF dengan data observasi satelit MSWEP pada area irisan (*common-area*) ketiga domain. Hal ini krusial untuk memvalidasi seberapa baik konfigurasi fisika model merepresentasikan realita atmosfer.

### 1. Time Series Curah Hujan
Grafik di bawah menunjukkan perbandingan rata-rata curah hujan per jam (*Mean Hourly Rainfall*). Pola temporal model WRF di ketiga resolusi (D01, D02, D03) terbukti cukup konsisten dalam mengikuti tren observasi MSWEP, terutama pada fase puncak badai.

![Time Series Rainfall](image_bf0182.png)

### 2. Perbandingan Titik Pusat Hujan (Rainfall Centroid)
Analisis spasial dilakukan dengan menghitung jarak *centroid* curah hujan antara WRF dan MSWEP. Jarak centroid berkisar antara 47–50 km, menunjukkan model mampu melacak lokasi pusat intensitas hujan dengan cukup presisi mengingat resolusi grid yang digunakan.

![Rainfall Centroid](image_bf0168.png)

### 3. Metrik Evaluasi dan Ranking Domain
Evaluasi kinerja model ditinjau secara holistik melalui berbagai metrik error dan kesamaan spasial, termasuk RMSE, MAE, Bias, Pearson *correlation*, SSIM, dan *Centroid Distance*. Berdasarkan metrik-metrik ini, dilakukan pemeringkatan (ranking) guna menentukan resolusi domain mana yang memberikan performa prediksi terbaik.

![Tabel Evaluasi](image_bf0148.png)
<br>
![Ranking Evaluasi](image_bf0163.png)

**Kesimpulan Evaluasi:** Domain **D02 (resolusi 10 km)** secara keseluruhan menunjukkan performa analitik terbaik (*Final Ranking: 1*) dibandingkan D01 dan D03 untuk studi kasus ini. Domain ini berhasil menyeimbangkan *error* yang relatif rendah dengan akurasi spasial (SSIM dan Centroid Distance) yang paling optimal.

## Troubleshooting

| Masalah | Penyebab | Solusi |
| :--- | :--- | :--- |
| **Nest 2 tidak muat di dalam domain induk**<br>(*Nest 2 does not fit within parent domain*) | Kombinasi parameter pembatas domain anak melewati batas luasan domain induknya. | Atur ulang posisi/ukuran domain menggunakan WRF Domain Wizard. Pastikan parameter berikut konsisten di `namelist.wps` dan `namelist.input`: <br> `parent_id`, `parent_grid_ratio`, `i_parent_start`, `j_parent_start`, `e_we`, dan `e_sn`. |
| **File `ERA5_p:*` atau `ERA5_sfc:*` tidak ditemukan** | Data ERA5 pressure level (PL) dan surface (SFC) harus di-*ungrib* terpisah menggunakan `prefix` berbeda, karena `metgrid.exe` butuh membaca dua struktur data sekaligus. | 1. Link & jalankan PL: `./link_grib.csh /path/to/ERA5/PL/*` lalu `./ungrib.exe`<br>2. Hapus link: `rm -f GRIBFILE.*`<br>3. Ubah config di namelist: `prefix = 'ERA5_sfc'`<br>4. Link & jalankan SFC: `./link_grib.csh /path/to/ERA5/SFC/*` lalu `./ungrib.exe`<br>Pastikan kedua set data rampung sebelum ke `metgrid`. |
| **`real.exe` berhenti dengan error `MPI_ABORT was invoked on rank 0`** | Pesan `MPI_ABORT` hanyalah notifikasi bahwa proses dihentikan paksa. Penyebab spesifik tidak akan terlihat dari command line. | 1. Cek file log sesungguhnya: `tail -50 rsl.error.0000`<br>2. Telusuri error di dalam log tersebut (biasanya terkait konsistensi domain, jumlah level vertikal, atau file input yang hilang/korup) lalu selesaikan sesuai dengan akar masalahnya. |

## Struktur Folder

```text
comsoftware/wrf/
├── WPS_GEOG/
├── WPS-4.3/
│   ├── data_era5/
│   ├── namelist.wps
│   ├── geo_em.d0*.nc
│   ├── ERA5_p:2017-*
│   ├── ERA5_sfc:2017-*
│   └── met_em.d0*.nc
└── WRF-4.3/test/em_real/
    ├── namelist.input
    ├── wrfbdy_d01
    ├── wrfinput_d0*
    ├── wrflowinp_d0*
    ├── rsl.error.0000
    └── wrfout_d0*
```

## Referensi

- Skamarock, W. C., et al. (2021). *A Description of the Advanced Research WRF Model Version 4.3*. NCAR Technical Note.
- ECMWF – Copernicus Climate Change Service. *ERA5 Reanalysis Documentation*.
- BMKG. *Siaran Pers "Cempaka Meluruh, Siklon Tropis Dahlia Lahir", November 2017*.
- BMKG. *Siaran Pers "Dahlia Punah, Cuaca Ekstrem Masih Mengintai", Desember 2017*.
- WRF Domain Wizard — jiririchter.github.io/WRFDomainWizard
