# Unstructured API - Home Assistant Add-on

This repository contains a custom Home Assistant Add-on that runs the **Unstructured.io API** locally. It allows you to ingest and parse complex documents (PDFs, HTML, images) directly within your local network.

## 🛠 The Development Journey

This add-on is the result of an iterative process to find the most stable and efficient way to run this heavy-duty API within the Home Assistant ecosystem. Here is how we arrived at this solution:

### Phase 1: The Build Attempt 🏗️
Initially, we considered building the image from scratch using a **Rocky Linux** base (replicating the source Dockerfile).
* **The Problem:** The Unstructured API requires compiling `tesseract`, `libreoffice`, and massive Python libraries (`torch`, `pandas`, `nltk`).
* **The Result:** This was deemed unsuitable for Home Assistant. The build process would likely timeout on the HA Supervisor, and maintaining the dependencies manually would be a nightmare.

### Phase 2: Switching to Official Images 🐳
We decided to leverage the pre-built, official Docker image provided by Unstructured.io (`downloads.unstructured.io`).
* **The Benefit:** This guarantees compatibility and eliminates the need for local compilation.
* **The Challenge:** The image is huge (~5GB compressed, ~10GB uncompressed).

### Phase 3: Hardware Validation 🖥️
We confirmed that this add-on is **not suitable for Raspberry Pis**.
* **Target Hardware:** This configuration is optimized for high-performance setups (e.g., **Intel NUC N305 with 32GB RAM** and SSD).
* **Resource Usage:** The API is memory-intensive and requires fast I/O.

### Phase 4: Solving Persistence (The "Aha!" Moment) 💾
The biggest hurdle was that Docker containers reset upon restart. By default, the Unstructured API downloads models (Hugging Face Transformers, NLTK data) to the container's temporary storage.
* **The Issue:** Every time the add-on restarted, it had to re-download gigabytes of models, wasting bandwidth and time.
* **The Solution:** We modified the `Dockerfile` to set environment variables (`HF_HOME`, `NLTK_DATA`) pointing to `/data`. In Home Assistant, the `/data` directory is persistent. Models are now downloaded once and saved forever.

### Phase 5: Process Management ⚙️
* **The Tweak:** We set `init: false` in `config.yaml`. This disables Home Assistant's default S6 overlay init system, allowing the official container's native entrypoint to manage the application process without conflicts.

---

## ⚠️ Prerequisites

**Do not attempt to run this on a Raspberry Pi.**

* **Architecture:** x86_64 (amd64) / aarch64 (Apple Silicon/High-end ARM)
* **RAM:** Minimum 8GB (16GB+ recommended)
* **Storage:** SSD is required due to the large image size.

## 📥 Installation

1.  Add this repository to your Home Assistant Add-on Store:
    * Settings > Add-ons > Add-on Store > **Repositories** (3 dots top right).
    * Paste the URL of this GitHub repo.
2.  Install the **Unstructured API** add-on.
    * *Note: The download may take 5-15 minutes depending on your internet speed. Be patient.*
3.  Start the add-on.

## ⚙️ Configuration

No complex configuration is needed. The `Dockerfile` automatically handles the environment variables for persistence.

**Note on First Run:**
When you send your first request (e.g., parsing a PDF), the system will download the necessary models to `/data`. **This will take time.** Please monitor the logs and wait for the download to complete before expecting a response.

## 🚀 Usage

Once running, the API listens on port `8000`. You can interact with it via `cURL` or Node-RED.

### Example: Parsing a PDF via cURL

Replace `homeassistant.local` with your IP address.

```bash
curl -X 'POST' \
  '[http://homeassistant.local:8000/general/v0/general](http://homeassistant.local:8000/general/v0/general)' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'files=@/path/to/your/document.pdf'
