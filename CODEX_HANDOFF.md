# Codex handoff — next iteration after listening review

This folder is already a working, tested dashboard. Codex can open it as a repository and run `npm run dev` to start the app. Before editing, read `AGENTS.md`, `README.md`, `DATA_SOURCES.md`, and `AUDIO_MAPPING.md`.

**Your first follow-up task:** listen to both tracks on headphones using Scientific and Musical modes. Have the project owner specify what is difficult to hear, or what science meaning should be encoded differently. Do not change mappings before listening feedback.

When tuning the audio:
- Edit `src/audio/mappings.js` (data → normalized pitch/timbre), `src/audio/presets.js` (defaults/envelope/scales), or `src/audio/engine.js` (synthesis, scheduling, recording). Avoid unrelated UI rewrites.
- Keep every visualization and live reading synchronized to the selected observation.
- Preserve fixed numeric domains until the owner explicitly approves changes and update legends/documents on any change.
- Document new mappings in `AUDIO_MAPPING.md`; give the owner a concise description of what changed in the music and why.
- Ensure no clipped notes, abrupt loudness changes, or intrusive 5-second playback loops. Start quietly.
- Run `npm test` and `python tests/browser_smoke_dev.py` (latter optional, requires Playwright).

**Potential next milestone:** true NASA SVS EIC **frame-to-data synchronization**. This is not yet implemented. For the Climate Spiral the official NASA animation uses monthly GISTEMP while the bundled source data uses annual GISTEMP; a faithful frame-by-frame version must first add the monthly measurements with exact timestamps and verify the selected frame correspondences, then let the audio play as the video time advances. For Arctic ice, NASA's daily animation depicts AMSR2 ice concentration and the sonified numeric series is NSIDC September extent, so these cannot be treated as synchronized measurements without selecting a corresponding daily dataset and addressing data-product differences.

**API keys:** current dashboard does not need any keys. Ask the owner before implementing any data API that requires authorization; prefer documented public, no-key datasets first. Never include credentials in a browser bundle.
