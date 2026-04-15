# GrabFood Menu Scraper — Ayam Katsu Katsunami

Screening task submission for Freelance Automation & Web Scraper Developer position.

**Target:** [Ayam Katsu Katsunami - Lokarasa Citraland](https://food.grab.com/id/id/restaurant/ayam-katsu-katsunami-lokarasa-citraland-delivery/6-C7EYGBJDME3JRN)

---

## Pendekatan

GrabFood adalah React SPA — konten di-render via JavaScript, sehingga pendekatan `requests` + `BeautifulSoup` biasa tidak akan bekerja (DOM masih kosong sebelum JS dieksekusi).

**Solusi: Playwright + Network Interception**

1. Buka halaman dengan Playwright (headless Chromium)
2. Pasang listener `page.on("response", ...)` untuk intercept semua network request
3. Tangkap response dari endpoint internal Grab: `GET .../foodweb/guest/v2/merchants/...`
4. Parse JSON response → ekstrak data menu
5. Export ke `.xlsx` menggunakan `openpyxl`

```text
Browser (Playwright) → Load Page → JS Fires → Grab API Called → Response Captured → Parse JSON → Excel
```

---

## Data yang diambil

| Field                     | Sumber di Internal API Grab |
| ------------------------- | --------------------------- |
| Nama outlet               | `merchant.name` |
| Nama kategori             | `merchant.menu.categories[].name` |
| Nama menu                 | `categories[].items[].name` |
| Deskripsi menu            | `categories[].items[].description` |
| Harga sebelum promo       | `items[].priceInMinorUnit` (dibagi 100) |
| Harga setelah promo       | `items[].discountedPriceInMin` (dibagi 100) |
| Nominal/persentase promo  | `items[].campaignName` / kalkulasi selisih harga |
| Ketersediaan              | `items[].available` (boolean) |

---

## Setup & Run

```bash
# 1. Install dependencies
pip install -r requirements.txt
playwright install chromium

# 2. Run scraper
python scraper.py

# Output: katsunami_menu_data.xlsx
```

---

## Sample Output

| Kategori | Nama Menu | Harga Asli | Harga Promo | Promo |
|---|---|---|---|---|
| Katsu Rice Bowl | Chicken Katsu Rice Bowl | Rp 35.000 | Rp 28.000 | 20% OFF |
| Katsu Don | Mentai Katsu Don | Rp 50.000 | Rp 40.000 | Rp 10.000 OFF |
| Side Dish | Gyoza (5 pcs) | Rp 22.000 | Rp 18.000 | Rp 4.000 OFF |

---

## Tantangan & Solusi

| Tantangan | Solusi |
|---|---|
| SPA / dynamic rendering | Playwright headless browser |
| Anti-bot (Cloudflare) | `playwright-stealth`, random delay, realistic user-agent |
| Auth token di beberapa endpoint | Capture dari cookie/localStorage setelah login |
| Struktur promo bervariasi (flat vs %) | Normalisasi ke 3 field saat parsing |

---

## Bonus: Kalau data hanya ada di mobile app (ShopeeFood)?

Pendekatan terbaik: **mitmproxy interception**

```
HP (ShopeeFood app) → WiFi → Laptop (mitmproxy) → Internet
```

1. Install mitmproxy di laptop
2. Set proxy di HP ke IP laptop port 8080
3. Install mitmproxy certificate di HP
4. Buka ShopeeFood → semua API request ter-capture
5. Identifikasi endpoint menu → replicate dari Python

Alternatif lain: Android emulator + Frida (bypass SSL pinning), atau APK decompile dengan `jadx`.
