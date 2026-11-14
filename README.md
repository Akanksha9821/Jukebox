# Earth Jukebox

**An interactive Earth science dashboard that turns measured changes in our planet into real-time synthesized sound.** A modular, no-key, no-dependency prototype inspired by NASA's Earth Information Center and NASA Space Apps Challenge. Not affiliated with or endorsed by NASA.

![Dashboard preview](./assets/dashboard-preview.png)

**Mobile preview:** [View mobile layout](./assets/mobile-preview.png)

## Quick start (two commands)

Requires **Node.js 18+**. Nothing to install from npm; audio synthesis uses built-in Web Audio. Alternatively, run `python -m http.server 4173` in this directory.

```bash
npm run dev
```

Open **http://127.0.0.1:4173** in Chrome, Edge, Safari or Firefox. (Python alternative: `http://localhost:4173`.) Do not double-click `index.html` as a `file://` URL; browsers block the dataset `fetch()` in that case.

Click **Play** in the bottom bar. Browser audio requires a user gesture. If the current observation is the last one, Play automatically starts at the first year; otherwise it starts at the selected year. If you press Stop, Play restarts from the selected point. Use the timeline or click the chart to jump to a year.

### Features

- **Two real datasets:** annual global temperature anomaly (1880–2025); Arctic September sea-ice extent (1979–2026). No fabricated records.
- **Real-time Web Audio** from selected measurements, with continuous scientifically mapped pitch or pentatonic musical quantization.
- **Tunable encoding** controls for pace, pitch span, timbre brightness, volume, and observation ticks. All configurable defaults and conversion functions are in their own modules.
- **Synchronized** live reading, timeline slider, interactive time-series graph and generated visual glyph.
- **Download** current dataset as CSV; **record** browser audio as WebM/Opus where MediaRecorder is supported.
- **Accessibility:** accessible labels, keyboard-operable controls, text for every value and optional speech-synthesis description.
- **NASA visual reference** links and previews, explicitly *not* year-matched; no claim that a static NASA still is the dynamically selected frame.
- **No credentials:** default experience runs on committed local JSON snapshots. NASA still images and external font may require internet but have local visual fallbacks.

## Folder map

```text
earth-jukebox/
├── index.html                 # Dashboard markup / semantic controls
├── src/
│   ├── main.js                # State, event binding, timeline, recording UI
│   ├── audio/
│   │   ├── presets.js         # ***** START HERE when tuning: defaults, base voice *****
│   │   ├── mappings.js        # ***** DATA -> SOUND: normalized value, pitch, timbre *****
│   │   └── engine.js          # Web Audio scheduling, synthesis, envelopes, capture
│   ├── data/loader.js         # Load/validate datasets, format readings, export CSV
│   ├── ui/
│   │   ├── visualizations.js  # Data-generated spiral/glyph and interactive chart
│   │   └── waveform.js        # Audio analyser waveform drawing
│   └── styles/app.css         # All responsive visual styling
├── data/
│   ├── temperature.json       # NASA GISTEMP annual values 1880–2025
│   └── ice.json               # NSIDC September extent 1979–2026
├── scripts/
│   ├── create_snapshots.py    # Reproduce pinned example datasets
│   └── refresh_data.py       # Fetch official NASA + NSIDC CSV; no key required
├── tools/dev-server.mjs       # Local dependency-free HTTP file server
├── tests/                     # Mapping, schema, and source parser tests
├── CODEX_HANDOFF.md           # Next Codex iteration and frame-sync work
├── AGENTS.md                  # Codex repository rules and constraints
├── assets/                    # Preview screenshots
├── DATA_SOURCES.md            # Scientific data lineage, known caveats, source URLs
├── AUDIO_MAPPING.md          # Formulae, synthesis chain, safe tuning workflow
└── README.md                  # This guide
```

## How the app works

`data/*.json` → `src/data/loader.js` → `src/main.js` → simultaneously:

1. `src/ui/visualizations.js` draws the spiral or extent glyph and time-series chart.
2. `src/audio/mappings.js` maps the *same measured value* to frequency and timbre parameters, with a **fixed per-dataset range** to keep years comparable.
3. `src/audio/engine.js` schedules oscillator notes through an audio envelope, gain, compressor, analyser and speakers / recording destination.

