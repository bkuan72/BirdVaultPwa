# BirdVault Pro: Comprehensive Technical Architecture & Design Documentation

## 1. System Vision & Platform Overview

**BirdVault Pro** is a client-side progressive web application (PWA) engineered specifically for wildlife photographers managing high-resolution DSLR and mirrorless photo collections ($24\text{--}60\text{ MP}$, $15\text{--}50\text{ MB}$ per JPEG/RAW file) directly inside modern web browsers.

The platform eliminates server-side image upload bottlenecks by combining local hardware acceleration, client-side binary parsing, machine learning vision models, and modern browser storage APIs:

* **Micro-Contrast Focus Culling Engine**: Automated $0\text{--}10$ sharpness evaluation based on spatial Discrete Laplacian operators, connected-component contour extraction, contrast-step normalization, and acutance scoring, mapped to $1\text{--}5$ star ratings.

* **Binary EXIF Header Extraction & TIFF IFD Parsing**: Fast byte-slice parsing of EXIF APP1 segments (`0xFFE1`) to extract camera body, lens, shutter speed, aperture, ISO, focal length, and hardware AF point tags (`SubjectArea 0x9214`).

* **Gemini AI Vision & Heatmap Overlays**: Multimodal species identification generating structured taxonomic records, conservation status, field marks, diagnostic trait descriptions, and spatial feature heatmap coordinates (`heatmapPoints`).

* **Real-Time API Quota & Token Monitor**: Live tracking dashboard for Google Gemini API limits—Requests Per Minute (RPM), Tokens Per Minute (TPM), and Requests Per Day (RPD)—complete with dynamic countdown timers for minute reset windows and UTC midnight resets.

* **Geographic Location Context & GPS Reverse Geocoding**: Automatic device location detection via the HTML5 Geolocation API integrated with OpenStreetMap Nominatim reverse geocoding, backed by an interactive autocomplete selector with global birding hotspot presets.

* **Universal 3-Tier Vault Storage Engine**: Cross-platform storage abstraction bridging the Desktop File System Access API, Apple iOS/Safari Origin Private File System (OPFS), and Web Share ZIP archives.

* **IndexedDB Micro-Thumbnail Persistence Engine (`birdvault_thumbs`)**: $240 \times 240\text{ px}$ micro-thumbnail binary caching that ensures instant grid re-population across app reloads without requiring folder re-selection.

---

