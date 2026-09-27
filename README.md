# SehatIn

Aplikasi gizi preventif berbasis pangan lokal Indonesia.

Dibangun untuk **Garuda Hacks 7.0**.

---

## Masalah

- Banyak orang tidak tahu kebutuhan kalori dan gizi harian mereka
- Rencana makan tidak dipersonalisasi berdasarkan kondisi tubuh dan tujuan
- Aplikasi gizi yang ada menggunakan bahan makanan barat, bukan Indonesia
- Kurangnya panduan olahraga yang terintegrasi dengan target nutrisi

## Fitur

| Fitur | Fungsi |
|---|---|
| **Asesmen Kesehatan** | Hitung BMI, BMR, TDEE, target protein/karbo/lemak/serat/gula/air |
| **Rekomendasi AI** | AI pilih bahan makanan + buat resep sesuai target nutrisi |
| **Ask — Koki Virtual** | Chat dengan AI: sebut bahan yang ada, dapat resep instan |
| **Muscle Scan** | Peta otot interaktif + panduan latihan + video tutorial |
| **Katalog Makanan** | 50+ bahan pangan lokal Indonesia lengkap dengan nilai gizi |
| **Riwayat Asesmen** | Semua hasil tersimpan, bisa dilihat dan dihapus |
| **Profil** | Data kesehatan, upload foto, dark/light mode |

## Alur Pakai

```
Buka Aplikasi → Register/Login → Dashboard
  ├── Asesmen Kesehatan → isi data → lihat hasil + rekomendasi AI
  ├── Ask → chat resep berdasarkan bahan yang ada
  ├── Muscle Scan → tap otot → lihat latihan
  ├── History → lihat riwayat asesmen
  └── Profile → edit data diri, ganti foto, logout
```

## Tech Stack

| Bagian | Teknologi |
|---|---|
| Frontend | React 19, Vite 8, Tailwind CSS v4, Lucide React |
| Backend | Supabase (Auth, PostgreSQL, Storage, Edge Functions) |
| Edge Functions | Deno (rule-engine, gemini-proxy) |
| AI | Groq Cloud — Llama 3.3 70B |
| Deploy | Vercel (frontend), Supabase (backend & functions) |
| Database | PostgreSQL — 5 tabel: profiles, macro_targets, food_catalog, food_recommendations, hasil_rekomendasi |

## Arsitektur

```
[React] ──supabase.auth──→ [Supabase Auth]
   │
   ├──supabase.functions.invoke──→ [rule-engine] (Deno)
   │   Hitung BMI/BMR/TDEE/makro, simpan ke DB
   │
   ├──supabase.functions.invoke──→ [gemini-proxy] (Deno)
   │   Kirim data ke Groq API → Llama 3.3 → rekomendasi + resep
   │
   └──supabase.from().select()──→ [PostgreSQL]
       profiles, macro_targets, food_catalog, ...
```

## Cara Jalankan

```bash
npm install
npm run dev          # Vite dev server (localhost:5173)

# Edge Functions lokal (butuh supabase CLI)
npx supabase functions serve

# Deploy
npm run build        # Build production
npx supabase functions deploy rule-engine
npx supabase functions deploy gemini-proxy
```

## Environment Variables

Buat file `.env` di root:

```
VITE_SUPABASE_URL=...
VITE_SUPABASE_ANON_KEY=...
```

Buat file `backend/supabase/.env` untuk local functions:

```
GROQ_API_KEY=...
```
