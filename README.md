# mzML_explorer
a web application for inspecting raw mass spectrometry mzML files.
no peptide identification. no database search. just instrument data.



**Interactive raw mass spectrometry data inspection — no identification, no inference, just the instrument signal.**

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-black.svg)](https://www.python.org/)
[![Version](https://img.shields.io/badge/version-1.0.0-black.svg)](CHANGELOG.md)

---

## Scientific Motivation

Mass spectrometry experiments begin long before any peptide identification or statistical analysis. Every downstream inference — protein quantification, differential expression, post-translational modification calling — rests entirely on the quality of the raw instrument signal. Yet the tools available for inspecting that signal are either locked inside vendor software, require full pipeline installations, or present data through search-result lenses that collapse important acquisition-level information.

mzML Explorer inspects the raw content of any mzML file: chromatograms, scan metadata, and mass spectra to accelerate the quality-control (QC) step that precedes every serious quantitative proteomics experiment.

---

## Overview

mzML Explorer is a self-hosted web application for interactive inspection of raw mass spectrometry data in the mzML open format. It parses any mzML 1.1.0-compliant file — including files converted from Thermo `.raw`, Waters `.raw`/`.wiff`, Bruker `.d`, and other vendor formats via ProteoWizard — and presents the acquisition metadata as a navigable, interactive interface.

The application runs entirely on your local machine. No data ever leaves your computer.

**Core capabilities:**

- Total Ion Chromatogram (TIC) and Base Peak Chromatogram (BPC) visualization
- Paginated, filterable scan table with MS level, retention time, and precursor metadata
- Interactive stick-plot spectrum viewer with per-peak hover details
- CSV export of summaries, scan metadata, and individual spectra

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Memory-efficient parsing** | Binary m/z/intensity arrays are never loaded during the scan index pass; only scalar metadata is stored |
| **Interactive chromatogram** | Plotly-powered TIC and BPC with zoom, pan, hover, and linked scan navigation |
| **Filterable scan table** | Filter by MS level, retention time range, or scan number; paginated at 200 rows |
| **MS2 precursor details** | Precursor m/z, charge state, and isolation window displayed for each MS2 spectrum |
| **Privacy by design** | All processing is local; no data is transmitted to any external server |

---

## Workflow

```
Raw Instrument File                         mzML Explorer
(.raw / .wiff / .d)                              │
        │                                         │
        ▼                                         │
  ProteoWizard                                    │
   msConvert                                      │
        │                                         │
        ▼                                         │
   .mzML file ──── Upload ─────────────────► Parse scan index
                                                  │
                          ┌───────────────────────┤
                          │                       │
                          ▼                       ▼
                   File Summary            Chromatogram
                   (scan counts,          (TIC / BPC,
                    RT range,              interactive,
                    max TIC)               zoomable)
                          │
                          ▼
                     Scan Table
                   (filter by MS level,
                    RT range, scan no.)
                          │
                          ▼ (click any row)
                   Spectrum Viewer
                   (stick plot, hover
                    peak details, MS2
                    precursor info)
                          │
                          ▼
                    CSV Export
                   (summary / scans /
                    individual spectrum)
```

---

## Requirements

- **Python** 3.10 or newer
- **pip** (bundled with Python)
- **Internet connection** at page load (for Plotly.js CDN)
- **A modern browser** (Chrome, Firefox, Safari, Edge)

No conda environment, Docker container, or R installation is required. All Python dependencies are installed automatically.

---

## Installation

```bash
# Clone the repository
git clone https://github.com/your-username/mzml-explorer.git
cd mzml-explorer

# Make the start script executable (first time only)
chmod +x run.sh

# Start the application
./run.sh
```

Then open **http://127.0.0.1:8000** in your browser.

The `run.sh` script will:
1. Verify Python 3.10+ is available
2. Create a virtual environment at `.venv/` (first run only, ~30 seconds)
3. Install all dependencies from `backend/requirements.txt`
4. Start the FastAPI/uvicorn server with hot-reload enabled

Press `Ctrl+C` in the terminal to stop the server.

> **Important:** Open `http://127.0.0.1:8000` in your browser. Do **not** open `frontend/index.html` directly — it must be served through the backend for API calls to work.

---

## Required Input Format

mzML Explorer accepts files in the **mzML 1.1.0** open format, maintained by the HUPO Proteomics Standards Initiative (HUPO-PSI). Both indexed and non-indexed mzML files are supported.

Most mass spectrometry data acquisition software produces proprietary binary formats. To convert to mzML, use **ProteoWizard msConvert**, the community standard:

| Vendor format | Instrument family | msConvert input |
|---------------|------------------|-----------------|
| `.raw` (Thermo) | Orbitrap, LTQ, TSQ | `--filter "msLevel 1-2"` |
| `.raw` / `.wiff` (Waters/SCIEX) | Q-TOF, Xevo, TripleTOF | standard conversion |
| `.d` (Bruker) | timsTOF, maXis | standard conversion |
| `.lcd` (Shimadzu) | LCMS series | standard conversion |

**Example msConvert command:**

```bash
msconvert sample.raw --mzML --filter "peakPicking true 1-" -o ./output/
```

Files produced by the ProteoWizard test suite and the PRIDE/MassIVE public repositories are confirmed compatible.

---

## Analysis Pipeline

### Step 1 — Upload and parse

Click **"Drop an .mzML file here, or click to browse"** and select your file, then click **"Analyze file"**. The backend streams the file to a temporary directory in 4 MB chunks, then makes a single sequential pass through all spectra to build a lightweight scan index. Only scalar metadata (scan number, retention time, MS level, TIC, base peak intensity, precursor m/z) is stored. Raw m/z and intensity arrays are not decoded at this stage.

Parsing a 36 MB Waters LC-MS file containing ~2,000 MS1 scans typically completes in 2–5 seconds.

### Step 2 — File summary

After parsing, a summary grid displays:

- **Total scans** — complete acquisition count
- **MS1 / MS2 scans** — breakdown by level
- **RT start / end / range** — acquisition time window in minutes
- **Average scan rate** — scans per minute
- **Max TIC** — largest total ion current observed
- **Max base peak** — largest base peak intensity observed

Click **↓ Download CSV** to save the summary.

### Step 3 — Chromatogram

The Total Ion Chromatogram (TIC) is plotted immediately after parsing. Toggle to **Base peak** to view the Base Peak Chromatogram (BPC). Both plots support zoom, pan, and per-point hover displaying scan number, retention time, and signal intensity.

### Step 4 — Scan table

All scans are listed in a paginated, filterable table (200 rows per page). Available filters:

- **MS level** — All, MS1, or MS2
- **RT range** — minimum and maximum retention time in minutes
- **Scan number** — specific scan or range

The **↓ Download CSV** button exports the current filtered view as a flat-file table of scan metadata.

### Step 5 — Spectrum viewer

Click any row in the scan table to load that scan's spectrum. The backend retrieves only that scan's m/z and intensity arrays using pyteomics-indexed random access — the file is sought directly to the correct byte offset without re-reading the entire file. Peaks are rendered as a vertical stick plot. For MS2 spectra, precursor m/z, charge state, and isolation window boundaries are displayed above the plot.

Click **↓ Download CSV** to save the spectrum as a two-column table (mz, intensity).

---

## Downloadable Outputs

| Output | Format | Contents |
|--------|--------|----------|
| File summary | CSV | Scan counts, RT range, max TIC, max base peak |
| Scan table | CSV | Per-scan: scan number, RT, MS level, TIC, base peak, precursor m/z, charge |
| Spectrum | CSV | Per-peak: m/z, intensity |

All exports are generated client-side (scan table) or streamed directly from the local FastAPI server (summary and spectrum). No data is sent to any external service.

---

## Biological Interpretation

mzML Explorer reports acquisition-level instrument metadata, not identified biological entities. The values displayed reflect what the mass spectrometer measured during the experiment:

**Total Ion Chromatogram (TIC):** The summed intensity of all ions detected in each MS1 scan across the gradient. A stable, bell-shaped TIC suggests consistent electrospray and uninterrupted chromatography. Sudden drops indicate spray instability, column void, or acquisition gaps.

**Base Peak Chromatogram (BPC):** The intensity of the single most abundant ion per scan. The BPC is more sensitive to discrete high-abundance species and often reveals co-eluting contaminants that the TIC obscures.

**MS2 precursor m/z and isolation window:** These values define exactly which peptide precursor ion was selected for fragmentation. Comparing the isolation window against the chromatogram helps identify whether co-isolation of interfering species is likely — a critical quality consideration for TMT/iTRAQ isobaric labeling experiments.

**Scan rate:** Average scans per minute reflects the duty cycle of the acquisition method. An unexpectedly low scan rate may indicate a malfunctioning dynamic exclusion setting or an overly narrow m/z range.


---

## Intended Use and Scope

mzML Explorer is designed for the **quality-control step that precedes computational proteomics analysis**. It is appropriate for:

- Verifying that an acquisition ran to completion with the expected number of scans
- Confirming that the chromatographic gradient performed as expected
- Inspecting individual spectra for signal quality before submitting to a search engine
- Diagnosing acquisition artifacts (spray loss, column void, calibration drift) from the raw signal


mzML Explorer is **not** intended for:

- Peptide or protein identification
- Quantification (label-free, TMT, SILAC, or other)
- Statistical analysis or differential expression

For full proteomics analysis pipelines, consider [FragPipe](https://fragpipe.nesvilab.org/), [MaxQuant](https://www.maxquant.org/), [Proteome Discoverer](https://www.thermofisher.com/), or [Skyline](https://skyline.ms/).

---

## Roadmap

The following features are planned for future releases. Community contributions toward any of these are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).

**v1.1 — Enhanced inspection**
- [ ] Multi-file comparison view (overlay TIC traces from multiple acquisitions)
- [ ] Scan-to-scan navigation from within the spectrum viewer
- [ ] Automatic detection and flagging of acquisition gaps in the chromatogram

**v1.2 — Search result overlay**
- [ ] Optional overlay of PSM identifications from a [mzIdentML](https://www.psidev.info/mzidentml) or [pepXML](http://tools.proteomics.ucsd.edu/) file onto the spectrum viewer
- [ ] Annotation of matched fragment ions (b/y series) in the stick plot

**v1.3 — QC report generation**
- [ ] Exportable PDF QC report summarizing acquisition metrics with embedded chromatogram figures
- [ ] Batch processing mode: parse a directory of mzML files and produce a comparative QC table

**v2.0 — Extended format support**
- [ ] mzXML support
- [ ] Direct reading of Thermo `.raw` files via [RawFileReader](https://github.com/thermofisher/rfd) (Windows only)
- [ ] imzML support for mass spectrometry imaging datasets

---

## References

1. Martens, L. *et al.* (2011). mzML — a Community Standard for Mass Spectrometry Data. *Molecular & Cellular Proteomics*, 10(1), R110.000133. https://doi.org/10.1074/mcp.R110.000133

2. Goloborodko, A. A., Levitsky, L. I., Ivanov, M. V., & Gorshkov, M. V. (2013). Pyteomics — a Python Framework for Exploratory Data Analysis and Rapid Software Prototyping in Proteomics. *Journal of The American Society for Mass Spectrometry*, 24(2), 301–304. https://doi.org/10.1007/s13361-012-0516-6

3. Chambers, M. C. *et al.* (2012). A cross-platform toolkit for mass spectrometry and proteomics. *Nature Biotechnology*, 30(10), 918–920. https://doi.org/10.1038/nbt.2377

4. Perez-Riverol, Y. *et al.* (2022). The PRIDE database resources in 2022: a hub for mass spectrometry-based proteomics evidences. *Nucleic Acids Research*, 50(D1), D543–D552. https://doi.org/10.1093/nar/gkab1038

5. Wang, M. *et al.* (2020). Sharing and community curation of mass spectrometry data with Global Natural Products Social Molecular Networking. *Nature Biotechnology*, 34(8), 828–837. https://doi.org/10.1038/nbt.3597

---

