# BirdVault Pro Developer Guide

## Project summary

BirdVault Pro is a single-file browser app for wildlife photographers and birders. It handles photo curation, sharpness scoring, AI species identification, and local vault management.

The codebase is intentionally lightweight and browser-native. Most logic lives in a single HTML file and run-time rendering is handled with React from CDN libraries.

## Repository layout

- `index.html` — main application source
- `README.md` — project overview and usage information
- `docs/ARCHITECTURE.md` — architecture and design summary
- `docs/DEVELOPER_GUIDE.md` — developer setup and operations guide
- `birdvault_detailed_documentation (1).md` — extended design document snapshot

## Local development

Because this project is a browser app, local development is simple:

1. Open the project folder in a browser or launch a local static server.
2. Serve the folder with any static server, for example:

```bash
cd /workspaces/BirdVaultPwa
python3 -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

## Recommended browser

Use a modern Chromium-based browser for the best compatibility:
- Google Chrome
- Microsoft Edge
- Brave
- Opera

This project depends on:
- File System Access API
- Canvas APIs
- localStorage
- IndexedDB
- OPFS support when available

## Required external dependency

The app requires a Gemini API key for AI species identification. The user must provide it in the browser UI or via local configuration.

## Runtime expectations

The app will function best when:
- the browser supports local file access APIs
- cookies/localStorage are enabled
- the user has adequate filesystem permissions for folder selection
- the device has enough RAM for high-resolution preview processing

## Common troubleshooting

### 1. API identification fails
Check:
- Gemini API key is set
- key is valid and active
- quota still remains
- network access exists

### 2. Folder import fails
Check:
- browser supports File System Access API
- permissions were granted to the selected directory
- file format is supported by the browser

### 3. Missing thumbnails or stale state
Clear browser local data for the site or delete the stored metadata keys.

### 4. Export fails
Verify:
- the browser allows file creation and downloads
- a directory was selected if requesting local export
- the vault contains files to archive

## Extending the app

Potential next steps for a real production version:
- move state validation into typed schemas
- add a proper build system instead of single-file runtime JSX
- separate UI components into modules
- add secure backend storage for API key and user data
- add cloud sync or multi-device support
- add tests around metadata parsing and export flows

## Notes on architecture

This project is intentionally kept small and direct. That makes it easy to run, but it also means:
- most logic lives in one main HTML file
- browser compatibility matters significantly
- there is no backend persistence layer
- the app is designed for an individual workflow rather than a shared multi-user service

## Suggested documentation conventions

When changing the project, keep documentation aligned with:
- user-facing features
- storage and browser API assumptions
- AI limits and quota behavior
- local-first data model constraints

## Summary

BirdVault Pro is a browser-native wildlife review and bird catalog tool. It is optimized for local-first use and for a single user’s toolkit, not for enterprise or multi-user deployment.
