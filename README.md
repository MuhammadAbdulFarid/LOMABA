# AksaraVision

Web pelestarian dan pembelajaran aksara Nusantara (fokus MVP: **Aksara Lontara** dan **Aksara Jawa**) dengan AI Vision dari Google Gemini.

| Rute | Fitur |
| --- | --- |
| `/` | Landing page |
| `/lens` | **Smart Lontar Lens** — unggah gambar naskah → deteksi aksara, transliterasi per suku kata, makna, catatan budaya, dan pelafalan (Web Speech API) |
| `/dojo` | **Kanvas Goresan** — latihan menulis 43 aksara dasar di kanvas bergrid, dinilai "AI Guru Aksara" |
| `/explore` | **Peta Aksara Nusantara** — peta interaktif persebaran 9 aksara daerah, sejarah singkat, contoh huruf Unicode, dan silsilah Brahmi → Kawi → aksara daerah |

## Menjalankan

```bash
npm install
cp .env.example .env.local   # lalu isi GEMINI_API_KEY
npm run dev                  # http://localhost:3000
```

> **Windows/PowerShell:** jangan membuat `.env.local` dengan `echo ... > .env.local` — PowerShell 5 menyimpannya sebagai UTF-16 sehingga tidak terbaca Next.js. Salin `.env.example` atau simpan sebagai UTF-8.

Perintah lain: `npm run lint`, `npm run build`, `npm start`.

### Variabel lingkungan

| Nama | Wajib | Keterangan |
| --- | --- | --- |
| `GEMINI_API_KEY` | Ya | Kunci dari [Google AI Studio](https://aistudio.google.com/apikey). Hanya dibaca di server. |
| `GEMINI_MODEL` | Tidak | Model utama, bawaan `gemini-2.5-flash`. |

Bila model utama ditolak (kunci baru tidak lagi mendapat akses `gemini-2.5-flash`), sedang sibuk, atau kuotanya habis, server otomatis beralih ke `gemini-3.6-flash` → `gemini-3.8-flash` → `gemini-3.5-flash-lite` (model *lite* hanya untuk penilaian goresan). Model yang benar-benar menjawab dikirim lewat header `X-AI-Model`.

## API

Semua respons berupa JSON murni sesuai PRD §4. Galat selalu berbentuk `{ "error": string, "code": string }`.

### `POST /api/vision-ocr`

```jsonc
// Permintaan
{ "image": "data:image/jpeg;base64,..." }
// Respons
{ "detectedScript": "Lontara (Bugis)", "latinTransliteration": "si-pa-ka-ta-u", "indonesianMeaning": "...", "culturalNotes": "...", "confidence": 0.95 }
```

### `POST /api/evaluate-stroke`

```jsonc
// Permintaan — reference & similarity opsional
{ "image": "data:image/png;base64,...", "reference": "data:image/png;base64,...", "similarity": 0.86, "target": { "script": "lontara", "latin": "ka" } }
// Respons
{ "score": 86, "isPassed": true, "feedback": "...", "tips": "..." }
```

Skor akhir = 50% penilaian visual AI + 50% kemiripan bentuk yang diukur di perangkat (`similarity`); lulus bila skor ≥ 70.

## Ketahanan demo

- **Sampel naskah** (3 rekonstruksi digital dengan font Unicode) punya hasil kurasi; bila AI lambat/gagal, hasil kurasi ditampilkan dengan label *Mode demo*.
- **Dojo** menilai goresan di perangkat bila AI tidak tersedia, dengan label *Penilaian lokal*.
- Route Handler membatasi ukuran body, menolak permintaan lintas situs, dan membatasi 20 permintaan/menit per IP.

## Struktur

```
app/              halaman (/, /lens, /dojo, /explore) & Route Handler (app/api/*)
components/       komponen UI arsip, studio Lens & Dojo, atlas /explore
hooks/            use-stroke-canvas (HTML5 Canvas API + Pointer Events)
lib/              data aksara, kontrak API, klien Gemini, utilitas kanvas & gambar
lib/geo/          peta Indonesia (path SVG dari Natural Earth 1:50m, domain publik)
```