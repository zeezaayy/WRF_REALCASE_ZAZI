# Simulasi Numerik Siklon Tropis Dahlia (2017) Menggunakan Model WRF-ARW

![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-blue)
![WRF](https://img.shields.io/badge/WRF-v4.3-orange)
![ERA5](https://img.shields.io/badge/Data-ERA5-green)

> Dokumentasi simulasi **Siklon Tropis Dahlia (29 November–2 Desember 2017)** menggunakan **WRF-ARW v4.3** dengan data **ERA5 Reanalysis**.

---

# Latar Belakang

**Weather Research and Forecasting (WRF)** merupakan model numerik atmosfer yang banyak digunakan untuk penelitian dan simulasi cuaca. Pada repository ini WRF digunakan untuk merekonstruksi kejadian **Siklon Tropis Dahlia** menggunakan data ERA5 sehingga dapat dianalisis pola curah hujan, suhu, dan angin selama kejadian berlangsung.

Siklon Tropis Dahlia terbentuk setelah Siklon Tropis Cempaka melemah pada akhir November 2017. Walaupun pusat siklon bergerak menjauhi Indonesia, sistem ini masih memicu hujan lebat, angin kencang, dan gelombang tinggi di sebagian wilayah Sumatera dan Jawa sehingga meningkatkan potensi bencana hidrometeorologi.

Referensi BMKG:

- https://www.bmkg.go.id/siaran-pers/cempaka-meluruh-dahlia-lahir-waspada-bencana-hidrometeorologi-menghadang
- https://bbmkg3.bmkg.go.id/bbmkg3_pdf_files/11122017060450.pdf

---

# Workflow

```text
ERA5
  │
  ▼
WRF Domain Wizard
  │
  ▼
WPS (geogrid → ungrib → metgrid)
  │
  ▼
WRF (real.exe → wrf.exe)
  │
  ▼
wrfout
  │
  ▼
Visualisasi & Validasi (MSWEP)
```

---

# Konfigurasi Domain

> Tambahkan gambar domain di bawah ini.

<p align="center">
<img src="docs/domain.png" width="900">
</p>

---

# Hasil Simulasi

## 1. Temperatur Permukaan

<p align="center">
<img src="temperature permukaan.png" width="900">
</p>

Visualisasi menunjukkan distribusi temperatur permukaan selama simulasi WRF.

---

## 2. Curah Hujan

<p align="center">
<img src="curah hujan.png" width="900">
</p>

Validasi dilakukan menggunakan data observasi satelit **MSWEP** untuk mengevaluasi kemampuan model dalam merepresentasikan distribusi dan intensitas curah hujan.

---

## 3. Arah Pergerakan Siklon

<table>
<tr>
<td align="center"><b>Hasil Simulasi WRF</b></td>
<td align="center"><b>Lintasan Siklon Dahlia (BMKG)</b></td>
</tr>
<tr>
<td><img src="siklon.png" width="420"></td>
<td><img src="siklon dahlia.jpg" width="420"></td>
</tr>
</table>

Perbandingan di atas menunjukkan bahwa pola lintasan hasil simulasi memiliki kesesuaian secara umum dengan lintasan yang dipublikasikan BMKG.

---

# Hasil Validasi

## Rata-rata Curah Hujan

<p align="center">
<img src="rerata hujan.png" width="750">
</p>

Perbandingan rata-rata curah hujan menunjukkan model mampu mengikuti pola temporal observasi meskipun terdapat perbedaan pada beberapa jam tertentu.

---

## RMSE Stasiun Pengamatan

<p align="center">
<img src="rmse stasiun pengamatan.png" width="750">
</p>

Nilai RMSE digunakan untuk mengevaluasi besarnya galat hasil simulasi terhadap data pengamatan.

---

## Scatter Plot

<p align="center">
<img src="scatter plot.png" width="700">
</p>

Scatter plot memperlihatkan hubungan antara hasil simulasi WRF dengan data observasi sebagai gambaran tingkat kesesuaian model.

---

# Referensi

1. Skamarock, W. C., et al. (2021). *A Description of the Advanced Research WRF Model Version 4.3.*
2. ERA5 Reanalysis Documentation.
3. BMKG. *Cempaka Meluruh, Siklon Tropis Dahlia Lahir: Waspada Bencana Hidrometeorologi Menghadang*.
4. BMKG Regional III. *Buletin Informasi Siklon Tropis Dahlia*.
