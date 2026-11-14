# Scientific data sources and provenance

The bundle includes two **real, source-derived numeric series** in JSON, plus links to official NASA Scientific Visualization Studio references. The source observations, static image, and in-app graphic are different objects; we deliberately do not imply frame-perfect alignment where it does not exist.

| Track | Numeric data actually driving sound | Pinned sample | Unit | Official NASA EIC visual reference |
|---|---|---|---|---|
| Global temperature | NASA GISS GISTEMP v4, combined land–ocean temperature index | Annual 1880–2025, 146 points | °C anomaly relative to 1951–1980 | [SVS 5190, Climate Spiral](https://svs.gsfc.nasa.gov/5190/) |
| Arctic sea ice | NSIDC/NOAA G02135, Sea Ice Index version 4 | September monthly mean 1979–2026, 48 points | Million km² of extent | [SVS 5046, Daily Arctic Sea Ice](https://svs.gsfc.nasa.gov/5046/) |

## Temperature

- **Primary original source:** [NASA GISS GISTEMP v4](https://data.giss.nasa.gov/gistemp/) · [NASA downloadable CSV](https://data.giss.nasa.gov/gistemp/tabledata_v4/GLB.Ts+dSST.csv).
- **Bundled snapshot acquisition:** values transcribed as annual ("GISTEMP") records from the [datasets/global-temp GitHub archive, `data/annual.csv`](https://github.com/datasets/global-temp/blob/main/data/annual.csv), blob SHA `5f05609ae82780c64b52ff759ac724f68b5c1bd0`, accessed November 10, 2026. This archive mirrors NASA data, **not** an official NASA endpoint. A build-time script is included for future direct NASA refresh.
- **Processing:** filter `Source=GISTEMP`; copy `Year` and `Mean` to JSON `{year,value}` preserving the source's decimals. No interpolation, smoothing, synthetic values or adjustment of baseline. Retain 1880–2025 inclusive. For the direct NASA refresh, use the `J-D` annual column and skip incomplete years.
- **Scientific meaning:** annual global land–ocean *temperature anomaly*, not an absolute mean temperature, referenced to the 1951–1980 baseline. It does not represent temperatures at a selected geographic location.
- **Audio fixed encoding domain:** −0.7 to +1.5 °C, set deliberately in `data/temperature.json` and changeable separately from data updates. Values beyond that range are clamped in sound mapping; the chart still displays actual values.
- **Visual note:** the official NASA Climate Spiral is based on *monthly* anomalies. The in-app spiral is a new annual, schematic derived visualization and is not a copy of NASA's 60-fps frame sequence. The NASA image in the reference tile is a static still, not matched to the selected year.
- **Citation:** GISTEMP Team, 2026: GISS Surface Temperature Analysis (GISTEMP), Version 4, NASA Goddard Institute for Space Studies. Lenssen, N., et al. (2024), *A GISTEMPv4 observational uncertainty ensemble*, JGR Atmospheres, 129, e2023JD040179. Include access date when publishing.

## Arctic sea ice

- **Primary original source:** [NSIDC Sea Ice Index v4](https://nsidc.org/data/g02135/versions/4) · [official September extent CSV](https://noaadata.apps.nsidc.org/NOAA/G02135/north/monthly/data/N_09_extent_v4.0.csv).
- **Bundled snapshot acquisition:** annual September extent column from the publicly accessible [Environment Dashboard NSIDC Arctic September table](https://environmentdashboard.org/data/sea-ice-extent/nsidc/arctic-september), which identifies NSIDC v4 as its original source; web page claims an October 5, 2026 access date. Transcribed November 10, 2026. For formal scientific publication re-download directly from NSIDC and check for subsequent revised values.
- **Processing:** filter September records; copy year and *extent* (not area). All 48 years 1979–2026 are represented. Example anchor values: 1979 7.05; 2012 3.57; 2025 4.75; 2026 4.81 million km².
- **Scientific meaning:** September monthly *sea ice extent* (ocean area with at least 15% ice coverage), **not** ice thickness, volume, or sea ice *area*. This is an Arctic-wide statistic, not an ice concentration map.
- **Audio fixed encoding domain:** 2.5–8.5 million km². Higher extent maps to higher pitch, which is a perceptual design choice and does not encode a claim that higher is "better".
- **Visual note:** [NASA SVS 5046](https://svs.gsfc.nasa.gov/5046/) uses JAXA AMSR2 daily sea ice imagery; it is **not identical** to the NSIDC September mean extent record. The reference still is dated **2022** (SVS file `arctic_sea_ice_w_dates.2022001_print.jpg`). The in-app radial glyph is strictly schematic, not a geographic ice map.
- **Citation:** Fetterer, F., Knowles, K., Meier, W. N., Savoie, M., Windnagel, A. K., & Stafford, T. (2025), *Sea Ice Index, Version 4*. NSIDC, Boulder, Colorado. DOI: [10.7265/a98x-0f50](https://doi.org/10.7265/a98x-0f50).

## NASA visual imagery / links

- Climate Spiral still: `https://svs.gsfc.nasa.gov/vis/a000000/a005100/a005190/GISTEMP_Spiral_English_degC_2160p60.01402_print.jpg`, identified by [SVS page API 5190](https://svs.gsfc.nasa.gov/api/5190/).
- Arctic sea ice still: `https://svs.gsfc.nasa.gov/vis/a000000/a005000/a005046/arctic_sea_ice_w_dates.2022001_print.jpg`, identified by [SVS page API 5046](https://svs.gsfc.nasa.gov/api/5046/).
- **Hotlinked stills:** browser must be online to load these NASA reference images. There is no local copy; audio/data visualization still work offline. When an image fails, the UI retains the link to its official source. Credit must accompany reuse.

## More than just colors

The audio engine uses explicit measurements from the JSON datasets. The image is never treated as raw temperature or ice extent: false-color palettes and image annotations are not physically comparable across scientific products. The generated graphic and numeric labels are synchronized to the selected observation, while NASA's official still provides external visual context only.

## Refresh and reproducibility

Run `python scripts/refresh_data.py`. It reads the official NASA CSV's annual `J-D` column and the NSIDC CSV's `extent` column; validates monotonic years and numeric values before replacing local snapshots. The browser uses only JSON files packaged in `data/`. There is no need for authentication or API keys. Direct CSV downloads might fail in restricted network environments; in that case bundled snapshots remain valid as dated historical records. On refresh, update the text above with date, changed record count, upstream data revision notes, and the repository commit of the new data.
