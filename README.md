# BirdVault Pro 🦅📸

**BirdVault Pro** is a browser-based wildlife photography assistant and life-list cataloging application built with React 18, Tailwind CSS, and Google's Gemini Vision AI (`gemini-3-flash-preview`).

Designed specifically for wildlife photographers handling high-resolution DSLR and mirrorless photo collections client-side, BirdVault Pro streamlines focus culling, micro-contrast inspection, AI species identification, and sighting cataloging—all within a fast, privacy-preserving browser environment.

---

## 🌟 Key Features

### 1. High-Performance Ingestion & Dual-Proxy Memory Pipeline
* **Hardware Thumbnail Downscaling**: Uses native browser `createImageBitmap()` to asynchronously create lightweight $240 \times 240\text{ px}$ JPEG previews. Culling cards with a saved crop get a separate crop-aware preview (up to $600\text{ px}$ on its longest edge), cached in IndexedDB to retain detail when zooming into tight crops.
* **Dual-Proxy Memory Architecture**: Retains lightweight thumbnail blob URLs alongside original `File` handles in client memory. Explicit `URL.revokeObjectURL()` calls execute automatically during photo deletion or batch culling to prevent browser memory leaks during long culling sessions.
* **Paged Workspace Grid**: Displays workspace thumbnails in paginated batches of 20 images per page to maintain low DOM node counts, fluid scrolling, and optimal GPU performance.
* **Folder-Order Ingestion**: Keeps the order returned by the browser's folder or file picker instead of sorting imports by modification date. Folder-handle imports begin displaying each 20-file batch as it is read; thumbnail generation remains limited to the visible page. Browsers do not guarantee that picker order matches the order shown by the operating system's file browser.

### 2. Micro-Contrast Focus Engine & 5-Star Rating System
* **Free On-Device Focus Ratings**: Generic batch culling compares normalized edge acutance and local edge continuity across overlapping image regions. It is less dependent on a centered subject and filters isolated texture/noise; analysis stays in the browser with no Gemini/API requests or quota usage.
* **AI Focus Rating During Identification**: When the user explicitly identifies a bird, the same Gemini response includes a 0–10 bird-focus score and assessment alongside the species result and heatmap. No extra AI request is made for the rating.
* **5-Star Rating Conversion**: Local or AI focus quality scores ($0$–$10$) automatically map to a 1–5 star scale ($\bigstar \bigstar \bigstar \bigstar \bigstar$).
* **Manual Star Overrides & Filtering**: Adjust star ratings directly on thumbnail cards or inside the Photo Inspector sidebar. Filter the workspace by star rating (All, 5 Stars [Tack Sharp], 4+ Stars [Sharp], 1–3 Stars [Blurry/Soft], Unrated) and sort by filename, capture date, or star rating.
* **Photo Sequence Video**: Select multiple culling photos to create a crossfade slideshow or a hard-cut animation. Slideshow durations range from 1 to 5 seconds; animation durations range from 0.1 to 2 seconds in 0.1-second steps, with no transitions between photos. Both modes include landscape/portrait/square formats, resolution, captions, and image-fit controls. Video encoding is componentized for reuse; MP4 is used when supported, otherwise the browser's supported WebM format is offered.

### 3. Precision Photo Inspector Tools
* **Lightbox Mode**: Features non-passive wheel zooming (`{ passive: false }`), touch pinch-to-zoom, double-click/double-tap $1\times \leftrightarrow 2.5\times$ zoom toggling, pan dragging, and actual-size ($1:1$) pixel inspection up to $800\%$.
* **Crop AI Tool**: Interactive region of interest (ROI) framing box to isolate diagnostic visual features (head, beak, eyes) prior to AI evaluation.
* **Screen-Calibrated & Persistent Loupe Magnifier**: Calibrates magnification relative to displayed on-screen image dimensions ($1.0\times$ baseline matching screen resolution up to $20.0\times$ detail zoom). The lens remains anchored on the photo during adjustment without vanishing.
* **AI Photo Processing Controls**: On mobile, open processing tools from the preview toolbar; adjustments appear in a compact panel below the image so the live preview remains visible while tuning sliders.
* **Image Version History**: Switch the active image version from the preview controls and delete processed versions; the original capture is protected from deletion.
* **Conditional Focus Target Overlay**: Displays a sky-blue target marker over camera AF spot coordinates or computed micro-contrast hotspots, automatically hiding during active zoom or crop preview.

### 4. Selection, Deletion & Local Export
* **Individual Selection Checkboxes**: Active selection checkboxes on each thumbnail card for custom batching.
* **Dynamic Batch Deletion**: Toolbar delete button adapts dynamically:
  * When photos are checked: **"Delete Selected (N)"**.
  * When no photos are checked: **"Delete Filtered (N)"**.
