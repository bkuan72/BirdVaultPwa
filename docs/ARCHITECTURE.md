# BirdVault Pro Architecture

## Overview

BirdVault Pro is a browser-first wildlife photo workflow application. It helps users:
- import bird photos from the local filesystem
- cull and rate images by technical sharpness
- inspect detail with a zoomable viewer and ROI crop tools
- identify species with the Gemini Vision API
- save species and sightings into a local life-list vault
- export the vault and media as a portable ZIP package

This project is intentionally local-first. It does not rely on a backend server for image processing or persistence.

## High-level architecture

BirdVault Pro uses a single-page React app running in the browser. The runtime includes:
- React 18 UI components
- Tailwind CSS for styling
- Canvas APIs for crop and focus analysis
- File System Access API for folder/file import
- IndexedDB for thumbnail/blob cache
- Origin Private File System (OPFS) for vault media storage
- localStorage for metadata and API configuration
- Google Gemini API for species identification

## Core workflow

1. Import photos from local disk
2. Generate fingerprint metadata and thumbnails
3. Evaluate focus/sharpness
4. Review images in the interactive viewer
5. Run AI species identification
6. Save successful results into the life-list vault
7. Export backup/archive files

## Main app modules

### 1. Culling workspace
Responsible for:
- file/folder selection
- photo list state in the order returned by the browser picker
- incremental folder loading and visible-page thumbnail generation
- filtering and sorting
- selection and deletion
- star ratings and focus score display

This is the user’s main review queue for a large set of wildlife photos.

### 2. Photo inspector
Responsible for:
- zoom and pan
- loupe magnification
- crop selection ROI
- sharpness evaluation
- location metadata
- AI identification workflow
- save-to-vault actions

This is the detailed review screen used to inspect an individual image.

### 3. Lifer vault
Responsible for:
- species catalog storage
- category and search filtering
- sightings timeline
- on-demand previews for visible species and selected sightings, with bounded OPFS reads
- metadata display
- ZIP export of the vault

This is the long-term archival and taxonomy component of the system.

## Data model

### Photo object
Each imported photo includes metadata such as:
- id / fingerprintId
- file name and file handle
- size and last modified timestamps
- blob and thumbnail URLs
- sharpness score
- user star rating
- focus spot and crop box
- EXIF-derived camera data
- AI identification result
- OPFS file references and versions

### Vault entry
Each vault record contains:
- speciesName
- scientificName
- category
- description
- habitat
- conservationStatus
- diet
- keyTraits
- sightings[]

### Sightings
Each sighting may retain:
- date and location
- image references
- version metadata
- local-file storage metadata

## Storage model

### localStorage
Used for:
- Gemini API key
- user preferences
- quota counters
- metadata indexes
- persisted photo fingerprint records

### IndexedDB
Used for:
- thumbnails
- cached blobs
- reusable image assets without re-reading the filesystem

### OPFS
Used for:
- saved species images and vault media
- local archival files tied to sightings

### ZIP export
Used for:
- portable backup of the full vault
- preserving media and manifest metadata outside browser storage

## Focus culling logic

The app estimates sharpness by analyzing the image’s edge structure and contrast. It converts to luminance and evaluates gradient intensity, subject-area edge concentration, and local contrast. The output is a score between 0 and 10, mapped to a 1–5 star rating.

This supports fast triage of large wildlife photo sets without manual review of every frame.

## AI identification flow

1. User chooses a photo or region of interest.
2. The app crops the image if a crop box is defined.
3. The cropped image is converted to JPEG and sent to Gemini.
4. A location string is included to ground species predictions in the local geography.
5. Gemini returns structured JSON containing species metadata and a diagnostic heatmap.
6. The app stores the identification result and optionally saves it to the vault.

## Security and operational considerations

This app is built for a single user, not a multi-user production backend. Important considerations:
- the Gemini API key is stored in browser localStorage
- client-side access to the API key is exposed to the browser environment
- no server-side auth or backend validation is used
- operations depend on browser capability and user permissions

## Design intent

BirdVault Pro is a local-first wildlife photo assistant that blends:
- asset management
- image quality review
- AI taxonomic identification
- life-list archiving
- exportable backup workflows

It is best understood as a personal birding and photography operations tool rather than a traditional SaaS application.
