# Grand Total By SPG

Webapp satu file (tanpa instalasi, tanpa build) untuk mencari **Nama SPG**, filter **Region** dan **Pencapaian**, mengurutkan berdasarkan **persentase pencapaian**, dan menampilkan **Grand Total**, **Target SPG**, serta **Gap**. Menggunakan color tone merah/oranye Lezza. Sumber data: sheet **"Performa SPG"** dari Google Sheet:

https://docs.google.com/spreadsheets/d/18fTd6PLLoRcOcRHrJGUKDBEap8xdbebqZFL95GoNO1I/edit?usp=sharing

## Cara pakai

1. Buka `index.html` langsung di browser (double-click), atau:
   ```bash
   cd spg-grand-total
   python3 -m http.server 8000
   ```
   lalu buka `http://localhost:8000`.
2. Saat dibuka, hanya kolom pencarian (ukuran besar) yang tampil. Ketik nama SPG (mis. `Anita`, `Dede`, `Meitina`) untuk mencari.
3. Satu-satunya tombol "tampilkan semua" ada di halaman awal: **TAMPILKAN SEMUA SPG**. Memilih Region/Pencapaian atau mengubah Urutkan juga langsung menampilkan data. Mengosongkan kolom pencarian kembali ke tampilan awal.
4. **Region**: pilih Jabodetabek, Bali, Jawa Barat, atau "Tanpa Region" (baris yang belum diberi region di sheet). Opsi **Semua Region** menampilkan semuanya.
5. **Pencapaian**: filter SPG berdasarkan status pencapaiannya (‼️ Kritis sampai 🏆 Kerja bagus, luar biasa!, plus ⚪ Belum ada target). Opsi **Semua Pencapaian** menampilkan semuanya. Filter ini bisa digabung dengan Region dan pencarian nama.
6. **Urutkan**: `Pencapaian tertinggi` (default), `Pencapaian terendah`, `Grand Total tertinggi`, atau `Nama A–Z`. Persentase dihitung `Grand Total / Target SPG × 100`; SPG tanpa target diletakkan di urutan terakhir untuk sortir pencapaian.
7. Tiap kartu menampilkan region, **Grand Total**, **Target SPG**, **Gap**, persentase pencapaian, progress bar, dan **status pencapaian** otomatis sesuai pencapaian.
8. Tekan tombol **LIHAT TOKO** pada kartu SPG untuk rincian pencapaian tiap toko. Tombol berubah jadi **SEMBUNYIKAN TOKO** saat terbuka.
9. Tombol **↻ Refresh** untuk mengambil data terbaru secara manual.

## Cara kerja data

- Saat halaman dibuka, webapp mengambil sheet **"Performa SPG"** melalui endpoint `gviz/tq` Google Sheets (JSONP), lalu mengolahnya di browser. Tidak perlu export CSV manual.
- Pengambilan ulang otomatis **setiap 1 menit**, juga saat tab dibuka kembali setelah minimal 1 menit, dan lewat tombol Refresh. Jadi perubahan di sheet otomatis tampil dengan jeda maksimal ±1 menit tanpa mengubah kode.
- Catatan: karena mengecek Google Sheets setiap menit, hindari membuka banyak tab sekaligus agar tidak kena pembatasan request Google. Interval bisa diubah lewat variabel `REFRESH_MS` di dalam `index.html` (satuan milidetik).
- Kolom yang dipakai:
  - `A` → **Region** (dipakai untuk filter; diambil dari baris pertama tiap SPG. Baris dengan region kosong ditampilkan sebagai "Tanpa Region")
  - `B` → **Nama SPG** (dipakai untuk pencarian & pengelompokan)
  - `C`, `D` → Cust Code & Toko (rincian)
  - `H` → total per toko
  - `I` → **Grand Total per SPG** (nilai di baris pertama tiap SPG). Jika kosong, total dihitung dari penjumlahan kolom `H`.
  - `J` → **Target SPG** (nilai di baris pertama tiap SPG).
  - `K` → **Gap** (nilai di baris pertama tiap SPG). Rumus di sheet adalah `Gap = Target − Grand Total`. Angka ditampilkan **tanpa tanda +/−** (nilai mutlak); statusnya cukup dilihat dari warna: **hijau = sudah melebihi target**, **merah = belum mencapai target**.