## 2. High-Level System Architecture & Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       BIRDVAULT PRO WORKFLOW ENGINE                         │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                      1. Open / Ingest Photo Directory
                                       │
                                       ▼
                     2. Compute Fingerprint ID (F_ID)
                        Deduplicate & Parse EXIF APP1
                                       │
                                       ▼
                  3. Generate 240px Micro-Thumbnail
                     Cache in IndexedDB (birdvault_thumbs)
                                       │
                                       ▼
                  4. Automated Micro-Contrast Culling
                     Grade Sharpness (0–10) & Stars (1–5)
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        ▼                              ▼                              ▼
┌───────────────────────┐    ┌───────────────────┐    ┌───────────────────────┐
│ Action A: Keepers     │    │ Action B: Vault   │    │ Action C: Delete      │
├───────────────────────┤    ├───────────────────┤    ├───────────────────────┤
│ • Export to selected  │    │ • Gemini Vision   │    │ • Invoke native       │
│   output directory    │    │   Identification  │    │   fileHandle.remove() │
│ • Clear from workspace│    │ • Commit to OPFS  │    │ • Reclaim disk space  │
└───────────────────────┘    └───────────────────┘    └───────────────────────┘
```

---

## 3. Subsystem Specifications

### A. Binary EXIF Header Slicing & AF Point Parsing

Rather than loading multi-megabyte photo files into browser RAM, BirdVault Pro reads only the first $64\text{ KB}$ slice using `file.slice(0, 65536)` via `DataView`:

1. **APP1 Header Search**: Locates marker `0xFFE1` and verifies the `Exif\0\0` identifier (`0x45786966`).
2. **TIFF Header Alignment**: Evaluates byte order indicator (`0x4949` Little-Endian vs. `0x4D4D` Big-Endian).
3. **IFD Tag Extraction**:
   * `0x010F` / `0x0110`: Camera Make and Model.
   * `0x829A`: Exposure Time / Shutter Speed (e.g., $1/2000\text{s}$).
   * `0x829D`: F-Number / Aperture (e.g., $f/4.0$).
   * `0x8827` / `0x8833`: ISO Speed Rating.
   * `0x920A`: Focal Length in millimeters.
   * `0xA434`: Lens Specification string.
   * `0x9214` (`SubjectArea`): Converts hardware AF subject area coordinates into relative percentage bounding boxes $(x_{\%}, y_{\%}, w_{\%}, h_{\%})$ to anchor the initial focus evaluation region.

### B. Micro-Contrast Focus Evaluation Engine

Focus quality is calculated using a two-dimensional Discrete Laplacian convolution applied over grayscale luminance values ($Y = 0.299R + 0.587G + 0.114B$):

$$
L(x,y) = -4 I(x,y) + I(x-1,y) + I(x+1,y) + I(x,y-1) + I(x,y+1)
$$

The system calculates edge steepness and micro-contrast acutance across connected-component contour graphs:

1. **Sobel Gradient Magnitude**: Computes horizontal ($G_x$) and vertical ($G_y$) edge intensity:

$$
M(x,y) = \sqrt{G_x(x,y)^2 + G_y(x,y)^2}
$$

2. **Connected-Component Graph Traversal**: Groups contiguous edge pixels ($M \ge 20$) to isolate closed subject shapes (e.g., bird eyes, beak contours, feather barbules).
3. **Acutance Normalization**: Measures local contrast step $\Delta I = I_{\max} - I_{\min}$. High acutance requires steep gradients ($M \ge 45$) relative to contrast step ($\Delta I \ge 20$):

$$
\text{Acutance} = \frac{M^2}{\Delta I + 1}
$$

4. **Grade Calculation**: The final score $S \in [0, 10]$ maps directly to star ratings:
   * **Score 9–10**: ⭐⭐⭐⭐⭐ (Tack Sharp / 5 Stars)
   * **Score 7–8**: ⭐⭐⭐⭐ (Sharp / 4 Stars)
   * **Score 5–6**: ⭐⭐⭐ (Acceptable / 3 Stars)
   * **Score 3–4**: ⭐⭐ (Soft / 2 Stars)
   * **Score 1–2**: ⭐ (Blurry / 1 Star)

---

### C. Gemini AI Quota & Rate Limit Monitoring Subsystem

To manage client-side API requests against free tier restrictions, BirdVault Pro includes a persistent Quota Monitor (`TokenUsageMonitorModal`):

| Quota Metric | API Threshold Limit | Monitoring Implementation |
| :--- | :--- | :--- |
| **Requests Per Minute (RPM)** | $15\text{ Requests / Min}$ | Dynamic sliding window tracking requests in 60-second intervals. Displays visual progress bar and countdown timer. |
| **Tokens Per Minute (TPM)** | $1,000,000\text{ Tokens / Min}$ | Estimated payload token counting ($\approx 320\text{--}350\text{ tokens}$ per image request). |
| **Requests Per Day (RPD)** | $1,500\text{ Requests / Day}$ | Persistent daily counter tracked in `localStorage` (`birdvault_gemini_usage`). Displays UTC midnight reset countdown. |

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     GEMINI QUOTA MONITOR DASHBOARD                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ Requests Per Minute (RPM): [████████████░░░░░░░░] 9 / 15 (60%)             │
│ Minute Reset Countdown:    42s                                              │
│ Tokens Per Minute (TPM):   ~3,150 / 1,000,000 (<1%)                        │
│ Daily Requests (RPD):      [██░░░░░░░░░░░░░░░░░░] 124 / 1,500 (8%)          │
│ Daily UTC Reset Countdown: 12h 48m 15s                                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### D. Geographic Context & GPS Reverse Geocoding Engine

Location context significantly improves species identification accuracy by constraining candidate taxonomy to region-specific avifauna:

1. **HTML5 Geolocation Integration**: Captures the device position and reverse-geocodes it to a broad administrative region instead of retaining a street address.
2. **OpenStreetMap Nominatim Reverse Geocoding**: Prioritizes the state, province, or region and country (for example, *"Auckland, New Zealand"*) asynchronously:

$$
\text{GET } \texttt{https://nominatim.openstreetmap.org/reverse?lat=\{lat\}\&lon=\{lon\}\&format=jsonv2\&addressdetails=1}
$$

3. **Region-First Autocomplete**: Searches Nominatim administrative states (`featuretype=state`) so species grounding favors a state, province, or region rather than a street-address match. Users can also enter a location manually.
4. **Prompt Injection**: Injects geographical constraints into the Gemini Vision API prompt to filter out non-native confusion species.

---

## 4. Universal 3-Tier Vault Storage Engine

To ensure full compatibility across operating systems, browsers (Desktop Chrome, Safari, Firefox), and mobile PWA sandboxes, BirdVault Pro uses a 3-tier storage abstraction layer:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BIRDVAULT PRO UNIFIED VAULT ENGINE                       │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
            ┌──────────────────────────┼──────────────────────────┐
            ▼                          ▼                          ▼
┌────────────────────────┐ ┌────────────────────────┐ ┌────────────────────────┐
│   Tier 1: Desktop      │ │   Tier 2: Safari /     │ │   Tier 3: Mobile       │
│   File System Access   │ │   Mobile OPFS          │ │   ZIP / Share Sheet    │
├────────────────────────┤ ├────────────────────────┤ ├────────────────────────┤
│ • Chrome, Edge, Brave  │ │ • iOS Safari 15.2+     │ │ • iOS Files App        │
│ • Direct folder write  │ │ • macOS Safari 15.2+   │ │ • iCloud Drive         │
│ • Host directory pick  │ │ • Firefox, WebViews    │ │ • Google Drive         │
│   `Pictures/BirdVault` │ │ • Native OPFS tree     │ │ • Structured 1-tap     │
│                        │ │   sandboxed on disk    │ │   album ZIP export     │
└────────────────────────┘ └────────────────────────┘ └────────────────────────┘
```

