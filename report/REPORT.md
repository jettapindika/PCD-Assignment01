# Laporan Analisis — PCD Assignment 01: Down Sampling & Up Sampling

**Mata Kuliah:** Pengolahan Citra Digital

## 1. Pernyataan masalah

Tugas ini menganalisis dua operasi fundamental resolusi citra: **down sampling** (menurunkan resolusi, destruktif) dan **up sampling** (menaikkan resolusi, tidak pernah menambah informasi). Down sampling diimplementasikan dengan tiga metode reduksi blok — **Max**, **Average**, dan **Median** — sementara up sampling memakai tiga interpolator: **Nearest Neighbor (NN)**, **Bilinear**, dan **Bicubic**. Analisis menggunakan empat citra uji 512×512 dengan karakteristik berbeda: gradasi halus, tepi tajam, frekuensi tinggi (papan catur 16 px), dan citra menyerupai foto. Kualitas diukur objektif dengan **PSNR** dan **SSIM** pada grayscale. (Catatan: soal menulis "Medium"; diinterpretasikan sebagai *Median*.)

## 2. Hasil utama

### 2.1 Round-trip down sampling → up sampling (factor 4)

| Citra | Kombinasi terbaik | PSNR / SSIM |
|---|---|---|
| Gradasi halus | average + bilinear | **70.15 dB / 0.9999** |
| Frekuensi tinggi | semua downsampler + NN | **∞ (identik)** |
| Tepi tajam | median + NN (SSIM) / average + NN (PSNR) | 23.5–24.9 dB / 0.94–0.96 |
| Menyerupai foto | average + bicubic | **35.81 dB / 0.8117** |

### 2.2 Kecepatan interpolasi (128×128 → 512×512, rata-rata 5 run)

| Metode | Waktu | Relatif |
|---|---|---|
| NN | 1.0 ms | 1.0× |
| Bicubic (Pillow) | 1.9 ms | 1.9× |
| Bilinear (NumPy manual) | 8.8 ms | 8.8× |

## 3. Analisis

**Metode reduksi menentukan informasi yang selamat.** Average adalah filter low-pass: frekuensi tinggi hilang tetapi struktur besar terjaga — round-trip-nya pada citra foto unggul (35.8 dB vs 27.6 dB untuk max) karena max mendistorsi distribusi intensitas ke arah terang (piksel outlier mendominasi blok). Median paling seimbang dan unggul pada SSIM citra tepi tajam (0.9622) karena tepi biner yang selaras grid bertahan utuh.

**Kemenangan bilinear pada gradasi halus (70.15 dB) adalah validasi matematis, bukan kebetulan.** Fungsi intensitas linear direkonstruksi secara eksak oleh interpolasi linear, dan average mempertahankan nilai rata-rata blok yang tepat berada pada grid sampel bilinear. Kesesuaian teori dan pengukuran ini menjadi bukti kebenaran implementasi.

**Kasus papan catur memperlihatkan aliasing vs replication.** Pola periodik 16 px dengan factor 4 menghasilkan blok 4×4 seragam sehingga NN mereplikasi tanpa error (∞ dB), sementara average menghapus frekuensi tinggi dan bilinear/bicubic hanya mencapai 13.98–15.48 dB. Justru ketika periode *tidak* habis dibagi factor, kombinasi tanpa anti-aliasing akan menghasilkan moiré — alasan `PIL.Image.resize` menerapkan low-pass secara default sebelum minifikasi.

**Trade-off interpolator.** NN tercepat (replikasi memori murni) tetapi berblok; bilinear menghaluskan dengan melembutkan tepi; bicubic paling tajam pada foto namun menghasilkan *ringing* di sekitar tepi biner (SSIM terendah 0.9228 pada citra tepi). Perbandingan waktu mencerminkan implementasi (NumPy vs C Pillow), bukan kompleksitas teoretis — per piksel keluaran: NN O(1), bilinear O(4), bicubic O(16).

## 4. Kesimpulan

1. Down sampling bersifat destruktif dan tidak dapat dibalik; kualitas round-trip ditentukan oleh kesesuaian (metode reduksi, interpolator, karakter frekuensi citra).
2. Aturan praktis berbasis data: citra foto → *average + bicubic*; citra grafis biner → *median + NN*; citra halus → *average + bilinear*; pola periodik selaras → NN.
3. PSNR dan SSIM kadang berbeda pendapat — keduanya wajib dilaporkan bersama, dilengkapi penilaian visual artefak.

---
*Seluruh angka pada laporan ini dihasilkan dari eksekusi notebook `PCD_Assignment01.ipynb`; metrik lengkap tersedia di `results/results_roundtrip.csv`.*