* **In-App Safety Modal**: In-app confirmation dialog replacing native browser `confirm()` prompts to guarantee safety inside sandboxed iFrames.
* **Quick Trash & Inspector Delete**: One-tap inline card deletion, plus top-bar inspector deletion that automatically advances to the next image in sequence.
* **Local Folder Export**: Uses the File System Access API (`showDirectoryPicker`) to export selected or filtered photos into user-selected device folders with automatic removal from the main workspace list.

### 5. Multimodal Gemini Vision AI Identification & Heatmap
* **Multimodal Identification**: Transmits cropped ROI JPEG payloads to Google's `gemini-3-flash-preview` model along with localized spatial prompts.
* **Geographic Grounding Context**: Incorporates device GPS or user-defined regional location text into the system prompt to restrict candidate species strictly to local avifauna.
* **Diagnostic Heatmap Grounding**: Returns diagnostic visual feature points (eye contrast, beak structure, wing plumage) rendered as pulsing target overlays.
* **Lifer List Vault**: Persistent master catalog recording species taxonomy, scientific names, sighting timestamps, cropped focus thumbnails, location context, and star ratings. Export complete vault backups, including all referenced image versions, and merge a backup into the current vault from the header actions menu. API key and quota settings are also in that menu.
* **Focus Rating in the Same Request**: The identification response also rates visible bird sharpness; selecting generic batch rating never calls Gemini.

---

## 📊 Gemini AI Token & Cost Breakdown

### 1. Token Breakdown per Request

| Phase / Data Type | Estimated Token Count | Description |
| :--- | :--- | :--- |
| **Vision Image Tokens** | ~258 tokens | Bounded $2048 \times 2048\text{ px}$ cropped ROI JPEG payload |
| **System & Prompt Context** | ~65–90 tokens | Ornithological system rules, location context & schema |
| **Output Response** | ~120–180 tokens | JSON payload (species name, confidence, focus rating, heatmap coordinates) |
| **TOTAL PER REQUEST** | **~350–500 tokens** | Complete round-trip token expenditure |

### 2. Pricing & Cost Model ($ USD)

* **Google AI Studio Free Allowance**:
  * **15 Requests Per Minute (RPM)**
  * **1,500 Requests Per Day (RPD)**
  * **Cost**: **$0.00 (100% Free)**
* **Paid Billing Plan Rates**:
  * Input Tokens: $\approx \$0.50$–$\$0.75$ per 1,000,000 tokens
  * Output Tokens: $\approx \$3.00$–$\$3.75$ per 1,000,000 tokens
* **Effective Unit Cost**:
  * **1 Identification Call**: $\approx \mathbf{\$0.0000007\text{ USD}}$ (less than $\frac{1}{10,000}\text{th}$ of a US cent)
  * **1,000 Identifications**: $\approx \mathbf{\$0.0007\text{ USD}}$
  * **10,000 Identifications**: $\approx \mathbf{\$0.0007\text{ USD}}$ (under 1 cent)

---

## 🛠️ Architecture & Tech Stack

* **Frontend Engine**: React 18 (Standalone / UMD build)
* **Styling & UI**: Tailwind CSS
* **Image Processing**: HTML5 Canvas API, WebKit `createImageBitmap`, EXIF DataView binary parsing
* **File Operations**: File System Access API (`showDirectoryPicker`), Object URL memory lifecycle control
* **AI Model**: Google Gemini API (`gemini-3-flash-preview`)
* **State & Storage**: Browser `localStorage` for API keys, usage metrics, user preferences, and Lifer Vault persistence

---

## 🚀 Getting Started

### Prerequisites
1. A modern web browser supporting HTML5 Canvas and File System Access API (Google Chrome, Microsoft Edge, Opera, or Brave recommended).
2. A **Google Gemini API Key** (obtainable for free at [Google AI Studio](https://aistudio.google.com/app/apikey)).

### Quick Start
1. Clone or download this repository:
   ```bash
   git clone https://github.com/your-username/birdvault-pro.git
   cd birdvault-pro
   ```
2. Open `index.html` directly in any web browser, or serve via a local static HTTP server:
   ```bash
   npx serve .
   ```
3. When prompted in the setup dialog, enter your **Gemini API Key**. Your key is stored locally and securely in your browser's local storage.

---

## 📱 Mobile & PWA Usage

BirdVault Pro is PWA-ready and optimized for mobile browsers (iOS Safari & Android Chrome):
* Add the app to your phone's home screen via **Add to Home Screen** to launch in full-screen standalone mode without browser URL bars.
* Mobile touch gestures support pinch-to-zoom in Lightbox view and direct touch dragging for the Loupe magnifier. Browser page zoom by pinching is disabled outside the image preview.

---

## 📚 Project Documentation

This repository includes the main project references and supporting design documents:

- [README.md](./README.md) — overview, features, quick start, and project summary
- [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) — system design, main components, and storage model
- [docs/DEVELOPER_GUIDE.md](./docs/DEVELOPER_GUIDE.md) — setup, browser requirements, and developer notes
- [birdvault_detailed_documentation (1).md](./birdvault_detailed_documentation%20(1).md) — deeper technical and design documentation

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.