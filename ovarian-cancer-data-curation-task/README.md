# Ovarian Cancer Data Curation Task


[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![Data: GDC TCGA-OV](https://img.shields.io/badge/data-GDC_TCGA--OV-green.svg)](https://portal.gdc.cancer.gov/projects/TCGA-OV)

---

## Project Overview

This repository curates, harmonizes, and integrates open-source ovarian cancer data . It produces a reusable **universal dataset** by merging TCGA-OV clinical and biospecimen data, complete with schema, metadata, visualizations, and reproducible ETL notebooks.

---

## Repository Structure

```
ovarian-cancer-data-curation-task/
│
├── README.md                          ← This file
├── curated_sources.md                 ← Catalog of 15 public OV datasets (Part A)
├── schema.json                        ← Column schema: types, constraints, provenance (Part E)
├── metadata.json                      ← Sources, citations, licenses, processing steps (Part E)
├── requirements.txt                   ← Python dependencies
│
├── notebooks/
│   ├── 01_load_and_audit.ipynb        ← Load raw data, flatten JSON, clean, audit (Part B + C)
│   ├── 02_analysis_and_viz.ipynb      ← Visualizations for both datasets (Part C)
│   └── 03_integration_and_schema.ipynb← Merge, universal dataset, schema export (Part E)
│
├── data/
│   ├── raw/                           ← Raw JSON files (not committed if large — see below)
│   │   ├── clinical_raw.json
│   │   └── biospecimen_raw.json
│   └── processed/
│       ├── clinical_clean.csv         ← Patient-level cleaned clinical data
│       ├── biospecimen_clean.csv      ← Sample-level cleaned biospecimen data
│       ├── universal_dataset.csv      ← Merged universal dataset
│       └── plots/                     ← All generated visualizations (PNG)
│
└── reports/
    └── linkage_report.md              ← Merge methodology, validation, roadmap
```

---

## Data Sources

Two datasets from the [GDC Data Portal](https://portal.gdc.cancer.gov/projects/TCGA-OV) (open tier, no login required):

| File | Modality | Rows (approx) |
|---|---|---|
| `clinical.project-tcga-ov.2026-03-02.json` | Clinical / Tabular | ~600 patients |
| `biospecimen.project-tcga-ov.2026-03-03.json` | Biospecimen metadata | ~600 patients, ~2000 samples |

See [`curated_sources.md`](./curated_sources.md) for the full catalog of 15 public OV datasets.

---

## Quick Start

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/ovarian-cancer-data-curation-task.git
cd ovarian-cancer-data-curation-task
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Download raw data
1. Go to [https://portal.gdc.cancer.gov/projects/TCGA-OV](https://portal.gdc.cancer.gov/projects/TCGA-OV)
2. Click **"Clinical"** → download JSON → save as `data/raw/clinical_raw.json`
3. Click **"Biospecimen"** → download JSON → save as `data/raw/biospecimen_raw.json`

> Raw files are not committed to this repo due to size. Download steps are documented above.

### 4. Run notebooks in order
```bash
jupyter notebook
```
Open and run:
1. `notebooks/01_load_and_audit.ipynb`
2. `notebooks/02_analysis_and_viz.ipynb`
3. `notebooks/03_integration_and_schema.ipynb`

---

## Key Design Decisions

### Deduplication after `explode()`
`follow_ups` and `diagnoses` are nested lists in the GDC JSON. After `explode()`, a single patient can produce multiple rows. We resolve this with `groupby("case_id").last()` — keeping the most complete record per patient.

### Vital Status: Censored ≠ Alive
Patients with missing `vital_status` and missing `days_to_death` are labeled `"Unknown/Censored"` — **not** assumed Alive. In TCGA, missing follow-up often means the patient was lost to follow-up (censored), not that they survived.

### Age Imputation: Median, not Mode
`age_at_index` is a continuous variable. Median is used for imputation instead of mode to avoid creating artificial spikes in the distribution. Outliers outside [18, 100] years are removed before analysis.

### Weight Imputation: Group + Global Fallback
Sample weight is imputed using per-`sample_type` group median. A global median is applied as fallback for groups where all values are missing.

### `analytes` Flattening
The `analytes` column is a nested list of dicts. We extract `analyte_type` from the first element per portion. Multi-analyte portions are noted in the linkage report for future work.

---

## Visualizations Produced

| Plot | File |
|---|---|
| Missing data heatmap (clinical) | `plot_clinical_missing_heatmap.png` |
| Age distribution histogram | `plot_age_distribution.png` |
| FIGO stage bar chart | `plot_figo_stage.png` |
| Vital status pie chart | `plot_vital_status.png` |
| Race distribution (bias check) | `plot_race_distribution.png` |
| Missing data heatmap (biospecimen) | `plot_biospc_missing_heatmap.png` |
| Tumor descriptor bar chart | `plot_tumor_descriptor.png` |
| Sample type bar chart | `plot_sample_type.png` |
| FFPE vs Non-FFPE pie chart | `plot_ffpe.png` |
| Analyte type distribution | `plot_analyte_type.png` |
| FIGO stage vs vital status (stacked) | `plot_stage_vs_vital.png` |
| Sample type by FIGO stage | `plot_sample_by_stage.png` |
| Age by vital status (boxplot) | `plot_age_by_vital.png` |

---

## Known Limitations

- TCGA-OV is **heavily late-stage** (III/IV) — not representative of early ovarian cancer
- **Predominantly white, US-based cohort** — results may not generalize globally
- No **CA-125 / HE4 biomarker** data in GDC exports
- No **imaging** data in this pipeline — TCIA linkage is future work
- `universal_dataset.csv` is **sample-level** — groupby `case_id` for patient-level analysis

---

## License

- **Code:** MIT License
- **Data:** NIH GDC Open Tier — see [GDC Data Access Policies](https://gdc.cancer.gov/access-data/data-access-policies)
- **Documentation:** CC BY 4.0

---

## Citation

If you use this pipeline, please cite the original TCGA-OV data:

> The Cancer Genome Atlas Research Network. (2011). Integrated genomic analyses of ovarian carcinoma. *Nature*, 474, 609–615. https://doi.org/10.1038/nature10166
