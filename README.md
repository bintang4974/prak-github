# Perkiraan Kompresi Audio (Estimasi Terukur)

Sumber: pengukuran pada sample audio di `storage/app/public/audio/7/77` (file contoh: `1778590343_6a0322870a8ab.mp3`)

- Durasi sample: **104.01 detik**
- Ukuran terukur (bytes):
  - Original MP3: **2.031.123 bytes**
  - Opus (64 kbps): **1.104.339 bytes**
  - Opus (32 kbps): **688.131 bytes**
  - Opus (16 kbps): **242.179 bytes**

---

## Rata-rata per-menit (terukur)
Nilai dihitung dengan menskalakan ukuran terukur ke 60 detik dari sample 104.01 detik.

| Format / Bitrate | Bytes / menit | KiB / menit | MB / menit |
|---|---:|---:|---:|
| Original MP3 (~156 kbps) | 1.171.689 | 1.144,2 | 1,12 |
| Opus 64 kbps | 637.057 | 622,1 | 0,61 |
| Opus 32 kbps | 396.960 | 387,7 | 0,38 |
| Opus 16 kbps | 139.705 | 136,4 | 0,13 |

---

## Perkiraan ukuran untuk durasi umum (skala terukur)
| Durasi | Original MP3 (~156 kbps) | Opus 64 kbps | Opus 32 kbps | Opus 16 kbps |
|---:|---:|---:|---:|---:|
| 1 menit (60s) | 1,12 MB | 0,61 MB | 0,38 MB | 0,13 MB |
| 1m 24s (84s) | 1,57 MB | 0,86 MB | 0,54 MB | 0,18 MB |
| 5 menit (300s) | 5,60 MB | 3,05 MB | 1,90 MB | 0,66 MB |
| 10 menit (600s) | 11,20 MB | 6,10 MB | 3,80 MB | 1,32 MB |

> Catatan: nilai ini diskalakan secara linier dari sample terukur. Overhead container dan metadata sudah tercermin di pengukuran.

---

## Contoh skenario skalabilitas (1000 pengguna)
Asumsi baru sesuai permintaan:
- 1000 pengguna aktif
- 3 rekaman per pengguna per hari
- Durasi rata-rata per rekaman: **2 menit**
- 30 hari per bulan

Total rekaman per bulan = 1000 × 3 × 30 = **90.000 rekaman**

Per-rekam (MB) = (MB / menit) × 2
- Original MP3: 1,12 × 2 = **2,24 MB / rekaman**
- Opus 16 kbps: 0,13 × 2 = **0,26 MB / rekaman**

Perkiraan storage bulanan:

| Skenario | Per-rekam (MB) | Total rekaman | Total storage (MB) | Total storage (GB) |
|---|---:|---:|---:|---:|
| Original MP3 (tanpa kompresi) | 2,24 | 90.000 | 201.600 | 196,88 GB |
| Opus 16 kbps (rekaman browser + kompresi rendah di server) | 0,26 | 90.000 | 23.400 | 22,85 GB |

Perkiraan penghematan: **~174,03 GB per bulan** (~88,5% pengurangan) dibanding menyimpan MP3 asli.

---

## Pengamatan & Rekomendasi
- Rekaman di sisi klien dengan `audioBitsPerSecond: 16000` (Opus 16 kbps) memberikan penghematan storage dan bandwidth terbesar, serta mengurangi beban CPU server.
- Untuk upload manual (MP3/WAV), tetap gunakan `CompressAudioJob` di server dengan pemetaan audio-only (`-map 0:a:0 -vn -sn -dn`) agar cover art tidak memecahkan proses transcode dan hasilnya konsisten (WebM/Opus).
- Atur `AUDIO_COMPRESSION_BITRATE` di `.env` untuk menyesuaikan trade-off kualitas/ukuran; 16 kbps sangat baik untuk konten suara saja.
- Tambahkan buffer 10–20% untuk kapasitas produksi untuk mengakomodasi metadata dan lonjakan penggunaan.

---

## Cara mereproduksi per-menit (contoh perintah)
Jalankan snippet PowerShell berikut untuk mereproduksi skala per-menit dari pengukuran (ganti variabel dengan ukuran/durasi terukur Anda):

```powershell
$durationSec = 104.01
$sizeBytes = 242179 # contoh untuk opus 16kbps
$perMin = ($sizeBytes / $durationSec) * 60
$kbPerMin = $perMin / 1024
$mbPerMin = $perMin / 1024 / 1024
Write-Output "$perMin bytes/min = $kbPerMin KiB/min = $mbPerMin MB/min"
```

---

Dihasilkan dari sesi diagnostik HifzhCare pada 2026-05-12. Lihat `app/Jobs/CompressAudioJob.php` untuk pipeline konversi yang digunakan dalam pengujian.
