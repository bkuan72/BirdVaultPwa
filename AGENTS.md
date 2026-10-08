# Repository guidance

## Project shape

- BirdVault Pro is a local-first browser application. The main application and most of its behavior live in `index.html`.
- React and Tailwind are loaded at runtime from CDNs; the app does not currently have a package manifest, build pipeline, or automated test suite.
- Supporting project documentation is in `README.md` and `docs/`. Keep it aligned with user-visible behavior and implementation changes.

## Implementation guidelines

- Preserve the browser-native, local-first design. Do not introduce a backend or build tooling unless the task requires it.
- Treat image files, EXIF metadata, crop boxes, zoom/pan state, object URLs, and persisted vault records as related data. When changing image processing or versioning, preserve original image dimensions and user framing unless the requested behavior explicitly changes them.
- Preserve the existing storage boundaries: localStorage for lightweight metadata and preferences, IndexedDB for binary cache, and OPFS for vault media where supported.
- Handle browser API availability and failures explicitly. Avoid silently substituting success-shaped values when storage or image operations fail.
- Keep changes focused, follow the existing style in `index.html`, and avoid unrelated refactors.
- Never put API keys, credentials, or private photo data in source code, documentation, logs, or commits.

## Validation

- There is currently no configured automated build, lint, or test command. Do not add tooling for routine changes unless the task calls for it.
- For changes to `index.html`, perform a targeted syntax or behavior check where feasible. For UI, storage, and image-processing changes, verify the affected browser workflow when a suitable browser runtime is available.
- Update the relevant documentation when a change affects setup, architecture, storage, or user-facing behavior.

## Git workflow

- Preserve pre-existing worktree changes; inspect `git status` before staging or committing and stage only files related to the requested change.
- When asked to commit and push, create the requested commit but do not push. The user will push manually unless they explicitly ask otherwise.
