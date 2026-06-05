# Parallel Image Processing

> **Perbandingan Komputasi Sekuensial vs Paralel** untuk Image Processing menggunakan Python  
> Tugas Mata Kuliah **Parallel Computing**

## 📌 Deskripsi

Program ini mendemonstrasikan **speedup** komputasi paralel dibanding sekuensial dengan memproses gambar (grayscale + blur) menggunakan `ProcessPoolExecutor`.

## 🚀 Cara Menjalankan

```bash
pip install Pillow
# Masukkan .jpg/.png ke folder images/
python parallel_image_processing.py
```

## 📊 Hasil

| Metode | Waktu (rata-rata) |
|--------|:---:|
| Sekuensial | 1.29 detik |
| Paralel | 1.03 detik |
| **Speedup** | **1.25x** |

## 📖 Dokumentasi Lengkap

👉 [**GitHub Pages**](https://valdiirivaldi.github.io/eval3_kompar/)

## 🛠 Teknologi

- Python 3
- Pillow (PIL)
- concurrent.futures.ProcessPoolExecutor