- Gap **tidak dihitung ulang** oleh webapp, mengikuti angka di sheet. Jadi jika ada baris yang arah warnanya tidak konsisten dengan `Target − Grand Total`, itu berasal dari sheet sumber (mis. saat ini `Atik Maryati`). Cek kembali rumus di sheet bila perlu.
- Label tanggal diambil dari baris pertama sheet (saat ini: `21 Nov`).
- Status di kartu dihitung dari persentase pencapaian yang sudah dibulatkan:

  | Pencapaian | Status di kartu |
  |---|---|
  | 0–20% | ‼️ Kritis |
  | 21–40% | 🟠 Waspada |
  | 41–60% | 🟡 Kejar lagi |
  | 61–80% | 🔵 Cukup Baik |
  | 81–90% | 🟢 Hampir Tercapai |
  | 91–100% | ✅ Sedikit lagi |
  | > 100% | 🏆 Kerja bagus, luar biasa! |
  | Tanpa target | ⚪ Belum ada target |

  Daftar yang sama dipakai sebagai pilihan dropdown **Pencapaian**, jadi filter selalu sinkron dengan label di kartu.

- Warna mengikuti tone logo Lezza: `#8E1D04`, `#C02105`, `#E2432A`, `#F08B71`, `#F7B7A4`, dengan teks `#4A1508`. Hijau `#2E7D4F` hanya dipakai sebagai penanda "sudah achieve", merah `#C02105` untuk "belum achieve".
- Jika Google Sheet belum bisa dijangkau (offline / izin berubah), app memakai **snapshot cadangan** yang ada di dalam file dan status berubah menjadi "Mode cadangan". Data langsung tetap diprioritaskan begitu tersedia.

## Deploy ke GitHub Pages (subdomain gratis `*.github.io`)

Subdomain gratis dari GitHub sudah otomatis, **tidak perlu membeli domain** dan **tidak perlu mengisi kolom "Custom domain"** di Settings → Pages (kolom itu hanya untuk domain milik sendiri).

1. Buat repository **public** di GitHub.
2. Upload file `index.html` ke root repository (GitHub → **Add file → Upload files**). File `README.md` boleh ikut.
3. Buka **Settings → Pages**.
4. **Build and deployment**
   - **Source:** `Deploy from a branch`
   - **Branch:** `main` (atau `master`) · **(root)**
   - klik **Save**.
5. Tunggu ±1–2 menit, lalu buka salah satu (sesuai nama repo):
   - Repo bernama `muhammadrivad-bot.github.io` → `https://muhammadrivad-bot.github.io/`
   - Repo bernama mis. `grand-total-spg` → `https://muhammadrivad-bot.github.io/grand-total-spg/`
6. Cek di browser (boleh incognito): status harus **"Data langsung dari Google Sheet"**.

> GitHub hanya menyediakan `username.github.io` dan `username.github.io/nama-repo`. Bentuk seperti `spg.muhammadrivad-bot.github.io` **tidak** tersedia gratis; itu baru bisa kalau pakai domain sendiri + CNAME.

## Kalau nanti mau pakai domain sendiri

1. Beli domain (mis. di Cloudflare Registrar / Namecheap).
2. Di GitHub: **Settings → Pages → Custom domain**, isi mis. `spg.namadomain.com`, Save.
3. Di DNS domain, buat record:
   - `CNAME` → `muhammadrivad-bot.github.io` (untuk `www`/subdomain seperti `spg.`)
   - atau `A` → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` (untuk domain apex `namadomain.com`)
4. Centang **Enforce HTTPS** setelah DNS aktif.

## Catatan

- Sheet harus tetap bisa diakses oleh siapa pun yang memiliki link (`Anyone with the link`) agar endpoint `gviz` bisa dibaca tanpa login.
- File `index.html` bisa di-host di static hosting mana pun (GitHub Pages, Netlify, dll.) tanpa backend.
- Webapp memakai JSONP ke `docs.google.com`, jadi tetap berfungsi walau di-host di submodule path `username.github.io/nama-repo/`.
- Tampilan responsif untuk HP: **Grand Total** jadi satu baris penuh, **Target SPG** dan **Gap** dua kolom di bawahnya, dan tabel rincian menyembunyikan kolom Cust Code agar tidak mepet. Sudah dicek sampai lebar 320 px tanpa konten terpotong.
