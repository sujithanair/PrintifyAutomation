

````markdown
# AI Printify → Etsy Launcher 🧠👕  
Automate turning a folder of designs into Etsy-ready product drafts via Printify.

This project scans a local folder of t-shirt designs, generates titles/descriptions/tags
from the filenames, uploads the designs to Printify, creates products (e.g. Gildan 5000),
and publishes them as **draft listings** to your Etsy-connected Printify shop.

> Goal: Turn “a bunch of PNGs on your desktop” into Etsy drafts with minimal manual work.

---

## ✨ Features

- 📂 **Folder-based workflow**  
  Drop PNG/JPG files into a folder, run the script, done.

- 🧠 **AI-style metadata generation**  
  - Title generated from filename (e.g. `too_cute_to_spook.png` → “Too Cute To Spook Tee”)  
  - Short SEO-friendly description  
  - Up to 13 Etsy tags from keywords in the filename

- 🖨️ **Printify automation**
  - Uploads artwork via the Printify API
  - Creates products for a chosen blueprint (e.g. Gildan 5000)
  - Assigns the design to the front print area across selected variants

- 🛍️ **Etsy draft publishing**
  - Uses Printify’s publish endpoint to send products as **drafts** to Etsy  
  - You can review/edit in Etsy before going live

---

## 🧱 Tech Stack

- Python 3.10+  
- `requests` for HTTP calls  
- Printify REST API  
- (Optional, future) OpenAI API for richer AI text generation

---

## 📁 Project Structure

Example structure:

```text
ai-printify-etsy-launcher/
├─ src/
│  ├─ main.py                # Entry point – orchestrates the flow
│  ├─ ai_text_generator.py   # Title/description/tag generation
│  ├─ printify_client.py     # Helpers to call Printify API
│  ├─ config_example.py      # Template config (no secrets)
│  └─ config.py              # Your real config (ignored by git)
├─ .gitignore
├─ README.md
├─ requirements.txt
└─ LICENSE
````

You can start with just `src/main.py` and grow into this structure.

---

## 🚀 Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/<your-username>/ai-printify-etsy-launcher.git
cd ai-printify-etsy-launcher
```

### 2. Create & activate a virtual environment (optional but recommended)

```bash
python3 -m venv .venv
source .venv/bin/activate      # macOS / Linux
# .venv\Scripts\activate.bat   # Windows (cmd)
# .venv\Scripts\Activate.ps1   # Windows (PowerShell)
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

Example `requirements.txt`:

```text
requests>=2.31.0
python-dotenv>=1.0.0
```

---

## 🔐 Configuration

You need:

* A **Printify account** with an Etsy store connected
* A **Printify API key** (`Settings → API`)
* A local folder with your designs (PNG/JPG)

### Option A – Environment variables (recommended)

Set your API key in your shell:

```bash
export PRINTIFY_API_KEY="YOUR_REAL_API_KEY_HERE"
```

Then in Python, you can read it:

```python
import os
PRINTIFY_API_KEY = os.getenv("PRINTIFY_API_KEY")
```

### Option B – Local `config.py` (don’t commit this)

`src/config_example.py`:

```python
# src/config_example.py

PRINTIFY_API_KEY = "YOUR_PRINTIFY_API_KEY_HERE"
DESIGNS_FOLDER = "/absolute/path/to/your/designs/folder"

# These IDs must be fetched from the Printify API for your account
BLUEPRINT_ID = 0        # e.g. Gildan 5000 blueprint id
PRINT_PROVIDER_ID = 0   # e.g. Printify Choice provider id

# List of variant IDs (sizes / colors) for this provider + blueprint
VARIANT_IDS = []  # e.g. [4012, 4013, 4014, 4015]

PUBLISH_DRAFT = True
BASE_PRICE = 2500   # Example: price in cents or in Printify’s expected format
```

Then copy it and fill in real values:

```bash
cp src/config_example.py src/config.py
```

Add `src/config.py` to `.gitignore` so you don’t leak keys:

```text
# .gitignore
__pycache__/
.venv/
src/config.py
.DS_Store
```

---

## 📦 Getting Blueprint, Provider & Variant IDs

To create products correctly, you need:

* `BLUEPRINT_ID` – which product (e.g. Gildan 5000)
* `PRINT_PROVIDER_ID` – which printer (e.g. Printify Choice)
* `VARIANT_IDS` – which sizes/colors to enable

You can discover these via the Printify API using your key.

### List all blueprints

```bash
curl -X GET "https://api.printify.com/v1/blueprints.json" \
  -H "Authorization: Bearer YOUR_PRINTIFY_API_KEY"
```

Find the product you want (e.g. Gildan 5000) and note its `id`.

### List providers for a blueprint

```bash
curl -X GET "https://api.printify.com/v1/blueprints/<BLUEPRINT_ID>/print_providers.json" \
  -H "Authorization: Bearer YOUR_PRINTIFY_API_KEY"
```

Find the provider you want (e.g. Printify Choice), note its:

* `id` → `PRINT_PROVIDER_ID`
* `variants` → each variant has an `id` (put these into `VARIANT_IDS`)

You can also build small helper functions in Python (e.g. `debug_list_blueprints()` / `debug_list_providers_for_blueprint()`) to print this to the console.

---

## ▶️ Running the Script

Once `config.py` (or env vars) are set:

```bash
python3 -m src.main
```

What happens:

1. Fetches your first Printify shop
2. Scans `DESIGNS_FOLDER` for `.png`, `.jpg`, `.jpeg`
3. For each file:

   * Generates a title, description, and tags
   * Uploads the image to Printify
   * Creates a product with the configured blueprint/provider/variants
   * Publishes the product as a **draft** to Etsy (if enabled)

Always start with **one test file** and confirm everything looks good in your Etsy drafts before running large batches.

---

## 🧠 AI Text Generation

Right now, the project uses **filename-based generation** in `ai_text_generator.py`:

* Cleans the filename (`too_cute_to_spook_cat.png` → `too cute to spook cat`)
* Builds:

  * Title: `"Too Cute To Spook Cat Tee"`
  * Description: generic but SEO-friendly
  * Tags: `["too", "cute", "to", "spook", "cat", ...]` (deduped, max 13)

Planned AI upgrades:

* Use the OpenAI API to:

  * Write more engaging product descriptions
  * Suggest niche-focused tags
  * Vary tone (e.g. “cute”, “spooky”, “minimalist”)

---

## ⚠️ Safety & Gotchas

* **API keys**:
  Never commit your real `PRINTIFY_API_KEY` or `config.py`. This repo is meant to be safe to share publicly.

* **Pricing format**:
  Printify’s API may expect prices in a specific format (e.g. cents). Verify and adjust `BASE_PRICE` & product creation payload accordingly.

* **Rate limiting**:
  There’s a small `sleep()` between API calls. If you scale up, you may need to tune this.

* **Check drafts before publishing live**:
  This script publishes to **Etsy drafts**, so you can sanity-check everything before making listings active.

---

## 🗺️ Roadmap

* [ ] Use OpenAI API for richer copy and keyword extraction
* [ ] Better error handling & logging to a file
* [ ] Support multiple blueprints (hoodies, mugs, etc.)
* [ ] Flags/CLI args for “dry run”, “one file only”, etc.
* [ ] Dockerfile for easy deployment

---

