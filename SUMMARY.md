# Summary Project MoMoney

MoMoney adalah aplikasi web untuk membantu pengguna mengelola pengeluaran dari nota, struk, atau invoice. Pengguna dapat login, membuat grup pengeluaran, menentukan kolom data yang ingin dicatat, mengunggah gambar nota, lalu sistem akan mengekstrak informasi penting secara otomatis menjadi data yang lebih rapi dan terstruktur.

Secara umum, proyek ini dibuat sebagai sistem pencatatan pengeluaran berbasis AI. Fokus utamanya bukan hanya menyimpan invoice, tetapi juga mengubah gambar nota yang tidak terstruktur menjadi data tabel yang bisa langsung dipakai untuk pencatatan, pengecekan, dan rekap pengeluaran.

## Fitur Utama

- Autentikasi pengguna dengan session login dan integrasi Google OAuth.
- Dashboard untuk melihat dan mengelola grup pengeluaran.
- Pembuatan grup dengan kolom data yang fleksibel sesuai kebutuhan pengguna.
- Upload gambar nota atau invoice dari frontend.
- OCR untuk membaca teks dari gambar nota.
- Ekstraksi data berbasis AI/LLM untuk mengubah teks OCR menjadi JSON terstruktur.
- Preview hasil ekstraksi sebelum data dimasukkan ke database.
- Penyimpanan data invoice dan hasil ekstraksi ke database.
- Tampilan tabel per grup untuk melihat semua invoice yang sudah diproses.

## Alur Kerja Aplikasi

1. Pengguna login ke aplikasi.
2. Pengguna membuat grup pengeluaran, misalnya untuk belanja, perjalanan, atau kebutuhan organisasi.
3. Pengguna menentukan kolom data yang ingin diekstrak, seperti nama barang, tanggal, harga, total, atau informasi lain.
4. Pengguna mengunggah gambar nota.
5. Backend melakukan preprocessing gambar agar lebih mudah dibaca OCR.
6. Tesseract OCR membaca teks dari gambar dalam bahasa Indonesia dan Inggris.
7. Sistem membangun prompt AI berdasarkan hasil OCR dan schema kolom grup.
8. LLM mengekstrak data sesuai schema dan mengembalikan hasil dalam format JSON.
9. Backend melakukan normalisasi hasil, seperti membersihkan angka harga, memilih tanggal yang paling tepat, dan memastikan field sesuai kolom grup.
10. Frontend menampilkan preview hasil ekstraksi.
11. Setelah dikonfirmasi pengguna, data disimpan sebagai invoice di database.

## Highlight: Spaces Detection pada OCR untuk Improve Prompt AI

Bagian paling penting dari proyek ini adalah proses spaces detection pada hasil OCR. Masalah utama ketika membaca nota adalah OCR biasa hanya menghasilkan teks mentah, sehingga struktur tabel sering hilang. Pada nota atau invoice, jarak antar kata sebenarnya sangat penting karena bisa menunjukkan hubungan antar kolom, misalnya nama item di kiri dan harga di kanan.

Karena itu, backend tidak hanya mengambil output teks dari Tesseract. Sistem juga mengambil data posisi setiap kata menggunakan `pytesseract.image_to_data`, seperti `left`, `top`, `width`, `height`, dan confidence. Data posisi ini kemudian dipakai untuk membangun ulang layout teks agar susunan nota mendekati tampilan visual aslinya.

Proses spaces detection dilakukan dengan cara:

- Mengambil bounding box setiap kata dari OCR.
- Menghapus kata kosong dan kata dengan confidence tidak valid.
- Mengurutkan kata berdasarkan posisi vertikal dan horizontal.
- Mengelompokkan kata ke baris visual berdasarkan posisi tengah vertikal setiap kata.
- Menghitung median tinggi kata untuk menentukan toleransi penggabungan baris.
- Menghitung estimasi median lebar karakter dari `width / jumlah karakter`.
- Mengukur jarak horizontal antar kata dalam satu baris.
- Mengubah gap horizontal tersebut menjadi jumlah spasi yang proporsional.
- Membatasi jumlah spasi maksimum agar hasil tetap mudah dibaca.
- Menghasilkan `reconstructed OCR rows`, yaitu teks OCR yang sudah menjaga jarak antar kolom.

Hasil spaces detection ini penting karena menjadi konteks tambahan untuk AI. Prompt ke LLM tidak hanya berisi `Raw OCR`, tetapi juga `Reconstructed OCR rows`. Dengan begitu, AI bisa memahami bahwa beberapa teks berada di kolom berbeda, bukan sekadar urutan kata biasa.

Contohnya, pada invoice tabel:

```text
Item                 Amount
Logo                 $35.00
Banner               $50.00
Poster               $20.00
Total                $105.00
```

Jika hanya memakai raw OCR, model bisa salah menganggap semua teks sebagai daftar biasa atau hanya mengambil item pertama. Dengan reconstructed rows, prompt dapat menjelaskan bahwa spasi besar menunjukkan struktur tabel. Ini membantu AI membedakan kolom item, harga per item, dan total pembayaran.

Spaces detection kemudian dipakai untuk improve prompt AI dengan beberapa strategi:

