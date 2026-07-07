# Simulasi Numerik WRF-ARW: Siklon Tropis Dahlia

Repositori ini berisi konfigurasi dan dokumentasi simulasi numerik WRF-ARW v4.3 untuk studi kasus Siklon Tropis Dahlia menggunakan data ERA5 dan tiga nested domain.

---

## Pendahuluan

Siklon Tropis Dahlia merupakan siklon tropis yang berkembang di Samudra Hindia selatan Indonesia pada akhir November 2017. Pada proyek ini dilakukan simulasi numerik menggunakan Weather Research and Forecasting Model (WRF-ARW) untuk menganalisis distribusi curah hujan serta pola kecepatan dan arah angin selama periode kejadian Siklon Tropis Dahlia.

---

## Prerequisites

- Docker Desktop
- WPS v4.3
- WRF v4.3
- Data ERA5
- WPS_GEOG
- WRF Domain Wizard
- Python / Google Colab untuk post-processing

---

## Konfigurasi Domain

Simulasi menggunakan 3 domain bersarang dengan konfigurasi akhir:

| Domain | Resolusi | Grid (e_we × e_sn) | Cakupan |
|---|---:|---:|---|
| D01 | 30 km | 165 × 93 | Regional |
| D02 | 10 km | 202 × 127 | Jawa–Bali dan sekitarnya |
| D03 | 3.33 km | 250 × 163 | Jawa bagian barat hingga tengah dan perairan selatan Jawa |

Parameter nesting:

```text
parent_id         = 1, 1, 2
parent_grid_ratio = 1, 3, 3
i_parent_start    = 1, 33, 36
j_parent_start    = 1, 11, 63
```

Pusat domain:

```text
ref_lat = -6.252
ref_lon = 115.525
```

---

## Data Input

### Data geografis
Data geografis menggunakan `geog_high_res_mandatory.tar.gz` yang diekstrak ke folder `WPS_GEOG`.

### Data meteorologi
Data meteorologi menggunakan ERA5 periode **29 November 2017 00 UTC sampai 2 Desember 2017 23 UTC**.

ERA5 diproses terpisah menjadi:
- pressure level
- surface level

---

## Konfigurasi WPS

File konfigurasi lengkap tersedia di folder `config/namelist.wps`.

Alur WPS yang digunakan:
1. `geogrid.exe`
2. `ungrib.exe` untuk ERA5 pressure level
3. ubah prefix ke `ERA5_sfc`
4. `ungrib.exe` untuk ERA5 surface
5. `metgrid.exe`

---

## Konfigurasi WRF

File konfigurasi lengkap tersedia di folder `config/namelist.input`.

Alur WRF yang digunakan:
1. `real.exe`
2. `wrf.exe`

Output utama:
- `wrfinput_d0*`
- `wrfbdy_d01`
- `wrflowinp_d0*`
- `wrfout_d0*`

---

## Troubleshooting

| No. | Masalah | Penyebab | Solusi |
|---|---|---|---|
| 1 | `Nest 2 does not fit within parent domain` | Posisi atau ukuran nested domain melewati batas parent domain. | Atur ulang domain dengan WRF Domain Wizard dan samakan parameter domain pada `namelist.wps` dan `namelist.input`. |
| 2 | File `ERA5_p:*` atau `ERA5_sfc:*` tidak ditemukan | Pressure level dan surface belum diproses terpisah dengan prefix yang sesuai. | Jalankan `ungrib.exe` terpisah untuk pressure level dan surface, lalu lanjutkan ke `metgrid.exe` setelah kedua file intermediate tersedia. |
| 3 | `real.exe` menghasilkan `MPI_ABORT` | `MPI_ABORT` hanya pesan penghentian proses; penyebab utama tercatat di `rsl.error.0000`. | Periksa error dengan `tail -50 rsl.error.0000`, perbaiki penyebabnya, lalu jalankan ulang `real.exe`. |

---

## Struktur Folder

```text
WRF_REALCASE_ZAZI/
├── README.md
├── .gitignore
├── config/
│   ├── namelist.wps
│   └── namelist.input
└── docs/
    └── TROUBLESHOOTING.md
```

---

## Referensi

- WRF Domain Wizard
- WRF-ARW v4.3 Documentation
- ERA5 Copernicus Documentation
- BMKG: Siklon Tropis Dahlia (2017)
