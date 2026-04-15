# GrabFood Menu Scraper — Ayam Katsu Katsunami

Screening task submission for the Freelance Automation & Web Scraper Developer position.

**Target:** [Ayam Katsu Katsunami - Lokarasa Citraland](https://food.grab.com/id/id/restaurant/ayam-katsu-katsunami-lokarasa-citraland-delivery/6-C7EYGBJDME3JRN)

---

## Approach

GrabFood is built as a React SPA (Single Page Application). The content is rendered dynamically via JavaScript, meaning standard `requests` + `BeautifulSoup` approaches will fail (the DOM is empty prior to JS execution).

**Solution: Playwright + Network Interception**

1. Launch a Playwright instance (headless Chromium).
2. Attach a `page.on("response", ...)` listener to intercept all network traffic.
3. Capture the XHR response hitting Grab's internal API endpoint: `GET .../foodweb/guest/v2/merchants/...`
4. Parse the extracted JSON response to isolate the structured menu data.
5. Export the dataset directly into an `.xlsx` file using `openpyxl`.

```text
Browser (Playwright) → Load Page → JS Fires → Grab API Called → Response Captured → Parse JSON → Excel
```

---

## Extracted Data Points

| Field                     | Internal API Source (`json`) |
| ------------------------- | --------------------------- |
| Outlet Name               | `merchant.name` |
| Category Name             | `merchant.menu.categories[].name` |
| Item Name                 | `categories[].items[].name` |
| Item Description          | `categories[].items[].description` |
| Original Price            | `items[].priceInMinorUnit` (divided by 100) |
| Promo Price               | `items[].discountedPriceInMin` (divided by 100) |
| Promo Details/Percentage  | `items[].campaignName` / calculated differential |
| Availability Status       | `items[].available` (boolean) |

---

## Setup & Execution

```bash
# 1. Install dependencies
pip install -r requirements.txt
playwright install chromium

# 2. Run the scraper
python scraper.py

# Output: katsunami_menu_data.xlsx
```

---

## Challenges & Implemented Solutions

| Challenge | Applied Solution |
|---|---|
| SPA / Dynamic Rendering | Deployed Playwright headless browser to natively execute JS engines. |
| Anti-bot Mechanisms (Cloudflare/WAF) | Used `playwright-stealth`, randomized viewport delays, and realistic user-agents. |
| Pricing Minor Units Formatting | Grab stores pricing in minor units (e.g., 54000 as 5400000). Normalized via division parsing layer. |
| Dynamic Promo Structures (Flat vs %) | Fallback checks against `campaignName` vs `discountPercentage` mapped to a unified string. |

---

## Bonus: How to extract data exclusively available on Mobile Apps (e.g. ShopeeFood)?

The optimal architecture is **TLS/SSL Interception (MITM)**.

```text
Phone (ShopeeFood app) → Local Network → Laptop (mitmproxy) → Internet
```

1. Install **mitmproxy** on the host machine.
2. Configure the mobile device's proxy to tunnel traffic through the host machine (Port 8080).
3. Install the mitmproxy CA Certificate onto the mobile device's trusted root certificates.
4. Open the target app (ShopeeFood) — all internal API requests are now unencrypted and captured.
5. Identify the exact API endpoint rendering the menu and replicate the HTTP request mapping in Python.

**Advanced Fallback / Obstacle Handling:**
If the application deploys **SSL Pinning**, spin up a rooted Android emulator combined with **Frida** to hook into and bypass the ssl-pinning verification logic at runtime, or directly decompile the base APK using `jadx` to locate the endpoint structures.