- Prompt menyertakan dua sumber: `Raw OCR` dan `Reconstructed OCR rows`.
- Prompt memberi tahu AI bahwa reconstructed rows mempertahankan gap horizontal untuk dokumen berbentuk tabel.
- AI diarahkan untuk membaca semua item row di antara header tabel dan bagian total/payment/note.
- AI diarahkan agar tidak hanya mengambil item pertama ketika ada beberapa item seperti Logo, Banner, dan Poster.
- AI diarahkan untuk mengambil total akhir sebagai price/amount/total, bukan harga per item.
- AI diminta mengembalikan hasil sesuai schema JSON dinamis dari kolom grup.

Model LLM utama yang digunakan adalah `Qwen2.5:1.5b` melalui Ollama. Karena ukuran model ini relatif kecil, kemampuan reasoning dan pemahaman layout-nya tidak sekuat model yang lebih besar. Di sinilah prompt engineering menjadi penting: prompt dibuat sangat eksplisit, konteks OCR diperkaya dengan reconstructed rows, dan aturan output dibuat ketat agar Qwen tetap bisa memahami hubungan antar elemen nota.

Dengan kata lain, sistem ini tidak mengandalkan kecerdasan model saja. Spaces detection membantu mengubah informasi visual menjadi teks yang lebih bermakna, lalu prompt engineering memberi instruksi yang cukup jelas agar `Qwen2.5:1.5b` dapat memahami konteks seperti item, harga, tanggal transaksi, dan total pembayaran meskipun modelnya terbatas.

Dengan pendekatan ini, peningkatan AI tidak hanya dilakukan melalui wording prompt, tetapi juga melalui kualitas konteks yang diberikan ke prompt. Spaces detection membuat input prompt lebih informatif karena struktur visual nota ikut terbawa ke teks. Ini membuat proses ekstraksi lebih akurat, terutama untuk nota yang memiliki format tabel dan alignment harga di sisi kanan.

Selain itu, sistem tetap memiliki deterministic parsing untuk membaca pola tanggal, harga, dan item dari OCR. Jika data deterministic sudah lengkap, backend bisa memakai hasil tersebut langsung. Jika belum lengkap, hasil spaces detection dan prompt AI digunakan untuk membantu LLM memahami struktur nota, lalu hasil akhirnya dinormalisasi kembali agar sesuai schema.

## Komponen AI dan OCR

- Image preprocessing:
  gambar diubah ke grayscale, diperbesar, dibersihkan noise-nya, dipertajam, lalu diberi threshold agar teks lebih mudah dibaca.

- OCR:
  Tesseract membaca teks dengan konfigurasi beberapa mode page segmentation, lalu hasilnya digabung agar teks yang terlewat pada satu mode masih bisa tertangkap pada mode lain.

- Spaces detection dan layout reconstruction:
  hasil OCR berbasis posisi kata disusun ulang menjadi baris visual. Sistem menghitung gap horizontal antar kata dan mengubahnya menjadi spasi proporsional, sehingga tabel nota tetap terbaca oleh AI sebagai tabel, bukan teks datar.

- LLM extraction:
  `Qwen2.5:1.5b` menerima raw OCR, reconstructed OCR rows hasil spaces detection, dan schema JSON, lalu mengembalikan data sesuai kolom yang diminta pengguna. Prompt dibuat detail agar model kecil tetap bisa mengikuti konteks dan format output.

- Normalisasi:
  backend membersihkan hasil AI, mengubah format uang menjadi angka, memilih satu nilai untuk field scalar, dan memastikan hanya kolom yang diminta yang disimpan.

## Tech Stack

### Frontend

- Next.js 16
- React 19
- TypeScript
- Tailwind CSS 4
- Zustand
- Axios
- pnpm/npm

### Backend

- Python 3.12
- FastAPI
- Uvicorn
- SQLModel
- Pydantic
- Authlib
- Starlette Session Middleware
- python-dotenv
- Requests

### Database dan Auth

- NeonDB/PostgreSQL melalui `DATABASE_URL`
- Google OAuth
- Cookie-based session authentication

### AI, OCR, dan Image Processing

- Ollama sebagai local LLM runtime
- Model default `qwen2.5:1.5b`, dipilih sebagai local LLM yang ringan
- OpenAI-compatible chat completions endpoint untuk komunikasi ke Ollama
- Tesseract OCR
- Pytesseract
- Pillow
- OpenCV
- NumPy

### Testing dan Dokumentasi API

- Pytest untuk unit/integration test backend
- Bruno collection untuk testing endpoint API
- OpenAPI schema dari FastAPI

## Struktur Project

```text
momoney-2/
├── backend/            # FastAPI backend, domain routes, services, OCR, AI extraction
├── frontend/           # Next.js frontend, dashboard, upload flow, preview, group pages
├── bruno-test-api/     # Koleksi API testing Bruno
├── docs/               # Static export/documentation build
├── README.md
└── SUMMARY.md
```

## Kesimpulan

MoMoney adalah aplikasi expense management berbasis web yang menggabungkan frontend modern, REST API, database, OCR, dan LLM lokal. Kontribusi utama proyek ini ada pada alur ekstraksi nota berbasis AI, terutama spaces detection pada OCR yang mempertahankan struktur visual nota sebelum dikirim ke prompt AI. Pendekatan ini membuat `Qwen2.5:1.5b`, model lokal yang relatif kecil, tetap mampu memahami konteks nota melalui prompt yang eksplisit dan input OCR yang lebih terstruktur. Dengan kombinasi OCR, spaces detection, layout reconstruction, prompt yang ketat, deterministic parser, dan normalisasi data, sistem dapat mengubah gambar nota menjadi data invoice yang siap disimpan dan dianalisis.
