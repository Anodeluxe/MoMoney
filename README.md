# MoMoney

MoMoney adalah aplikasi web untuk mengelola dan merapikan pengeluaran dari nota/struk. Pengguna dapat login, membuat grup pengeluaran, mengunggah nota, lalu sistem akan menjalankan OCR dan LLM lokal melalui Ollama untuk mengekstrak informasi nota menjadi data pengeluaran yang lebih terstruktur.

Proyek ini terdiri dari tiga bagian utama:

- `frontend`: aplikasi web Next.js untuk dashboard, login, grup, dan upload nota.
- `backend`: REST API FastAPI untuk autentikasi, database, grup, invoice, OCR, dan ekstraksi AI.
- `ollama`: service LLM lokal yang dipakai backend untuk memproses hasil OCR nota.

## Anggota Kelompok

- Kelompok 12
- Ketua Kelompok: AZFANOVA SAMMY RAFIF SAPUTRA - 521764
- Anggota 1: Polikarpus Arya Pradhanika - 512404
- Anggota 2: Gabriele Ghea De Palma - 512218
- Anggota 3: AZFANOVA SAMMY RAFIF SAPUTRA - 521764

## Tech Stack

### Frontend

- Next.js 16: framework React untuk aplikasi web.
- React 19: library UI utama.
- TypeScript: static typing untuk kode frontend.
- Tailwind CSS 4: styling UI.
- Zustand: state management ringan di sisi client.
- Axios dan Fetch API: komunikasi HTTP ke backend.
- pnpm/npm: package manager untuk menjalankan project frontend.

### Backend

- Python 3.12: runtime backend.
- FastAPI: framework REST API.
- Uvicorn: ASGI server untuk menjalankan FastAPI.
- SQLModel: ORM/model database berbasis SQLAlchemy dan Pydantic.
- NeonDb: database aplikasi melalui `DATABASE_URL`.
- Authlib: integrasi Google OAuth.
- Starlette Session Middleware: session login berbasis cookie.
- python-dotenv: membaca konfigurasi dari file `.env`.
- Pytesseract, Pillow, OpenCV, NumPy: OCR dan preprocessing gambar nota.
- Requests: request HTTP ke Ollama.

### AI dan OCR

- Ollama: menjalankan model LLM secara lokal.
- Model default: `qwen2.5:1.5b`.
- Tesseract OCR: membaca teks dari gambar nota, dengan bahasa Indonesia dan Inggris.

## Struktur Folder

```text
momoney-2/
+-- backend/            # FastAPI backend
+-- frontend/           # Next.js frontend
+-- bruno-test-api/     # Koleksi API testing Bruno
+-- docs/               # Static export/frontend docs
+-- README.md
```

## Prasyarat Local Server

Pastikan software berikut sudah tersedia:

- Node.js 20 atau lebih baru.
- pnpm, atau npm jika ingin memakai `package-lock.json`.
- Python 3.12.
- uv atau pip/venv untuk dependency Python.
- Tesseract OCR.
- Ollama.
- Database NeonDb untuk development sederhana.

Instalasi Tesseract di Ubuntu/WSL:

```bash
sudo apt update
sudo apt install tesseract-ocr tesseract-ocr-ind
```

## Environment Variables

Jangan commit file `.env` yang berisi value asli. Buat file env lokal berdasarkan daftar key berikut.

### Backend: `backend/.env`

```env
# Database
DATABASE_URL=

# Session
SESSION_SECRET_KEY=

# Google OAuth
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Frontend callback/redirect URL
FRONTEND_URL=http://localhost:3000

# Ollama / LLM
OLLAMA_URL=http://localhost:11434/v1
LLM_MODEL=qwen2.5:1.5b

# Optional debugging
EXTRACTION_DEBUG=false
```

Keterangan:

- `DATABASE_URL`: URL koneksi database. Contoh format lokal:
  - SQLite: `sqlite:///./momoney.db`
  - PostgreSQL: `postgresql://USER:PASSWORD@localhost:5432/DB_NAME`
- `SESSION_SECRET_KEY`: secret acak untuk menandatangani session cookie.
- `GOOGLE_CLIENT_ID` dan `GOOGLE_CLIENT_SECRET`: credential OAuth dari Google Cloud Console.
- `FRONTEND_URL`: URL frontend lokal yang dipakai redirect setelah login OAuth.
- `OLLAMA_URL`: base URL API Ollama yang kompatibel dengan OpenAI API.
- `LLM_MODEL`: nama model Ollama yang akan dipakai backend.
- `EXTRACTION_DEBUG`: set `true` jika ingin log debug proses ekstraksi.

### Frontend: `frontend/.env.local`

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

Keterangan:

- `NEXT_PUBLIC_API_URL`: URL backend yang dipanggil frontend. Untuk local development gunakan `http://localhost:8000`.

## Cara Menjalankan Ollama Local

Jalankan Ollama server:

```bash
ollama serve
```

Di terminal lain, download model yang dipakai aplikasi:

```bash
ollama pull qwen2.5:1.5b
```

Cek model sudah tersedia:

