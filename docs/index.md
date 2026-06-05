# Parallel Image Processing

> **Perbandingan Komputasi Sekuensial vs Paralel untuk Image Processing**  
> Parallel Computing — Universitas Komputer Indonesia

---

## 📋 Daftar Isi

- [Latar Belakang](#latar-belakang)
- [Konsep](#konsep)
- [Teknologi yang Digunakan](#teknologi-yang-digunakan)
- [Struktur Folder](#struktur-folder)
- [Alur Kerja Program](#alur-kerja-program)
- [Cara Menjalankan](#cara-menjalankan)
- [Hasil dan Analisis](#hasil-dan-analisis)
- [Penjelasan Kode](#penjelasan-kode)
- [Referensi](#referensi)
- [Kontributor](#kontributor)

---

### 📖 Dokumentasi Lainnya

| Halaman | Deskripsi |
|---------|-----------|
| [🔬 Bagaimana Cara Kerjanya?](how-it-works.md) | Penjelasan mendalam tentang konsep parallel computing, GIL, ProcessPoolExecutor, Hukum Amdahl |
| [💻 Penjelasan Kode Lengkap](code-walkthrough.md) | Panduan baris-per-baris untuk memahami semua kode dalam program |
| [📊 Hasil dan Analisis](results.md) | Data pengujian, tabel speedup, grafik, dan analisis performa |

---

## Latar Belakang

Dalam dunia komputasi, terdapat tugas-tugas yang disebut **Embarrassingly Parallel** — tugas yang dapat dengan mudah dipecah menjadi bagian-bagian kecil yang dapat diproses secara independen tanpa perlu sinkronisasi atau komunikasi antar-proses.

**Image processing** adalah contoh sempurna dari tugas ini. Memproses 10 gambar sekaligus tidak memerlukan data dari satu sama lain — setiap gambar berdiri sendiri. Ini membuat image processing menjadi kandidat ideal untuk demonstrasi konsep **parallel computing**.

Proyek ini bertujuan untuk membandingkan performa eksekusi **sekuensial** (berurutan) vs **paralel** (bersamaan) dalam konteks pengolahan citra, menggunakan Python.

---

## Konsep

### Sekuensial (Berurutan)

```
Gambar 1 → [Proses] → Selesai → Gambar 2 → [Proses] → Selesai → ...
```

Pada pendekatan sekuensial, setiap gambar diproses **satu per satu**. CPU hanya mengerjakan satu gambar dalam satu waktu. Jika ada 10 gambar, waktu total adalah **penjumlahan** waktu setiap gambar.

### Paralel (Bersamaan)

```
              ┌→ [Proses] Gambar 1
              │
CPU Core 1 ───┼→ [Proses] Gambar 2
              │
              └→ [Proses] Gambar 3

              ┌→ [Proses] Gambar 4
              │
CPU Core 2 ───┼→ [Proses] Gambar 5
              │
              └→ [Proses] Gambar 6
```

Pada pendekatan paralel, gambar-gambar dibagi ke **beberapa CPU core** yang bekerja secara simultan. Waktu total mendekati waktu terlama dari satu batch, bukan penjumlahan semua.

### Speedup

**Speedup** adalah rasio antara waktu sekuensial dibagi waktu paralel:

```
Speedup = Waktu Sekuensial / Waktu Paralel
```

- **Speedup > 1** → Paralel lebih cepat ✅
- **Speedup = 1** → Sama saja
- **Speedup < 1** → Sekuensial lebih cepat (overhead paralel terlalu besar)

---

## Teknologi yang Digunakan

| Teknologi | Kegunaan |
|-----------|----------|
| **Python 3** | Bahasa pemrograman utama |
| **Pillow (PIL)** | Membaca, memanipulasi, dan menyimpan gambar |
| **concurrent.futures** | Menyediakan `ProcessPoolExecutor` untuk paralelisasi |
| **time** | Mengukur durasi eksekusi dengan presisi |
| **pathlib** | Manipulasi path file system (cross-platform) |

### Kenapa ProcessPoolExecutor?

Image processing adalah **CPU-bound task** — tugas yang membutuhkan banyak komputasi prosesor. Python memiliki **GIL (Global Interpreter Lock)** yang membatasi eksekusi thread. Akibatnya:

- ❌ **ThreadPoolExecutor** → Tidak efektif untuk CPU-bound karena GIL
- ✅ **ProcessPoolExecutor** → Membuat beberapa proses independen, masing-masing dengan interpreter Python sendiri, sehingga benar-benar memanfaatkan **multi-core CPU**

---

## Struktur Folder

```
📁 project-root/
├── 📄 parallel_image_processing.py   # Program utama
├── 📁 images/                         # Folder input (letakkan gambar di sini)
│   ├── foto1.jpg
│   ├── foto2.png
│   └── ...
├── 📁 hasil_sekuensial/              # Output hasil sequential (auto-generated)
├── 📁 hasil_paralel/                 # Output hasil parallel (auto-generated)
└── 📁 docs/                          # Dokumentasi GitHub Pages
    ├── 📄 index.md
    └── 📄 _config.yml
```

---

## Alur Kerja Program

1. **Validasi Input** — Memeriksa apakah folder `images/` ada dan berisi file `.jpg`/`.png`
2. **Persiapan Output** — Membuat folder `hasil_sekuensial/` dan `hasil_paralel/` jika belum ada
3. **Proses pada Setiap Gambar**:
   - Buka file gambar
   - Konversi ke **grayscale** (mode `'L'`)
   - Terapkan **filter blur** (`ImageFilter.BLUR`)
   - Simpan hasil ke **kedua folder output**
4. **Tes Sekuensial** — Proses semua gambar satu per satu dengan loop `for`, ukur waktu
5. **Tes Paralel** — Proses semua gambar bersamaan dengan `ProcessPoolExecutor`, ukur waktu
6. **Kesimpulan** — Tampilkan perbandingan waktu dan hitung speedup

---

## Cara Menjalankan

### 1. Clone Repository

```bash
git clone https://github.com/Student-Embedded-Control-and-AI-Fest/CuffnCode.git
cd CuffnCode
```

### 2. Install Dependencies

```bash
pip install Pillow
```

### 3. Siapkan Gambar

Letakkan file `.jpg` atau `.png` ke dalam folder `images/`.

```
images/
├── contoh1.jpg
├── contoh2.jpg
├── contoh3.jpg
└── ...
```

### 4. Jalankan Program

```bash
python parallel_image_processing.py
```

### 5. Lihat Hasil

- Hasil pemrosesan akan tersimpan di folder `hasil_sekuensial/` dan `hasil_paralel/`
- Perbandingan waktu akan ditampilkan di terminal

---

## Hasil dan Analisis

### Contoh Output Terminal

```
============================================================
  PERBANDINGAN SEKUENSIAL vs PARALEL (IMAGE PROCESSING)
============================================================

Ditemukan 6 gambar: ['foto1.jpg', 'foto2.jpg', ..., 'foto6.jpg']

--------------------------------------------------
[SEKUENSIAL] Memproses gambar secara berurutan...
  -> foto1.jpg selesai
  -> foto2.jpg selesai
  ...

Waktu sekuensial: 1.25 detik
--------------------------------------------------

--------------------------------------------------
[PARALEL] Memproses gambar secara paralel...
  -> foto1.jpg selesai
  -> foto2.jpg selesai
  ...

Waktu paralel: 0.42 detik
--------------------------------------------------

====================================================
================ KESIMPULAN ========================
Waktu Sekuensial : 1.25 detik
Waktu Paralel    : 0.42 detik
Komputasi Paralel lebih cepat 2.98x lipat!
====================================================
```

### Analisis

| Metode | Waktu | Kecepatan |
|--------|-------|-----------|
| **Sekuensial** | 1.25 detik | 1x (baseline) |
| **Paralel** (4 core) | 0.42 detik | ~3x lebih cepat |

**Catatan:** Speedup sangat tergantung pada:
- Jumlah CPU core komputer
- Ukuran gambar
- Jumlah gambar yang diproses
- Proses latar belakang lain yang berjalan

---

## Penjelasan Kode

### Fungsi Inti: `proses_satu_gambar()`

Fungsi ini adalah **unit kerja** terkecil. Fungsi yang SAMA digunakan oleh metode sekuensial maupun paralel — menjamin perbandingan yang **adil (apple-to-apple)**.

```python
def proses_satu_gambar(nama_file: str) -> str:
    path_input = FOLDER_INPUT / nama_file
    gambar = Image.open(path_input)
    gambar_grayscale = gambar.convert("L")
    gambar_blur = gambar_grayscale.filter(ImageFilter.BLUR)
    gambar_blur.save(FOLDER_SEKUENSIAL / nama_file)
    gambar_blur.save(FOLDER_PARALEL / nama_file)
    gambar.close()
    return nama_file
```

### Eksekusi Sekuensial

```python
for gambar in daftar_gambar:
    proses_satu_gambar(gambar)
```

Loop sederhana — proses satu per satu. Sederhana, mudah dipahami, tapi lambat untuk banyak gambar.

### Eksekusi Paralel

```python
with ProcessPoolExecutor() as executor:
    hasil = executor.map(proses_satu_gambar, daftar_gambar)
```

- `ProcessPoolExecutor()` — Membuat kumpulan worker process (default: jumlah CPU core)
- `.map(fungsi, iterable)` — Mendistribusikan tugas ke worker-worker secara otomatis
- Mirip dengan `map()` bawaan Python, tapi worker berjalan di **proses terpisah** (paralel nyata)

### Guard `if __name__ == "__main__"`

```python
if __name__ == "__main__":
    main()
```

**WAJIB** saat menggunakan multiprocessing di Python. Tanpa guard ini:

1. Proses anak (child process) akan meng-import ulang script
2. Import ulang → menjalankan kode di level modul
3. Kode tersebut membuat `ProcessPoolExecutor` lagi
4. Terjadi **infinite recursion** → program crash, memory overflow

---

## Referensi

- [Pillow (PIL) Documentation](https://pillow.readthedocs.io/)
- [Python concurrent.futures Documentation](https://docs.python.org/3/library/concurrent.futures.html)
- [Understanding the GIL (Global Interpreter Lock)](https://realpython.com/python-gil/)
- [CuffnCode Repository](https://github.com/Student-Embedded-Control-and-AI-Fest/CuffnCode)

---

## Kontributor

| Nama | Peran |
|------|-------|
| *[Nama Anda]* | Pengembang & Dokumentasi |

> Proyek ini dibuat untuk memenuhi tugas mata kuliah **Parallel Computing**  
> Universitas Komputer Indonesia — Tahun Akademik 2025/2026

---

### 📚 Navigasi Cepat

| | | |
|---|---|---|
| [🏠 Beranda](index.md) | [🔬 Cara Kerja](how-it-works.md) | [💻 Penjelasan Kode](code-walkthrough.md) |
| | [📊 Hasil & Analisis](results.md) | |
