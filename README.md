# API Contract — AI Service (A26-CS300: Toko Listrik Bina Harja)

## 1. Informasi Umum
- **Base URL (Lokal):** `http://localhost:8000` (atau port FastAPI/Flask yang disepakati)
- **Format Data:** `application/json`
- **Pihak Terkait:** Gen AI (Zayyana Maulida & Najib Alfa Rojaya), Full-Stack (Tutur Pryambadha), Data Science (Az-Zahrawani & Muhammad Sidiq Fathurochman)

## 2. Aturan & Konvensi Bersama
- Penamaan field menggunakan `snake_case` secara konsisten di seluruh request dan response.
- Nama produk mengacu pada daftar/skema produk kanonikal alat listrik yang disepakati tim.
- Perhitungan harga final, pengurangan stok, dan penyimpanan transaksi utama diproses oleh Backend Express.js; AI Service fokus pada ekstraksi/pemrosesan teks dan respons AI.
- Jika layanan AI lambat, gagal, atau terkena batas kuota, Backend/Frontend tetap dapat melanjutkan alur melalui fallback kasir manual.

## 3. Daftar Endpoint
### 3.1 Health Check
- **Method & Path:** `GET /health`
- **Tujuan:** Memeriksa status aktif layanan AI sebelum/saat integrasi.
- **Response (`200 OK`):**
  - `status`: `"ok"`
  - `service`: string nama layanan

### 3.2 Ekstraksi Transaksi dari Teks Bebas (Smart Text-to-Transaction)
- **Method & Path:** `POST /api/v1/ai/parse-transaction`
- **Tujuan:** Mengubah catatan teks kasir menjadi draf item transaksi terstruktur (nama barang, jumlah, metode bayar/nominal jika ada), serta mendeteksi input kurang lengkap atau di luar konteks usaha.
- **Request Body:**
  - `raw_text` (string, wajib): teks catatan transaksi dari kasir.
  - `store_context` (object, opsional): informasi konteks toko dan metode pembayaran yang diizinkan (misalnya `Tunai`, `QRIS`).
- **Response (`200 OK`):**
  - `status`: `"parsed"` | `"incomplete"` | `"out_of_scope"`
  - `is_complete` (boolean)
  - `needs_confirmation` (boolean)
  - `message` (string): pesan panduan/notifikasi untuk ditampilkan ke pengguna.
  - `data` (object | null): hasil ekstraksi (`transaction_type`, `payment_method`, `items`, `total_amount`).
  - `missing_fields` (array of string): daftar field yang belum lengkap (jika `status: "incomplete"`).
  - `fallback_to_manual` (boolean)

### 3.3 Endpoint Fitur Tambahan AI (Disiapkan untuk Minggu 3)
- **Method & Path:** `POST /api/v1/ai/assistant` (atau penamaan endpoint RAG/tanya jawab produk yang disepakati)
- **Tujuan:** Menangani pertanyaan/bantuan terkait produk alat listrik dengan guardrails penolakan sopan untuk pertanyaan di luar konteks usaha.
- **Request Body:**
  - `query` (string, wajib)
  - `channel` (string, opsional): `"web"` | `"telegram"` | `"whatsapp"`
- **Response (`200 OK`):**
  - `status`: `"answered"` | `"out_of_scope"` | `"fallback"`
  - `answer` (string)
  - `sources` (array of string, opsional)

## 4. Standar Kode Status HTTP & Error
- `200 OK`: Request berhasil diproses sesuai status domain (`parsed`, `incomplete`, `out_of_scope`, `answered`).
- `400 Bad Request`: Payload tidak valid atau input kosong.
- `422 Unprocessable Entity`: Tipe/struktur data tidak sesuai skema validasi.
- `429 Too Many Requests`: Rate limit / kuota API tercapai.
- `500 Internal Server Error` / `503 Service Unavailable`: Kesalahan internal atau layanan AI sedang tidak tersedia.
- **Format Error Standar:**
  - `error_code` (string)
  - `message` (string)
  - `fallback_to_manual` (boolean)

## 5. Catatan Integrasi & Mock Response
- Selama model/endpoint AI belum selesai diimplementasi penuh, Backend dapat memakai contoh JSON mock response sesuai kontrak ini agar alur Frontend–Backend tidak terhambat.
- Setiap perubahan schema/field wajib disepakati bersama sebelum diubah pada kode.
