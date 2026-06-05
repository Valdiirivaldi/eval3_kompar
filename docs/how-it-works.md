# Bagaimana Cara Kerjanya?

> Penjelasan mendalam tentang konsep komputasi paralel yang digunakan dalam proyek ini.

---

## Daftar Isi

- [Apa itu Embarrassingly Parallel?](#apa-itu-embarrassingly-parallel)
- [Mengapa Image Processing Cocok untuk Paralelisasi?](#mengapa-image-processing-cocok-untuk-paralelisasi)
- [Prosesor Multi-Core dan Parallel Computing](#prosesor-multi-core-dan-parallel-computing)
- [ProcessPoolExecutor vs ThreadPoolExecutor](#processpoolexecutor-vs-threadpoolexecutor)
- [Apa itu GIL (Global Interpreter Lock)?](#apa-itu-gil-global-interpreter-lock)
- [Bagaimana .map() Bekerja Secara Paralel?](#bagaimana-map-bekerja-secara-paralel)
- [Speedup dan Hukum Amdahl](#speedup-dan-hukum-amdahl)

---

## Apa itu Embarrassingly Parallel?

**Embarrassingly Parallel** (atau *pleasantly parallel*) adalah istilah dalam parallel computing untuk tugas yang dapat dengan mudah dipecah menjadi sub-tugas independen tanpa memerlukan:

- Komunikasi antar-proses
- Sinkronisasi data
- Memory bersama (shared memory)

Ciri-ciri tugas Embarrassingly Parallel:

| Ciri | Penjelasan |
|------|------------|
| **Independen** | Setiap sub-tugas tidak bergantung pada hasil sub-tugas lain |
| **Tanpa komunikasi** | Tidak perlu bertukar data antar-proses |
| **Tanpa sinkronisasi** | Urutan eksekusi tidak penting |
| **Hasil kumulatif** | Hasil akhir adalah kumpulan dari semua hasil sub-tugas |

### Contoh Embarrassingly Parallel

| Tugas | Alasan |
|-------|--------|
| **Image processing** (proyek ini) | Setiap gambar diproses sendiri-sendiri |
| Render frame animasi | Setiap frame di-render independen |
| Web scraping banyak halaman | Setiap halaman di-download sendiri |
| Parametric sweep (simulasi) | Setiap parameter dijalankan sendiri |
| Uji A/B machine learning | Setiap model dilatih dengan data berbeda |

---

## Mengapa Image Processing Cocok untuk Paralelisasi?

Dalam proyek ini, setiap gambar melewati pipeline yang sama:

```
Buka File → Konversi Grayscale → Filter Blur → Simpan Hasil
```

Pipeline ini adalah **unit kerja yang identik** untuk setiap gambar. Tidak ada gambar yang membutuhkan data dari gambar lain. Ini berarti:

```
Gambar A: [Buka → Grayscale → Blur → Simpan]  ← independen
Gambar B: [Buka → Grayscale → Blur → Simpan]  ← independen
Gambar C: [Buka → Grayscale → Blur → Simpan]  ← independen
```

Ketiga gambar di atas bisa diproses **bersamaan** tanpa masalah.

---

## Prosesor Multi-Core dan Parallel Computing

Komputer modern memiliki CPU dengan beberapa core (inti). Contoh:

| Tipe CPU | Jumlah Core |
|----------|-------------|
| Intel Core i3 | 2-4 core |
| Intel Core i5 | 4-6 core |
| Intel Core i7 | 6-10 core |
| Intel Core i9 | 8-18 core |
| AMD Ryzen 5 | 6 core |
| AMD Ryzen 7 | 8 core |
| AMD Ryzen 9 | 12-16 core |

### Analogi Restoran 🍽️

Bayangkan sebuah restoran dengan **1 koki vs 4 koki**:

| Situasi | 1 Koki (Sekuensial) | 4 Koki (Paralel) |
|---------|---------------------|-------------------|
| **10 menu** | Memasak 1 per 1 → Total = jumlah waktu semua menu | Membagi 10 menu ke 4 koki → Masing-masing 2-3 menu |
| **Waktu per menu** | ~2 menit | ~2 menit |
| **Total waktu** | ~20 menit | ~5-6 menit ✅ |

Di sinilah letak **speedup** — tugas yang sama diselesaikan dalam waktu lebih singkat dengan memanfaatkan banyak pekerja (core).

---

## ProcessPoolExecutor vs ThreadPoolExecutor

### Perbandingan

| Aspek | `ProcessPoolExecutor` | `ThreadPoolExecutor` |
|-------|-----------------------|----------------------|
| **Unit eksekusi** | Proses terpisah | Thread dalam satu proses |
| **Memory** | Terpisah (masing-masing proses punya memory sendiri) | Bersama (shared memory) |
| **Terkena GIL?** | ❌ Tidak (setiap proses punya GIL sendiri) | ✅ Ya (satu GIL untuk semua thread) |
| **Cocok untuk** | CPU-bound tasks | I/O-bound tasks |
| **Overhead** | Lebih besar (membuat proses baru) | Lebih kecil |
| **Cara kerja** | `fork()` atau `spawn()` | Dalam satu proses |

### CPU-bound vs I/O-bound

| Tipe | Ciri | Contoh | Tool yang Tepat |
|------|------|--------|-----------------|
| **CPU-bound** | Banyak kalkulasi, processor jadi bottleneck | Image processing, video encoding, matrix multiplication | `ProcessPoolExecutor` |
| **I/O-bound** | Banyak menunggu input/output, kecepatan I/O jadi bottleneck | Web scraping, file reading, database query | `ThreadPoolExecutor` atau `asyncio` |

### Ilustrasi

**CPU-bound (Image Processing)**:
```
Thread 1: [████████████████████] 100% CPU
Thread 2: [████████████████████] 100% CPU  ← GIL blocking!
```
Dengan ThreadPoolExecutor, thread 2 harus menunggu GIL yang sedang dipegang thread 1.

**ProcessPoolExecutor (Image Processing)**:
```
Proses 1: [████████████████████] Core 1
Proses 2: [████████████████████] Core 2 ← BEBAS dari GIL!
```

---

## Apa itu GIL (Global Interpreter Lock)?

**GIL** adalah mekanisme di CPython (implementasi standar Python) yang memastikan **hanya satu thread** yang mengeksekusi bytecode Python dalam satu waktu.

### Kenapa GIL Ada?

GIL ada karena manajemen memory Python (**garbage collection**) tidak thread-safe. Tanpa GIL, perlu locking yang sangat kompleks dan lambat untuk setiap operasi memory.

### Dampak GIL

```python
# Ini TIDAK akan lebih cepat dengan ThreadPoolExecutor
# karena CPU-bound task dan GIL

from concurrent.futures import ThreadPoolExecutor  # ❌

def tugas_berat():
    for i in range(10000000):
        _ = i * i * i
```

```python
# Ini akan lebih cepat dengan ProcessPoolExecutor
# karena setiap proses punya GIL sendiri

from concurrent.futures import ProcessPoolExecutor  # ✅

def tugas_berat():
    for i in range(10000000):
        _ = i * i * i
```

### Solusi untuk CPU-bound Task

| Solusi | Cara Kerja |
|--------|------------|
| **`ProcessPoolExecutor`** | Bikin beberapa proses, masing-masing punya GIL sendiri ✅ |
| **Library C extension** | NumPy, Pandas — melepas GIL saat eksekusi C |
| **PyPy / Jython** | Implementasi Python tanpa GIL (atau dengan GIL berbeda) |
| **multiprocessing** | Modul serupa, lebih low-level |

---

## Bagaimana .map() Bekerja Secara Paralel?

Method `.map()` pada `ProcessPoolExecutor` adalah salah satu cara termudah untuk parallel processing.

### Analogi: Ban Berjalan 🏭

```
Input: [gambar1, gambar2, gambar3, gambar4, gambar5, gambar6]
                           ↓
                ┌──────────────────────┐
                │  ProcessPoolExecutor  │
                │  (4 worker processes) │
                └──────────────────────┘
                           ↓
Worker 1: gambar1 → [proses] → selesai
Worker 2: gambar2 → [proses] → selesai
Worker 3: gambar3 → [proses] → selesai
Worker 4: gambar4 → [proses] → selesai
                           ↓
Worker 1: gambar5 → [proses] → selesai
Worker 2: gambar6 → [proses] → selesai
                           ↓
Output: [gambar1, gambar2, gambar3, gambar4, gambar5, gambar6]
```

### Cara Kerja Detail

```python
with ProcessPoolExecutor() as executor:
    hasil = executor.map(proses_satu_gambar, daftar_gambar)
```

1. **Executor** membuat worker processes (jumlahnya = CPU core)
2. **.map()** mengambil fungsi `proses_satu_gambar` dan iterable `daftar_gambar`
3. **Distribusi**: Setiap worker diberi gambar untuk diproses
4. **Eksekusi paralel**: Semua worker bekerja **bersamaan**
5. **Pengumpulan hasil**: `hasil` berisi return values, urutannya sesuai input
6. **Cleanup**: `with` statement menutup executor secara otomatis

### Perbandingan dengan map() Biasa

```python
# SEKUENSIAL - map() biasa
hasil = list(map(proses_satu_gambar, daftar_gambar))  # Satu per satu

# PARALEL - ProcessPoolExecutor.map()
with ProcessPoolExecutor() as executor:
    hasil = executor.map(proses_satu_gambar, daftar_gambar)  # Bersamaan!
```

---

## Speedup dan Hukum Amdahl

### Hukum Amdahl

Hukum Amdahl adalah rumus yang memprediksi **speedup maksimum** yang bisa dicapai dengan parallel computing:

```
Speedup = 1 / ((1 - P) + P/N)

Di mana:
    P = Proporsi tugas yang bisa diparalelkan (0.0 - 1.0)
    N = Jumlah processor/core
```

### Contoh Penerapan

Jika 80% tugas bisa diparalelkan (P = 0.8) dengan 4 core (N = 4):

```
Speedup = 1 / ((1 - 0.8) + 0.8/4)
        = 1 / (0.2 + 0.2)
        = 1 / 0.4
        = 2.5x
```

Artinya, bahkan dengan core tak terbatas, speedup maksimum untuk tugas dengan 80% paralelisasi adalah:
```
Speedup_max = 1 / (1 - P) = 1 / 0.2 = 5x
```

### Tabel Speedup Teoritis

| P (dapat diparalelkan) | N=2 | N=4 | N=8 | N=16 | Maksimum |
|------------------------|-----|-----|-----|------|----------|
| 50% (0.5) | 1.33x | 1.60x | 1.78x | 1.88x | 2x |
| 75% (0.75) | 1.60x | 2.29x | 2.91x | 3.37x | 4x |
| 90% (0.9) | 1.82x | 3.07x | 4.70x | 6.40x | 10x |
| **99% (0.99)** | **1.98x** | **3.88x** | **7.48x** | **13.9x** | **100x** |
| **100% (1.0)** | **2x** | **4x** | **8x** | **16x** | **∞** |

> Proyek image processing ini mendekati **P = 100%** karena tidak ada bagian serial — semua gambar independen. Inilah mengapa speedup yang diamati mendekati jumlah core.

---

## Kesimpulan

1. **Embarrassingly Parallel** = tugas yang mudah diparalelkan karena sub-tugas independen
2. **Image processing** adalah contoh sempurna dari tugas ini
3. **ProcessPoolExecutor** digunakan karena tugas CPU-bound membutuhkan proses terpisah (menghindari GIL)
4. **.map()** adalah cara elegan untuk mendistribusikan tugas ke worker processes
5. **Hukum Amdahl** membatasi speedup maksimum — tapi untuk tugas 100% paralel, speedup mendekati jumlah core

---

[← Kembali ke Beranda](index.md) | [Lanjut ke Penjelasan Kode →](code-walkthrough.md)
