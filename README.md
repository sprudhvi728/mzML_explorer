# mzML_explorer
a web application for inspecting raw mass spectrometry mzML files.
no peptide identification. no database search. just instrument data.




[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-black.svg)](https://www.python.org/)
[![Version](https://img.shields.io/badge/version-1.0.0-black.svg)](CHANGELOG.md)

---

## overview
Mass spectrometry has been a major discovery tool in the life sciences. As an analytical tool, it is used to analyze the molecular composition of biological samples by ionizing analyte molecules and then measuring the m/z ratios of ions. The resulting spectra and metadata are then processed by specialized software packages (DIA-NN, FragPipe) to identify sampled ions. Vendor-specific formats complicate direct inspection and cross-platform access to raw files. To standardize data, the Human Proteome Organization (HUPO) Proteomics Standards Initiative (PSI) established the mzML standard open-source format. 


mzML Explorer a lightweight interface for interactive inspection of raw mass spectrometry data in the mzML open format prior to downstream identification or quantification. It parses any mzML 1.1.0-compliant file — including files converted from Thermo `.raw`, Waters `.raw`/`.wiff`, Bruker `.d`, and other vendor formats via ProteoWizard — and presents the acquisition metadata as a navigable, interactive interface. Users can examine acquisition-level metadata, chromatographic signal, scan structure, precursor information, and individual spectra without submitting the data to a search engine.

The application runs entirely on your local machine. No data ever leaves your computer.

---

## key features

| Feature | Description |
|---------|-------------|
| **Memory-efficient parsing** | Binary m/z/intensity arrays are never loaded during the scan index pass; only scalar metadata is stored |
| **Interactive chromatogram** | Plotly-powered TIC and BPC with zoom, pan, hover, and linked scan navigation |
| **Filterable scan table** | Filter by MS level, retention time range, or scan number; paginated at 200 rows |
| **MS2 precursor details** | Precursor m/z, charge state, and isolation window displayed for each MS2 spectrum |
| **Privacy by design** | All processing is local; no data is transmitted to any external server |

---

## workflow

```
Raw Instrument File                         mzML Explorer
(.raw / .wiff / .d)                               │
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

## requirements

- **Python** 3.10 or newer
- **pip** (bundled with Python)
- **Internet connection** (for Plotly.js CDN)

No R installation is required. All Python dependencies are installed automatically.

---

## installation

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

> **IMPORTANT:** Open `http://127.0.0.1:8000` in your browser. Do **not** open `frontend/index.html` directly — it must be served through the backend for API calls to work.

---

## required input format

mzML Explorer accepts files in the **mzML 1.1.0** open format, maintained by the HUPO Proteomics Standards Initiative (HUPO-PSI). Both indexed and non-indexed mzML files are supported.

Most mass spectrometry data acquisition software produces proprietary binary formats. To convert to mzML, use **ProteoWizard msConvert**, the community standard:

| Vendor format | Instrument family | msConvert input |
|---------------|------------------|-----------------|
| `.raw` (Thermo) | Orbitrap, LTQ, TSQ | `--filter "msLevel 1-2"` |
| `.raw` / `.wiff` (Waters/SCIEX) | Q-TOF, Xevo, TripleTOF | standard conversion |
| `.d` (Bruker) | timsTOF, maXis | standard conversion |
| `.lcd` (Shimadzu) | LCMS series | standard conversion |

**example msConvert command:**

```bash
msconvert sample.raw --mzML --filter "peakPicking true 1-" -o ./output/
```

---

## analysis pipeline
<p align="center">
        <img width="75%" height="75%" alt="mzML-demo" src="https://github.com/user-attachments/assets/6cd4c9b9-ea4f-4839-aa83-eac32c10d086" />
</p>


### step 1 — upload and parse

Click **"Drop an .mzML file here, or click to browse"** and select your file, then click **"Analyze file"**. The backend streams the file to a temporary directory in 4 MB chunks, then makes a single sequential pass through all spectra to build a lightweight scan index. Only scalar metadata (scan number, retention time, MS level, TIC, base peak intensity, precursor m/z) is stored. Raw m/z and intensity arrays are not decoded at this stage.

Parsing a 36 MB Waters LC-MS file containing ~2,000 MS1 scans typically completes in 2–5 seconds.

### step 2 — file summary

After parsing, a summary grid displays:

- **Total scans** — complete acquisition count
- **MS1 / MS2 scans** — breakdown by level
- **RT start / end / range** — acquisition time window in minutes
- **Average scan rate** — scans per minute
- **Max TIC** — largest total ion current observed
- **Max base peak** — largest base peak intensity observed

Click **↓ Download CSV** to save the summary.

### step 3 — chromatogram

The Total Ion Chromatogram (TIC) is plotted immediately after parsing. Toggle to **Base peak** to view the Base Peak Chromatogram (BPC). Both plots support zoom, pan, and per-point hover displaying scan number, retention time, and signal intensity.

### step 4 — scan table

All scans are listed in a paginated, filterable table (200 rows per page). Available filters:

- **MS level** — All, MS1, or MS2
- **RT range** — minimum and maximum retention time in minutes
- **Scan number** — specific scan or range

The **↓ Download CSV** button exports the current filtered view as a flat-file table of scan metadata.

### step 5 — spectrum viewer

Click any row in the scan table to load that scan's spectrum. The backend retrieves only that scan's m/z and intensity arrays using pyteomics-indexed random access — the file is sought directly to the correct byte offset without re-reading the entire file. Peaks are rendered as a vertical stick plot. For MS2 spectra, precursor m/z, charge state, and isolation window boundaries are displayed above the plot.

Click **↓ Download CSV** to save the spectrum as a two-column table (mz, intensity).

---

## downloadable outputs

| Output | Format | Contents |
|--------|--------|----------|
| File summary | CSV | Scan counts, RT range, max TIC, max base peak |
| Scan table | CSV | Per-scan: scan number, RT, MS level, TIC, base peak, precursor m/z, charge |
| Spectrum | CSV | Per-peak: m/z, intensity |

All exports are generated client-side (scan table) or streamed directly from the local FastAPI server (summary and spectrum). No data is sent to any external service.


---

## intended use and scope

mzML Explorer is designed for the **quality-control step that precedes computational proteomics analysis**. It is appropriate for:

- Verifying that an acquisition ran to completion with the expected number of scans
- Confirming that the chromatographic gradient performed as expected
- Inspecting individual spectra for signal quality before submitting to a search engine


mzML Explorer is **not** intended for:

- Peptide or protein identification
- Quantification (label-free, TMT, SILAC, or other)
- Statistical analysis or differential expression

For full proteomics analysis pipelines, consider [FragPipe](https://fragpipe.nesvilab.org/), [MaxQuant](https://www.maxquant.org/), [Proteome Discoverer](https://www.thermofisher.com/), or [Skyline](https://skyline.ms/).

## limitations
- Performance with very large mzML files
- Centroided vs. profile spectra
- Vendor metadata that may not survive conversion
- Whether ion mobility data are supported
- Whether DIA acquisition metadata are fully represented
- Browser/server memory considerations
- Tested mzML variants/instruments

---

## references

1. Martens, L. *et al.* (2011). mzML — a Community Standard for Mass Spectrometry Data. *Molecular & Cellular Proteomics*, 10(1), R110.000133. https://doi.org/10.1074/mcp.R110.000133

2. Goloborodko, A. A., Levitsky, L. I., Ivanov, M. V., & Gorshkov, M. V. (2013). Pyteomics — a Python Framework for Exploratory Data Analysis and Rapid Software Prototyping in Proteomics. *Journal of The American Society for Mass Spectrometry*, 24(2), 301–304. https://doi.org/10.1007/s13361-012-0516-6

3. Chambers, M. C. *et al.* (2012). A cross-platform toolkit for mass spectrometry and proteomics. *Nature Biotechnology*, 30(10), 918–920. https://doi.org/10.1038/nbt.2377

4. Perez-Riverol, Y. *et al.* (2022). The PRIDE database resources in 2022: a hub for mass spectrometry-based proteomics evidences. *Nucleic Acids Research*, 50(D1), D543–D552. https://doi.org/10.1093/nar/gkab1038

5. Wang, M. *et al.* (2020). Sharing and community curation of mass spectrometry data with Global Natural Products Social Molecular Networking. *Nature Biotechnology*, 34(8), 828–837. https://doi.org/10.1038/nbt.3597

---