### Storage Mechanism Comparison

| Feature / Metric | Tier 1: Host Directory | Tier 2: OPFS Directory | Tier 3: ZIP / Share Sheet |
| :--- | :--- | :--- | :--- |
| **API Target** | `window.showDirectoryPicker()` | `navigator.storage.getDirectory()` | `JSZip` + `navigator.share()` |
| **Supported Platforms** | Desktop Chrome, Edge, Brave | iOS Safari 15.2+, macOS Safari, Firefox, Android | Universal Mobile Fallback |
| **Storage Location** | User-selected OS folder | Origin Private File System sandbox | System Files App / iCloud / Drive |
| **Persistence** | Native disk files | Permanent sandboxed browser storage | Native mobile system export |

---

## 5. End-to-End Vault Ingestion & Memory Lifecycle

When the user taps **"Save / Update Lifer Vault"** inside the Photo Inspection Deck, the system executes a 5-phase transactional lifecycle:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        "SAVE / UPDATE LIFER VAULT" ACTION                              │
└───────────────────────────────────────────┬────────────────────────────────────────────┘
                                            │
                                            ▼
                  ┌──────────────────────────────────────────────────┐
                  │ 1. FILENAME & SIGHTING ID GENERATION             │
                  │ • ISO Date + Sighting ID (sighting_a1b2c3d)      │
                  │ • Target: 2026-10-04_sighting_a1b2c3d.jpg         │
                  └─────────────────────────┬────────────────────────┘
                                            │
                                            ▼
                  ┌──────────────────────────────────────────────────┐
                  │ 2. DUAL PHYSICAL STORAGE COMMIT                  │
                  │ • Primary: Write JPEG to OPFS Directory          │
                  │   /BirdVault_Lifer_Catalog/[Category]/[Species]/ │
                  │ • Secondary: Cache JPEG in IndexedDB             │
                  │   (BirdVault_Storage_DB -> vault_blobs)          │
                  │ • Write species_info.json in species directory   │
                  └─────────────────────────┬────────────────────────┘
                                            │
                                            ▼
                  ┌──────────────────────────────────────────────────┐
                  │ 3. MANIFEST & FINGERPRINT METADATA SYNC          │
                  │ • Prepend sighting to birdvault_entries_v5        │
                  │ • Update birdvault_fingerprints_v1 with          │
                  │   opfsFileName path pointer                      │
                  │ • Overwrite OPFS vault_manifest.json root file   │
                  └─────────────────────────┬────────────────────────┘
                                            │
                                            ▼
                  ┌──────────────────────────────────────────────────┐
                  │ 4. CULL WORKSPACE PURGE & GARBAGE COLLECTION     │
                  │ • Filter photo out of React state 'photos'       │
                  │ • Revoke ephemeral session Object URLs           │
                  │ • Free JavaScript heap memory from active RAM    │
                  └─────────────────────────┬────────────────────────┘
                                            │
                                            ▼
                  ┌──────────────────────────────────────────────────┐
                  │ 5. LIGHTBOX CLOSING & TAB TRANSITION             │
                  │ • setSelectedPhoto(null)                         │
                  │ • setActiveTab('vault')                          │
                  │ • Re-hydrate fresh session Object URL in Vault   │
                  └──────────────────────────────────────────────────┘
