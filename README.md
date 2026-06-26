# Merchandise Order Form

A static order form hosted on GitHub Pages that posts orders to a Make.com webhook, which writes the data to Google Sheets.

## Repository structure

```
/
├── index.html        ← The order form (no changes needed for normal use)
├── products.json     ← Edit this to add/change products
├── images/           ← Add product images here
│   ├── hoody.jpg
│   ├── sweater.jpg
│   └── ...
└── README.md
```

---

## Setup

### 1. Enable GitHub Pages

1. Push this repo to GitHub.
2. Go to **Settings → Pages**.
3. Set **Source** to `Deploy from a branch`, select `main`, folder `/` (root).
4. Your site will be live at `https://<your-username>.github.io/<repo-name>/`.

### 2. Set up Make.com webhook

1. In Make.com, create a new Scenario.
2. Add a **Webhooks → Custom webhook** trigger. Copy the URL it gives you.
3. Add a **Google Sheets → Add a Row** module, map the incoming fields (see payload structure below).
4. Activate the scenario.

### 3. Enter the webhook URL on the form

When you open the order form, paste the Make.com webhook URL into the **Webhook URL** field at the bottom. It's saved automatically in your browser's localStorage so you only need to enter it once per device.

> **Tip:** If you want the URL pre-filled for all users, hard-code it in `index.html`:
> Find `const PRODUCTS_URL = './products.json';` and add below it:
> ```js
> document.getElementById('webhook-url').value = 'https://hook.eu1.make.com/YOUR_KEY';
> ```

---

## Adding / editing products (`products.json`)

Each product is an object in the JSON array. Two types are supported:

### Sized product (e.g. clothing)
```json
{
  "id": "tshirt",
  "name": "T-Shirt",
  "type": "sized",
  "sizes": ["XS", "S", "M", "L", "XL", "2XL"],
  "image": "images/tshirt.jpg",
  "description": "Short-sleeve branded tee"
}
```

### Quantity-only product
```json
{
  "id": "mug",
  "name": "Mug",
  "type": "quantity",
  "image": "images/mug.jpg",
  "description": "Ceramic branded mug"
}
```

- `id` — unique snake_case identifier (used in the webhook payload)
- `name` — displayed on the card
- `type` — `"sized"` or `"quantity"`
- `sizes` — array of size strings (only for `"sized"` type)
- `image` — relative path to image in the `images/` folder
- `description` — short subtitle on the card

---

## Webhook payload structure

Make.com receives a JSON body like this:

```json
{
  "submitted_at": "2025-06-26T09:15:00.000Z",
  "contact": {
    "first_name": "Jane",
    "last_name": "Smith",
    "email": "jane@example.com",
    "mobile": "+27 82 000 0000"
  },
  "order_lines": [
    { "product": "Hoody", "size": "L",   "qty": 2, "label": "Hoody (L)" },
    { "product": "Hoody", "size": "XL",  "qty": 1, "label": "Hoody (XL)" },
    { "product": "Umbrella", "size": null, "qty": 3, "label": "Umbrella" }
  ],
  "order_summary": "Hoody (L) x2; Hoody (XL) x1; Umbrella x3"
}
```

In Google Sheets you can map individual fields or use `order_summary` for a single-column order description.

---

## Adding product images

1. Create an `images/` folder in the repo root.
2. Drop in your image files (JPG or PNG recommended, ~800×600px or any 4:3 ratio).
3. Make sure the filename matches the `"image"` path in `products.json`.

If an image file is missing, the card shows a neutral placeholder — no errors.

---

## Why `mode: 'no-cors'`?

Make.com webhook endpoints don't return CORS headers, so the browser would block the response if we tried to read it. Using `no-cors` lets the request go through and Make.com receives the data, but the browser can't confirm success. In practice this is fine — if the URL is correct, the data arrives. If you see it failing, check the Make.com scenario history to confirm receipt.
