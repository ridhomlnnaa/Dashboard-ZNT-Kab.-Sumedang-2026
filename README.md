# Dashboard ZNT Kab. Sumedang 2026

Dashboard pemantauan survei Zona Nilai Tanah (ZNT) berbasis satu file `index.html`.
Semua data diproses di browser pengguna, tanpa server dan tanpa database.
Tampilan dan fitur mengikuti Dashboard ZNT Kab. Pangandaran 2026, dengan pembanding **ZNT 2023**.

## Tampilan

Tombol di kanan atas mengatur tampilan:

- **Peta + Data:** panel kiri berisi kartu ringkasan, peta di kanan, tabel rekap di bawah
- **Peta:** hanya peta
- **Data:** hanya panel kiri dan tabel rekap

### Isi tab Data

Panel kiri:
- Filter surveyor, kecamatan, desa, dan pencarian nomor zona
- Klasifikasi nilai tanah 8 kelas (7 batas manual, atau kuantil otomatis)
- Data (jumlah zona dan titik, lapisan yang termuat, unduh CSV zona dan titik)
- Status zona, Kualitas data (QC), Capaian titik aktual
- Progres per surveyor, Tren harian survei, Pemeriksaan data, Log sampel terbaru

Tabel rekap (enam sub-tab, semuanya mengikuti filter):
1. Analisis nilai tanah (perubahan vs ZNT 2023 per kecamatan, kenaikan/penurunan terbesar, harga tertinggi/terendah, sampel terbanyak)
2. Belum survei
3. Sampel kurang
4. RSD di atas 30%
5. Tanggal transaksi
6. Semua zona

Setiap sub-tab punya tombol **Salin untuk spreadsheet**, dan klik baris zona membuka zona itu di peta.

## Cara memuat data

Klik **Muat file (.zip / .dbf)**, lalu pilih semua file sekaligus.

| File | Format | Dikenali dari |
|---|---|---|
| Zona (poligon) | `.zip` shapefile | kolom `HISTZONE` |
| Titik sampel | `.dbf` atau `.zip` | kolom `Nomor_Samp` dan `Tanggal_Pe` |
| ZNT 2023 | `.zip` shapefile | nama file mengandung `2023` |
| Kecamatan | `.zip` shapefile | nama file mengandung `kec` |
| Desa | `.zip` shapefile | nama file mengandung `desa` atau `kel` |
| AOI awal | `.zip` shapefile | nama file mengandung `aoi` |
| AOI baru | `.zip` shapefile | nama file mengandung `aoi` dan `baru` |

Catatan penting:
- Peta zona **membutuhkan poligon**, jadi zona dimuat dari `.zip` shapefile. DBF saja hanya memuat tabelnya, bukan bentuk zona.
- Titik sampel cukup berupa DBF karena koordinatnya diambil dari kolom `X` dan `Y`.
- Shapefile dalam `.zip` sebaiknya menyertakan `.shp`, `.shx`, `.dbf`, dan `.prj`. Dengan `.prj`, koordinat dikonversi ke WGS84 otomatis.
- File titik tidak perlu dimuat ulang saat file zona berubah, dan sebaliknya.
- Tombol **Lihat data contoh** hanya untuk mencoba tampilan. Datanya acak.

## Kolom dan sumber data (tidak perlu mengedit kode)

Nama kolom **dideteksi otomatis**. Khusus kolom nomor zona pada titik sampel, dashboard memilih kolom yang nilainya paling cocok dengan nomor zona yang ada, jadi namanya boleh berbeda dari Pangandaran.

Setelah data dimuat, lihat kartu **Pemeriksaan data & kolom** di panel kiri:
- Peringatan muncul bila ada yang janggal, misalnya titik tidak terhubung ke zona, kolom nilai kosong, atau luas kolom berbeda dari poligon.
- Menampilkan contoh nomor zona dari data zona dan dari titik, supaya perbedaan format mudah terlihat.
- Menu **Pemetaan kolom dan sumber data** memungkinkan Anda memilih kolom yang benar untuk setiap isian, lalu klik **Terapkan**. Pilihan Anda diingat di browser.

Tiga pilihan sumber data:
- **Zona titik:** Otomatis (pakai kolom bila cocok, jika tidak pakai lokasi titik di dalam poligon zona), Dari kolom, atau Dari lokasi (spasial).
- **Sumber luas:** Dari poligon (bawaan) atau dari kolom. Satuan kolom luas (m², hektare, km²) dikenali otomatis.
- **Statistik zona:** dari kolom atribut zona (Jml_Titik, RSD_Persen) atau dihitung dari titik sampel.

Nomor zona dibakukan sebelum dicocokkan: huruf besar-kecil, spasi, dan nol di depan diabaikan (`" 210p "` sama dengan `210P`).

Aturan lain (target titik menurut luas, batas RSD, jumlah minimal titik, filter kabupaten) ada di bagian `CFG` pada awal `<script>` di `index.html`.

## Rumus

- **Luas zona** = luas poligon (m²), dihitung dari koordinat. Bagian zona yang terpecah dijumlahkan
- **Rata-rata** = rata-rata nilai titik yang nilainya lebih dari 0 dalam zona
- **RSD (%)** = simpangan baku sampel / rata-rata × 100 (dihitung bila ada minimal 3 titik, kecuali kolom `RSD_Persen` sudah terisi di data zona)
- **Capaian:** Selesai bila titik ≥ target, Proses bila ada titik tetapi kurang, Belum bila 0
- **QC:** Belum (0 titik), Kurang (1 atau 2 titik), Revisi (RSD di atas batas), Valid (selain itu)
- **Perubahan vs 2023 (%)** = (rata-rata 2026 / nilai ZNT 2023 − 1) × 100

### Pembanding ZNT 2023

Zona 2026 dicocokkan dengan poligon ZNT 2023 yang sejenis (Pertanian dengan Pertanian, Non-Pertanian dengan Non-Pertanian). Pencocokan memakai **sebaran titik grid di dalam zona**, bukan irisan geometri yang persis. Nilai pembanding dipakai hanya bila poligon 2023 menutupi minimal 50% luas zona. Hasilnya pendekatan yang baik untuk pemantauan, tetapi bukan pengganti perhitungan resmi.

## Mengunggah ke GitHub Pages

1. Buat repository baru di GitHub, misalnya `Dashboard-ZNT-Kab.Sumedang2026`.
2. Unggah `index.html` dan `README.md` ke branch `main`.
3. Buka **Settings → Pages**. Pada **Build and deployment**, pilih **Deploy from a branch**, lalu branch `main` dan folder `/ (root)`, kemudian **Save**.
4. Tunggu 1 sampai 2 menit. Alamatnya berbentuk `https://<username>.github.io/<nama-repo>/`.

Data tidak ikut tersimpan di GitHub. Setiap kali halaman dibuka, file data dimuat lagi lewat tombol **Muat file**. Cara ini aman untuk data survei yang bersifat internal.

## Pustaka yang dipakai (via CDN)

- Leaflet 1.9.4
- shpjs 6.1.0
- Peta dasar Esri dan OpenStreetMap
- Font IBM Plex (Google Fonts)

Dashboard memerlukan koneksi internet untuk memuat pustaka dan peta dasar.