The sound **does not** claim to be a literal recording of Earth. It is an explicit, reversible design choice translating numerical data to sound. The imagery in the "NASA visual reference" tile is independently credited and not aligned to the timeline. In particular, the EIC climate spiral uses **monthly** GISTEMP data, whereas our bundled listening track uses **annual** mean values. The app's own graphic is generated from the actual annual series.

## Audio fine-tuning — recommended order

1. Edit `src/audio/presets.js` to modify the default UI settings, lowest MIDI note, maximum semitone span, envelope, and secondary pad/tick volume.
2. Edit **`encodeObservation()`** in `src/audio/mappings.js`. This is where you change how scientific quantities map onto pitch/timbre; the function has no browser dependencies and is unit-tested.
3. Adjust synth voice types, compressor and note duration in `src/audio/engine.js`, while preserving the envelope and low default master gain.
4. Listen in Scientific mode, then Musical mode. Scientific is continuous; Musical rounds to C-major pentatonic and adds a quiet fifth.
5. Run `npm test` and check the mapping invariants. Keep a mapping legend and document all scale changes if you modify the scientific interpretation.

See **[AUDIO_MAPPING.md](./AUDIO_MAPPING.md)** for equations, file references, and safety notes.

## Updating the datasets

The package ships with **pinned, real source-derived snapshots** so it runs without networking or cross-origin browser requests. To refresh against the original publishers:

```bash
python scripts/refresh_data.py
# or individually:
python scripts/refresh_data.py --dataset temperature
python scripts/refresh_data.py --dataset ice
```

The refresh script uses only Python's standard library and direct public CSV downloads; **no NASA Earthdata account or API key is needed for these two products**. The script validates that years are continuous and replaces only successfully fetched files. Some NASA and NSIDC endpoints may be restricted by a network; pinned snapshots keep the app working. Always review updated data for revisions, especially where satellite inputs change. After refresh, rerun `npm test` and update `DATA_SOURCES.md` with your acquisition date.

If you later add **NASA FIRMS active fire** observations, that API requires a free FIRMS MAP_KEY. Do not commit keys; create a server-side proxy with environment variables rather than exposing secrets to the browser. The currently implemented tracks need no credentials.

## Tests

```bash
npm test
```

Tests cover numerical mappings, JSON integrity, selected known observations, and CSV parser behavior. There is no external npm dependency.

## Recording notes

Click **Record audio**, click **Play**, then **Stop recording** to export a browser-recorded file. MIME type depends on browser (`.webm` in Chrome/Edge, possibly `.mp4` in Safari). The file records the synth output, not the optional spoken descriptions. In unsupported browsers recording gracefully reports an error. For high-quality WAV/FLAC exports, add an OfflineAudioContext renderer later.

## Limitations and roadmap

**Current v0.1 is a strong interactive scientific sonification dashboard**, not a full frame-by-frame NASA EIC video sonifier. It synchronizes measurements with **original derived graphics** and includes NASA visualization reference stills. It does **not** decode each source video's frames, infer the scientific quantity from NASA map pixels, or guarantee a frame-accurate correspondence with NASA's published EIC animations. Future additions:

- Fetch NASA SVS video or frames *and their source quantities*, then align timestamps through an explicit frame-to-measurement mapping.
- Offer different regional climate datasets / spatial audio and multiple scientific layers.
- Add data-sonification comprehension studies with blind and low-vision users.
- Optional live data refresh through a server or build job, not from browser cross-origin scraping.
- Provide offline NASA image assets only after reviewing media credit/usage requirements.

## Credits

NASA GISS/GISTEMP v4, the National Snow and Ice Data Center (NSIDC) Sea Ice Index v4, and NASA's Scientific Visualization Studio. See [DATA_SOURCES.md](./DATA_SOURCES.md) for original links and exact interpretation. NASA visualizations are copyrighted or credited as specified on their pages; follow NASA's [media usage guidelines](https://www.nasa.gov/nasa-brand-center/images-and-media/). No NASA logos are used here. Dashboard code is original and built independently for educational use.