```

### Phase 1: Filename Generation & Path Sanitization
Constructs a collision-free filename combining ISO date and random sighting ID:
```javascript
const sightingId = 'sighting_' + Math.random().toString(36).substring(2, 9);
const fileName = `${new Date().toISOString().split('T')[0]}_${sightingId}.jpg`;
```
Sanitizes directory names via `sanitizeDirectoryName(name)` to strip slashes, diacritics, and special characters.

### Phase 2: Physical Disk Write (OPFS & IndexedDB)
Writes the JPEG binary into OPFS under `/BirdVault_Lifer_Catalog/[Category]/[Species]/[FileName]`. Concurrently caches the binary in IndexedDB (`vault_blobs` object store) to guarantee fallback access in sandboxed webviews.

### Phase 3: Manifest & Fingerprint Registry Sync
Updates `localStorage` registries (`birdvault_entries_v5` and `birdvault_fingerprints_v1`) with the relative path pointer (`opfsFileName`). Serializes the complete master index to `vault_manifest.json` in the OPFS root.

### Phase 4: Workspace Purge & Heap Memory Cleanup
Filters the photo out of active React state (`photos`), revokes temporary session URLs (`URL.revokeObjectURL`), and releases binary references to allow browser Garbage Collection to reclaim $\approx 15\text{--}30\text{ MB}$ of uncompressed RAM per photo.

### Phase 5: UI Transition & Vault Re-Hydration
Unmounts the Lightbox modal, switches views to the **Lifer List Vault** tab, and invokes `getOpfsSightingFileBlob` to generate a fresh session `blob:` URL for immediate card rendering.

---

## 6. Mobile Memory Architecture & Image Pyramid

To prevent WebKit silent memory eviction on mobile devices ($\sim 1\text{ GB}$ tab RAM cap), BirdVault Pro uses a 3-tier image pyramid:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     SOURCE FILE (24–60 MP RAW / JPEG)                   │
│               Stored as zero-memory 'File' handle in JS                 │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         ▼                           ▼                           ▼
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│  Tier 1: Thumb   │         │ Tier 2: Proxy   │         │ Tier 3: ROI Tile│
│  240 x 240 px   │         │  2048 x 2048 px │         │  1:1 Native Crop│
│  ~15 KB RAM     │         │  ~16 MB VRAM    │         │  ~4 MB VRAM     │
│  (Grid View)    │         │ (Full Lightbox) │         │ (Zoom / Loupe)  │
└─────────────────┘         └─────────────────┘         └─────────────────┘
```

* **Batch Page Window**: Rendered in batches of 20 items (`pageSize = 20`).
* **Active Buffer Window**: Pre-fetches screen proxies for current page items plus adjacent buffer items ($N-1$ and $N+1$).
* **LRU Eviction**: Off-page Blob URLs are revoked as cards scroll out of the active buffer.

---

## 7. App Reload & Re-Hydration Matrix

| Data / State Category | Persistence Engine | Reload & Re-Hydration Mechanics |
| :--- | :--- | :--- |
| **Lifer Vault Sightings** | Physical OPFS Disk + IndexedDB | **100% Retained**. `reloadWorkspaceAndVault()` queries physical OPFS storage, fetches binary handles, generates fresh session URLs, and renders species cards automatically. |
| **Workspace Micro-Thumbnails** | IndexedDB (`birdvault_thumbs`) | **100% Retained**. $240\times 240\text{ px}$ thumbnail blobs are fetched by `fingerprintId` key and re-hydrated into fresh session URLs without needing folder re-selection. |
| **Un-Vaulted Workspace Photos** | HTML5 `File` Handles (Transient RAM) | In-memory `File` objects are purged by browser security upon refresh. Re-selecting or dropping folders recalculates $F_{ID}$ fingerprints and re-attaches saved ratings, scores, and EXIF metadata. |
| **Gemini API Token Usage** | `localStorage` (`birdvault_gemini_usage`) | **100% Retained**. Tracks minute and daily API call counts across refreshes and enforces rate limits. |
| **User Location Settings** | `localStorage` (`birdvault_user_location`) | **100% Retained**. Restores last saved location string and GPS coordinates. |