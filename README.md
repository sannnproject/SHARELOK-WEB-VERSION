# Sherlock Web

Sherlock Web adalah antarmuka web modern untuk [Sherlock Project](https://github.com/sherlock-project/sherlock), yaitu tools OSINT untuk mencari username di berbagai platform media sosial.

Aplikasi ini menggunakan teknologi modern:
- **Frontend**: Next.js 15 (App Router), Tailwind CSS, shadcn/ui.
- **Backend**: FastAPI (Python), terintegrasi langsung dengan Sherlock menggunakan `asyncio.create_subprocess_exec`.

## Fitur Utama

- **Pencarian Cepat**: Pencarian non-blocking yang aman karena dijalankan via subprocess (bukan shell=True).
- **Statistik & Filter**: Ringkasan hasil (Found, Not Found, Unknown, Waktu Pencarian) beserta filter tabs.
- **Export Data**: Unduh hasil pencarian dalam format JSON atau CSV.
- **Mode Gelap / Terang**: Mendukung Dark Mode bergaya GitHub.
- **Desain Modern**: Skeleton loading, animasi halus dengan Tailwind.

## Cara Menjalankan Secara Lokal

### Prasyarat
- Node.js (minimal v18)
- Python (minimal v3.8)
- `pip`

### 1. Instalasi Frontend (Node.js)

```bash
# Install dependencies Node.js
npm install
```

### 2. Instalasi Backend (Python)

Karena ini menggunakan integrasi dengan script Python secara langsung, kamu butuh menginstall dependensi Python:

```bash
# Install dependencies Python
pip install -r requirements.txt
```

### 3. Menjalankan Server

Ketika dideploy di Vercel, frontend Next.js dan backend FastAPI berjalan bersamaan. Namun secara lokal, kamu dapat menjalankannya dengan cara berikut.

Untuk UI/Frontend (Next.js):
```bash
npm run dev
```

Untuk mencoba API FastAPI (Backend) secara terpisah:
```bash
uvicorn api.index:app --reload --port 8000
```
> Catatan: Pada mode pengembangan (development preview UI), aplikasi mungkin menggunakan file mock API di `app/api/search/route.ts` jika Python tidak tersedia di environment NodeJS. Untuk deployment production di Vercel, file `vercel.json` akan memprioritaskan runtime Python.

## Deploy ke Vercel

Aplikasi ini dirancang khusus agar mudah dideploy di Vercel **tanpa VPS** berkat konfigurasi Python Runtime Vercel di `vercel.json`.

1. Buat akun di [Vercel](https://vercel.com/)
2. Install Vercel CLI (opsional) atau hubungkan repository GitHub kamu ke Vercel.
3. Deploy proyek ini. Vercel akan otomatis membaca file `package.json` untuk Frontend, dan file `vercel.json` serta `requirements.txt` untuk Backend (FastAPI).

### Konfigurasi `vercel.json`
Vercel secara otomatis akan me-rewrite semua request ke `/api/*` untuk dieksekusi oleh runtime Python pada file `api/index.py`.

```json
{
  "rewrites": [
    {
      "source": "/api/(.*)",
      "destination": "/api/index.py"
    }
  ]
}
```

## Environment & Keamanan
Aplikasi ini tidak menggunakan database (Supabase/lainnya) dan langsung memproses request.
Pastikan tidak mengekspos environment variable sembarangan. Sherlock dipanggil menggunakan subprocess argumen (list array argumen Python) untuk mencegah OS command injection.

## Keamanan & Rate Limit
- **Rate Limit**: API dapat dibatasi sesuai kebutuhan dengan menambah middleware Rate Limit (Secara default, Vercel Edge juga memiliki limitasi DDOS/Spam).
- **Validasi Input**: Username divalidasi menggunakan Regex `^[a-zA-Z0-9._-]{2,30}$` untuk mencegah karakter ilegal.
