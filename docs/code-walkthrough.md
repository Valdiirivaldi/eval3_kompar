# Penjelasan Kode Lengkap

> Panduan langkah-demi-langkah untuk memahami setiap baris kode dalam program.

---

## Daftar Isi

- [Struktur Program](#struktur-program)
- [Bagian 1: Import Library](#bagian-1-import-library)
- [Bagian 2: Konstanta Folder](#bagian-2-konstanta-folder)
- [Bagian 3: Fungsi Inti — `proses_satu_gambar()`](#bagian-3-fungsi-inti--proses_satu_gambar)
- [Bagian 4: Fungsi Sekuensial — `proses_sekuensial()`](#bagian-4-fungsi-sekuensial--proses_sekuensial)
- [Bagian 5: Fungsi Paralel — `proses_paralel()`](#bagian-5-fungsi-paralel--proses_paralel)
- [Bagian 6: Fungsi Utama — `main()`](#bagian-6-fungsi-utama--main)
- [Bagian 7: Guard `if __name__ == "__main__"`](#bagian-7-guard-if-__name__--__main__)
- [Kode Lengkap](#kode-lengkap)

---

## Struktur Program

```
parallel_image_processing.py
│
├── 1. IMPORT LIBRARY
│   ├── os                     ← Operasi file system
│   ├── time                   ← Mengukur waktu eksekusi
│   ├── pathlib.Path           ← Manipulasi path modern
│   ├── PIL.Image              ← Membaca & manipulasi gambar
│   ├── PIL.ImageFilter        ← Filter untuk gambar
│   └── concurrent.futures     ← ProcessPoolExecutor
│
├── 2. KONSTANTA FOLDER
│   ├── FOLDER_INPUT           ← images/
│   ├── FOLDER_SEKUENSIAL      ← hasil_sekuensial/
│   └── FOLDER_PARALEL         ← hasil_paralel/
│
├── 3. FUNGSI INTI
│   └── proses_satu_gambar()   ← Buka → Grayscale → Blur → Simpan
│
├── 4. EKSEKUSI SEKUENSIAL
│   └── proses_sekuensial()    ← Loop for biasa
│
├── 5. EKSEKUSI PARALEL
│   └── proses_paralel()       ← ProcessPoolExecutor.map()
│
├── 6. FUNGSI UTAMA
│   └── main()                 ← Orchestrator
│
└── 7. GUARD
    └── if __name__ == ...     ← Mencegah infinite loop
```

---

## Bagian 1: Import Library

```python
import os
import time
from pathlib import Path

from PIL import Image, ImageFilter
from concurrent.futures import ProcessPoolExecutor, as_completed
```

### `os`
Library standar Python untuk berinteraksi dengan sistem operasi — membuat folder, mengecek path, dll.

### `time`
Digunakan untuk mengukur waktu eksekusi. Fungsi `time.perf_counter()` memberikan waktu dengan presisi tinggi (hingga nanosecond).

### `pathlib.Path`
Cara modern (Python 3.4+) untuk memanipulasi path file system. Lebih bersih daripada `os.path.join()`.

```python
# ❌ Cara lama
path = os.path.join("images", "foto.jpg")

# ✅ Cara baru (pathlib)
path = Path("images") / "foto.jpg"
```

### `PIL.Image` dan `PIL.ImageFilter`
Dari library **Pillow** — library image processing paling populer di Python.

- `Image.open()` — membaca file gambar
- `Image.convert("L")` — konversi ke grayscale
- `Image.filter()` — menerapkan filter

### `concurrent.futures`
Library standar Python untuk asynchronous execution. Berisi:

- **`ProcessPoolExecutor`** — Menjalankan tugas di beberapa proses (untuk CPU-bound task)
- **`ThreadPoolExecutor`** — Menjalankan tugas di beberapa thread (untuk I/O-bound task)
- **`as_completed()`** — (opsional) Mengambil hasil tugas sesuai urutan selesai

---

## Bagian 2: Konstanta Folder

```python
FOLDER_INPUT = Path("images")
FOLDER_SEKUENSIAL = Path("hasil_sekuensial")
FOLDER_PARALEL = Path("hasil_paralel")
```

Menggunakan huruf kapital (konvensi Python untuk konstanta) agar mudah dikenali. Dengan `Path()`, path otomatis kompatibel dengan semua sistem operasi:

| OS | Path |
|----|------|
| Windows | `images\foto.jpg` |
| Linux/macOS | `images/foto.jpg` |

---

## Bagian 3: Fungsi Inti — `proses_satu_gambar()`

```python
def proses_satu_gambar(nama_file: str) -> str:
    path_input = FOLDER_INPUT / nama_file

    # Langkah 1: Buka gambar
    gambar = Image.open(path_input)

    # Langkah 2: Konversi ke Grayscale (mode 'L')
    gambar_grayscale = gambar.convert("L")

    # Langkah 3: Terapkan filter Blur
    gambar_blur = gambar_grayscale.filter(ImageFilter.BLUR)

    # Langkah 4: Simpan ke kedua folder output
    gambar_blur.save(FOLDER_SEKUENSIAL / nama_file)
    gambar_blur.save(FOLDER_PARALEL / nama_file)

    # Langkah 5: Tutup file gambar (free memory)
    gambar.close()

    return nama_file
```

### Type Hint

```python
def proses_satu_gambar(nama_file: str) -> str:
```

`nama_file: str` menunjukkan parameter harus string.
`-> str` menunjukkan fungsi mengembalikan string.

### Alur Pemrosesan

```
Input: foto.jpg
           ↓
   Image.open("images/foto.jpg")        ← Baca file asli
           ↓
   .convert("L")                         ← Ubah ke hitam-putih (grayscale)
           ↓
   .filter(ImageFilter.BLUR)             ← Haluskan gambar (blur)
           ↓
   .save("hasil_sekuensial/foto.jpg")    ← Simpan untuk sequential
   .save("hasil_paralel/foto.jpg")       ← Simpan untuk parallel
```

### Kenapa fungsi ini penting?

Fungsi ini adalah **satu-satunya fungsi yang memproses gambar**. Baik metode sekuensial maupun paralel memanggil fungsi yang **sama persis**. Ini memastikan:

- Perbandingan waktu yang **adil** (apple-to-apple)
- Tidak ada perbedaan kualitas hasil
- Kode lebih **mudah di-maintain**

---

## Bagian 4: Fungsi Sekuensial — `proses_sekuensial()`

```python
def proses_sekuensial(daftar_gambar: list) -> float:
    print("[SEKUENSIAL] Memproses gambar secara berurutan...")
    waktu_mulai = time.perf_counter()

    for gambar in daftar_gambar:
        proses_satu_gambar(gambar)
        print(f"  -> {gambar} selesai")

    waktu_selesai = time.perf_counter()
    total_waktu = waktu_selesai - waktu_mulai

    return total_waktu
```

### Cara Kerja

```
waktu_mulai = 0.0s

Loop:
  Iterasi 1: proses gambar1 (0.0s - 0.2s) → selesai
  Iterasi 2: proses gambar2 (0.2s - 0.4s) → selesai
  Iterasi 3: proses gambar3 (0.4s - 0.6s) → selesai
  Iterasi 4: proses gambar4 (0.6s - 0.8s) → selesai

waktu_selesai = 0.8s
total_waktu = 0.8s - 0.0s = 0.8s
```

CPU hanya mengerjakan satu gambar dalam satu waktu. Waktu total = **penjumlahan** waktu setiap gambar.

### `time.perf_counter()`

Mengembalikan waktu dalam detik dengan presisi tinggi. Berbeda dengan `time.time()`:

| Fungsi | Presisi | Kegunaan |
|--------|---------|----------|
| `time.time()` | Detik | Waktu kalender |
| `time.perf_counter()` | Nanosecond | **Benchmarking** ✅ |
| `time.process_time()` | Nanosecond | CPU time (tidak termasuk sleep) |

---

## Bagian 5: Fungsi Paralel — `proses_paralel()`

```python
def proses_paralel(daftar_gambar: list) -> float:
    print("[PARALEL] Memproses gambar secara paralel...")
    waktu_mulai = time.perf_counter()

    with ProcessPoolExecutor() as executor:
        hasil = executor.map(proses_satu_gambar, daftar_gambar)

    for nama in hasil:
        print(f"  -> {nama} selesai")

    waktu_selesai = time.perf_counter()
    total_waktu = waktu_selesai - waktu_mulai

    return total_waktu
```

### Cara Kerja Detail

#### 1. `with ProcessPoolExecutor() as executor:`

Statement `with` membuat **context manager** yang:
- Secara otomatis membuat worker processes saat masuk blok `with`
- Secara otomatis menutup worker processes saat keluar blok `with`
- Jumlah worker default = `os.cpu_count()` (jumlah core CPU)

#### 2. `executor.map(fungsi, iterable)`

Method `.map()` adalah jantung dari paralelisasi. Cara kerjanya:

```
Input: [gbr1, gbr2, gbr3, gbr4, gbr5, gbr6, gbr7, gbr8]
                             ↓
                ProcessPoolExecutor (4 workers)
                             ↓
        ┌───────┬───────┬───────┬───────┐
        │ Wkr 1 │ Wkr 2 │ Wkr 3 │ Wkr 4 │
        ├───────┼───────┼───────┼───────┤
        │ gbr1  │ gbr2  │ gbr3  │ gbr4  │
        │ gbr5  │ gbr6  │ gbr7  │ gbr8  │
        └───────┴───────┴───────┴───────┘
                             ↓
Output: [gbr1, gbr2, ..., gbr8]  ← urutannya SAMA dengan input
```

#### 3. Perbedaan `map()` vs `submit()`

| Method | Kegunaan | Kelebihan |
|--------|----------|-----------|
| **`executor.map()`** | Memanggil fungsi yang sama untuk setiap item | Sederhana, hasil berurutan ✅ |
| **`executor.submit()`** | Memanggil fungsi dengan argumen berbeda | Lebih fleksibel, bisa `as_completed()` |

### Timeline Perbandingan

```
SEKUENSIAL (1 core):
Core 1: [gbr1][gbr2][gbr3][gbr4][gbr5][gbr6][gbr7][gbr8]
         ↑                                       ↑
        0.0s                                    0.8s
        Total: 0.8 detik

PARALEL (4 core):
Core 1: [gbr1][gbr5]
Core 2: [gbr2][gbr6]
Core 3: [gbr3][gbr7]
Core 4: [gbr4][gbr8]
         ↑              ↑
        0.0s           0.2s
        Total: ~0.2 detik
```

Dalam ilustrasi di atas, paralel **4x lebih cepat**!

---

## Bagian 6: Fungsi Utama — `main()`

```python
def main():
```

Fungsi ini mengatur seluruh alur program:

```
main()
  │
  ├── Cek folder images/           → Error jika tidak ada
  │
  ├── Kumpulkan daftar gambar      → Filter .jpg/.png
  │
  ├── Buat folder output           → mkdir(parents=True, exist_ok=True)
  │
  ├── Panggil proses_sekuensial()  → Catat waktu
  │
  ├── Panggil proses_paralel()     → Catat waktu
  │
  └── Tampilkan kesimpulan         → Hitung speedup
```

### Error Handling

```python
if not FOLDER_INPUT.exists():
    print(f"[ERROR] Folder '{FOLDER_INPUT}' tidak ditemukan!")
    return
```

```python
ekstensi_diizinkan = {".jpg", ".jpeg", ".png"}
daftar_gambar = [
    f.name for f in FOLDER_INPUT.iterdir()
    if f.is_file() and f.suffix.lower() in ekstensi_diizinkan
]

if not daftar_gambar:
    print(f"[ERROR] Folder '{FOLDER_INPUT}' kosong!")
    return
```

### Membuat Folder Output

```python
FOLDER_SEKUENSIAL.mkdir(parents=True, exist_ok=True)
FOLDER_PARALEL.mkdir(parents=True, exist_ok=True)
```

- `parents=True` — Membuat folder induk jika belum ada
- `exist_ok=True` — Tidak error jika folder sudah ada

---

## Bagian 7: Guard `if __name__ == "__main__"`

```python
if __name__ == "__main__":
    main()
```

### Apa yang Terjadi Tanpa Guard?

Saat Python menjalankan script:

```
1. python parallel_image_processing.py
       ↓
2. Python membaca seluruh file
       ↓
3. Kode di level modul DIEKSEKUSI
       ↓
4. if __name__ == "__main__" → True → jalankan main()
       ↓
5. main() membuat ProcessPoolExecutor
       ↓
6. ProcessPoolExecutor membuat PROSES ANAK
       ↓
7. Proses anak meng-import file ini
       ↓
8. Tanpa guard → kode level modul dijalankan LAGI
       ↓
9. Proses anak juga membuat ProcessPoolExecutor
       ↓
10. Terjadi INFINITE RECURSION → CRASH! 💥
```

### Visualisasi Masalah

```
Tanpa Guard:
┌─ Proses Induk ─────────────────────────────────────┐
│  main()                                             │
│    └─ ProcessPoolExecutor                           │
│         └─ Anak 1: import file → main() lagi!       │
│              └─ ProcessPoolExecutor                  │
│                   └─ Cucu 1: import file → main()!   │
│                        └─ ProcessPoolExecutor        │
│                             └─ ... INFINITE LOOP     │
└─────────────────────────────────────────────────────┘

Dengan Guard:
┌─ Proses Induk ─────────────────────────────────────┐
│  __name__ == "__main__" → True → main()             │
│    └─ ProcessPoolExecutor                            │
│         └─ Anak 1: __name__ ≠ "__main__" → STOP ✅  │
│         └─ Anak 2: __name__ ≠ "__main__" → STOP ✅  │
└─────────────────────────────────────────────────────┘
```

### Kenapa Lebih Berbahaya di Windows?

- **Linux/macOS**: Menggunakan `fork()` — proses anak mewarisi memory dari induk
- **Windows**: Menggunakan `spawn()` — membuat proses baru dari nol, meng-import ulang semua file

Karena Windows harus meng-import ulang, tanpa guard akan lebih cepat crash.

---

## Ringkasan

| Bagian | Fungsi | Baris Kunci |
|--------|--------|-------------|
| **Import** | Memuat library | `from concurrent.futures import ProcessPoolExecutor` |
| **Konstanta** | Definisi folder | `FOLDER_INPUT = Path("images")` |
| **Fungsi Inti** | Proses 1 gambar | `gambar.convert("L").filter(ImageFilter.BLUR)` |
| **Sekuensial** | Loop biasa | `for gambar in daftar_gambar:` |
| **Paralel** | Panggil paralel | `executor.map(proses_satu_gambar, daftar_gambar)` |
| **main()** | Orchestrator | Validasi → Proses → Kesimpulan |
| **Guard** | Safety | `if __name__ == "__main__":` |

---

[← Kembali ke Cara Kerja](how-it-works.md) | [Lanjut ke Hasil & Analisis →](results.md)
