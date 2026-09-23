# BirdVault Pro 🦅📸

**BirdVault Pro** is a browser-based wildlife photography assistant and life-list cataloging application built with React 18, Tailwind CSS, and Google's Gemini Vision AI (`gemini-3-flash-preview`).

Designed specifically for wildlife photographers handling high-resolution DSLR and mirrorless photo collections client-side, BirdVault Pro streamlines focus culling, micro-contrast inspection, AI species identification, and sighting cataloging—all within a fast, privacy-preserving browser environment.

---

## 🌟 Key Features

### 1. High-Performance Ingestion & Dual-Proxy Memory Pipeline
* **Hardware Thumbnail Downscaling**: Utilizes native browser `createImageBitmap()` to asynchronously decode high-resolution source files into lightweight $300 \times 300\text{ px}$ JPEGs ($\approx 15\text{ KB}$ per image) on ingestion.
* **Dual-Proxy Memory Architecture**: Retains lightweight thumbnail blob URLs alongside original `File` handles in client memory. Explicit `URL.revokeObjectURL()` calls execute automatically during photo deletion or batch culling to prevent browser memory leaks during long culling sessions.
* **Paged Workspace Grid**: Displays workspace thumbnails in paginated batches of 20 images per page to maintain low DOM node counts, fluid scrolling, and optimal GPU performance.

### 2. Micro-Contrast Focus Engine & 5-Star Rating System
* **Dual Focus Detection Pipeline**:
  1. **Hardware EXIF AF Extraction**: Parses raw binary EXIF MakerNotes for camera SubjectArea AF point coordinates (`0x9214`).
  2. **Spatial Laplacian Grid Variance**: For images missing hardware AF tags, the focus engine divides the image into a $5 \times 5$ spatial grid and calculates 2D discrete Laplacian edge variance:
     $$L(x,y) = -4Y(x,y) + Y(x, y-1) + Y(x, y+1) + Y(x-1, y) + Y(x+1, y)$$
     across BT.601 luminance values ($Y = 0.299R + 0.587G + 0.114B$) to pinpoint peak micro-contrast hotspots.
* **5-Star Rating Conversion**: Focus quality scores ($0$–$10$) automatically map to a 5-star scale ($\bigstar \bigstar \bigstar \bigstar \bigstar$).
* **Manual Star Overrides & Filtering**: Adjust star ratings directly on thumbnail cards or inside the Photo Inspector sidebar. Filter the workspace by star rating (All, 5 Stars [Tack Sharp], 4+ Stars [Sharp], 1–3 Stars [Blurry/Soft], Unrated) and sort by filename, capture date, or star rating.

### 3. Precision Photo Inspector Tools
* **Lightbox Mode**: Features non-passive wheel zooming (`{ passive: false }`), touch pinch-to-zoom, double-click/double-tap $1\times \leftrightarrow 2.5\times$ zoom toggling, pan dragging, and actual-size ($1:1$) pixel inspection up to $800\%$.
* **Crop AI Tool**: Interactive region of interest (ROI) framing box to isolate diagnostic visual features (head, beak, eyes) prior to AI evaluation.
* **Screen-Calibrated & Persistent Loupe Magnifier**: Calibrates magnification relative to displayed on-screen image dimensions ($1.0\times$ baseline matching screen resolution up to $20.0\times$ detail zoom). The lens remains anchored on the photo during adjustment without vanishing.
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
* **Lifer List Vault**: Persistent master catalog recording species taxonomy, scientific names, sighting timestamps, cropped focus thumbnails, location context, and star ratings.

---

## 📊 Gemini AI Token & Cost Breakdown

### 1. Token Breakdown per Request

| Phase / Data Type | Estimated Token Count | Description |
| :--- | :--- | :--- |
| **Vision Image Tokens** | ~258 tokens | Bounded $2048 \times 2048\text{ px}$ cropped ROI JPEG payload |
| **System & Prompt Context** | ~65–90 tokens | Ornithological system rules, location context & schema |
| **Output Response** | ~100–150 tokens | JSON payload (species name, confidence, heatmap coordinates) |
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
* Mobile touch gestures support pinch-to-zoom in Lightbox view and direct touch dragging for the Loupe magnifier.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.