# Codex project instructions — Earth Jukebox

Goal: maintain a credible, accessible, scientifically grounded dashboard that maps published NASA/NSIDC observations into real-time sound. Change the sonification with explicit formulas and tests; do not obscure sources or replace observations with synthetic/generated values.

## Before changing code

1. Read `README.md`, `DATA_SOURCES.md`, `AUDIO_MAPPING.md`.
2. Review audio flow: `data/*.json` → `src/data/loader.js` → `src/audio/mappings.js` → `src/audio/engine.js`.
3. Run `npm test` before and after. Requires Node 18+ and Python 3.9+. No npm install step.

## Development instructions

- Keep mapping math in `src/audio/mappings.js` **pure** and testable, parameter defaults in `src/audio/presets.js`, DSP/scheduling in `src/audio/engine.js`. Do not embed mapping math in UI components.
- Maintain scientifically accurate labels and units; global temperature values are **anomalies** relative to 1951–1980, Arctic **sea ice extent** is not ice area or thickness.
- Any new satellite visual must label acquisition date, source and any false-color mapping. Do not describe source stills as time-aligned with observations until there is an explicit timestamp match.
- Sound must be reproducible from the input data and mapping configuration. If adding generative AI, make it optional and avoid claiming its music encodes exact measurements without validation.
- Don't expose API keys to client code or commit `.env` files. Current tracks need no key. Future FIRMS key integration should use a backend proxy.
- Keep keyboard access, ARIA, text readings and narration. Audio alone is not an accessible interface.
- Use conservative audio volume, smoothing and short envelopes. Never autoplay sound before a user gesture.
- When adding sources, extend `DATA_SOURCES.md` with original publisher, source URL, retrieval date, units, field name, baseline, data revisions, license/credits, and visual/sound synchronization rules.
- Keep data snapshots deterministic; update with `python scripts/refresh_data.py` and ensure tests reflect upstream changes. Don't silently overwrite original values.

## How to run

`npm run dev` opens `http://127.0.0.1:4173`. `npm test` runs Node tests and Python tests. To publish to a static host, copy the full project directory and configure the host to serve `index.html` at `/` with ES modules/JSON content types; no build tool needed.

## Suggested next development sprint

1. Build a time-aligned player from the NASA SVS API's frame metadata and matching observations. Record provenance and verify frame-to-observation timestamps.
2. Add an `A/B` compare sonification preserving fixed absolute numeric ranges.
3. Add a per-track calibration legend for selected note and range.
4. Run task-based non-visual tests with blind/low-vision collaborators.
5. Add offline render to lossless WAV via `OfflineAudioContext`.