```bash
ollama list
```

Jika menggunakan model lain, ubah `LLM_MODEL` di `backend/.env` sesuai nama model tersebut.

## Cara Menjalankan Backend Local

Masuk ke folder backend:

```bash
cd backend
```

Buat manual file `backend/.env` memakai daftar key pada bagian Environment Variables. Jangan isi README dengan value asli credential.

### Opsi A: Menggunakan uv

Install dependency:

```bash
uv sync
```

Jalankan backend:

```bash
uv run uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

### Opsi B: Menggunakan venv dan pip

Buat virtual environment:

```bash
python -m venv .venv
```

Aktifkan virtual environment:

```bash
source .venv/bin/activate
```

Install dependency:

```bash
pip install -r requirements.txt
```

Jalankan backend:

```bash
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

Backend akan berjalan di:

```text
http://localhost:8000
```

Cek health check:

```bash
curl http://localhost:8000/health
```

Response yang diharapkan:

```json
{ "status": "ok" }
```

## Cara Menjalankan Frontend Local

Masuk ke folder frontend:

```bash
cd frontend
```

Buat file `.env.local`:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

Install dependency dengan pnpm:

```bash
pnpm install
```

Jalankan frontend:

```bash
pnpm dev
```

Alternatif dengan npm:

```bash
npm install
npm run dev
```

Frontend akan berjalan di:

```text
http://localhost:3000
```

## Urutan Run Local yang Disarankan

Jalankan setiap service di terminal terpisah.

1. Jalankan Ollama:

```bash
ollama serve
```

2. Pastikan model tersedia:

```bash
ollama pull qwen2.5:1.5b
```

3. Jalankan backend:

```bash
cd backend
uv run uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

4. Jalankan frontend:

```bash
cd frontend
pnpm dev
```

5. Buka aplikasi:

```text
http://localhost:3000
```

## Endpoint Utama Backend

Beberapa endpoint yang tersedia:

- `GET /health`: cek status backend.
- `POST /test-login`: login test user tanpa Google OAuth.
- `GET /auth/google`: mulai login Google OAuth.
- `GET /auth/callback`: callback Google OAuth.
- `POST /auth/logout`: logout.
- `GET /users/`: ambil user session saat ini.
- `GET /groups/`: list grup.
- `POST /groups/`: buat grup.
- `GET /groups/{group_id}`: detail grup.
- `PATCH /groups/{group_id}`: update grup.
- `DELETE /groups/{group_id}`: hapus grup.
- `POST /groups/{group_id}/upload-receipt`: upload nota ke grup.
- `GET /invoices/`: list invoice.
- `GET /invoices/{invoice_id}`: detail invoice.
- `PATCH /invoices/{invoice_id}`: update invoice.
- `DELETE /invoices/{invoice_id}`: hapus invoice.
- `POST /invoices/{invoice_id}/upload-receipt`: upload nota untuk invoice.
- `POST /invoices/{invoice_id}/extractions`: buat ekstraksi invoice.
- `GET /invoices/{invoice_id}/extractions`: ambil hasil ekstraksi invoice.

## Menjalankan dengan Docker

### Backend Docker

```bash
cd backend
docker build -t momoney-backend:latest .
docker run -d --name momoney-backend -p 8000:8000 --env-file .env momoney-backend:latest
```

Lihat log:

```bash
docker logs -f momoney-backend
```

Stop container:

```bash
docker stop momoney-backend
docker rm momoney-backend
```

### Frontend Docker

```bash
cd frontend
docker build --build-arg NEXT_PUBLIC_API_URL=http://localhost:8000 -t momoney-frontend:latest .
docker run -d --name momoney-frontend -p 3000:3000 momoney-frontend:latest
```

Lihat log:

```bash
docker logs -f momoney-frontend
```

Stop container:

```bash
docker stop momoney-frontend
docker rm momoney-frontend
```

## Testing

### Backend Tests

```bash
cd backend
uv run pytest
```

Atau jika menggunakan venv/pip:

```bash
cd backend
pytest
```

### Frontend Lint

```bash
cd frontend
pnpm lint
```

## Troubleshooting

### Frontend gagal memanggil backend

Pastikan `frontend/.env.local` berisi:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

Restart `pnpm dev` setelah mengubah env.

### Backend error `DATABASE_URL environment variable not set`

Pastikan `backend/.env` sudah dibuat dan berisi `DATABASE_URL`.

Untuk development cepat bisa memakai SQLite:

```env
DATABASE_URL=sqlite:///./momoney.db
```

### Ekstraksi AI gagal karena Ollama

Pastikan Ollama berjalan dan model tersedia:

```bash
ollama serve
ollama list
```

Pastikan `backend/.env` memakai URL berikut:

```env
OLLAMA_URL=http://localhost:11434/v1
LLM_MODEL=qwen2.5:1.5b
```

### OCR tidak membaca nota

Pastikan Tesseract dan language pack Indonesia sudah terinstal:

```bash
tesseract --version
tesseract --list-langs
```

Minimal language yang dibutuhkan:

```text
eng
```
