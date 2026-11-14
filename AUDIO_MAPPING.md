# Audio encoding — what to change, and why

The core goal is **deterministic, explainable sonification**, not adding unrelated music to a visualization. Sound is synthesized in real time in the user's browser from the same numeric observation the interface shows.

## Mathematical mapping

In `src/audio/mappings.js`:

1. Pick the selected observation `v`.
2. Normalize against a **fixed** dataset-specific audio domain `[a,b]`: `n = clamp((v-a)/(b-a), 0, 1)`.
3. Compute `M = lowestMidi + n * maxSemitones * (pitchSlider/100)`.
4. In **Scientific mode**, keep `M` continuous; in **Musical mode**, quantize `M` to its closest C-major-pentatonic MIDI note.
5. Convert MIDI to frequency `f = 440 * 2^((M-69)/12)` hertz.
6. Set filter cutoff as a function of the **brightness control**, not a new physical variable: `cutoff = 800 + (brightness/100)*(4800-800)` hertz.
7. Increase voice amplitude gently with `n`: `amplitude = 0.25 + 0.35*n`. Volume slider and master safety gain apply afterward.

This means **higher numerical values** yield **higher frequencies** in both datasets. For sea ice, lower ice extent becomes a lower note. The mapping is a chosen accessibility technique, not a normative assessment of whether higher/lower numbers are good or bad.

### Fixed calibration

The per-track `soundDomain` is set in `data/{track}.json`. Avoid re-normalizing by min/max of each selected year subset: that would make two different decades sound deceptively similar. If future extreme values exceed domain, pitch mapping clamps while numeric readings remain truthful; revise domain and document any change in musical comparison tests.

## Synth voice routing

`src/audio/engine.js`:

```text
value → encodeObservation() → oscillator (sine / triangle)
                             → lowpass → ADSR-like gain envelope
       musical mode only     → quiet fifth harmonic oscillator
       tick enabled          → short transient oscillator
                             → master gain (square law; conservative)
                             → compressor → analyser → speakers
                                                        └→ MediaStream recorder
```

The example instrument is intentionally sparse. The optional fifth and tick can be disabled or tuned independently. Changing timbre changes the *presentation* of the data but does not change the observed value. The scheduler prepares notes slightly ahead of time using `AudioContext.currentTime`; the UI updates at the scheduled timestamp.

## Tuning locations

| Change | File | Symbol |
|---|---|---|
| Initial UI control positions | `src/audio/presets.js` | `DEFAULT_SETTINGS` |
| Lower frequency and span | `src/audio/presets.js` | `VOICE.lowestMidi`, `VOICE.maxSemitones` |
| Scientific formula (pitch, amplitude, timbre) | **`src/audio/mappings.js`** | **`encodeObservation()`** |
| Musical scale | `src/audio/presets.js`, `src/audio/mappings.js` | `PENTATONIC_INTERVALS`, `nearestPentatonic()` |
| Oscillator waveform | `src/audio/engine.js` | `playObservation()` / `osc.type` |
| Attack, decay, release | `src/audio/presets.js`, `src/audio/engine.js` | `VOICE.*`, `playObservation()` |
| Observation tick | `src/audio/presets.js`, `src/audio/engine.js` | `tickGain`, `tickHz` |
| Scheduling tempo and latency | `src/audio/engine.js` | `schedule()` |
| Master gain & compressor | `src/audio/engine.js` | `setVolume()`, `ensureAudio()` |
| UI controls / labels | `index.html`, `src/main.js` | `pace`, `pitch`, `brightness`, `volume` |

## Good evaluation questions

- Can a listener correctly identify which of two years had a higher anomaly/extent based on the sound alone?
- Can the listener tell a trend from a single anomalous year?
- Do people confuse louder with higher temperature? If so, remove or fix amplitude coupling.
- Can a participant operate Play, pause, describe, and change observations by keyboard only?
- Are controls calibrated and explained with a sound legend, including the domain and units?
- Does the selected year label match the actual audio note and chart point?

For rigorous comparison, use the continuous Scientific mode, a fixed volume, a fixed domain, and no musical fifth or optional tick. Always conduct listening at safe levels; strongly recommended to begin at the default 45% master setting or below.

## Important non-features

- No generative music model, data inference from false-color NASA imagery, and no assertion of causality beyond the source record.
- No lat-long panning; there is no geographic point dimension in these global or Arctic-wide time series.
- No need for credentials or API keys to synthesize sound from the bundled records.
- Audio exported as a browser-supported MediaRecorder stream, **not** lossless .wav; speaker / browser permission constraints apply.
