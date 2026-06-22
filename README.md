# CYP_Phenoconversion_AoU

Analysis code for a study of **Pharmacotherapy context shapes CYP2D6, CYP2C19 and CYP2B6 effects on antidepressant switching** in the
*All of Us* Research Program. The pipeline classifies CYP2C19, CYP2D6, and CYP2B6 metabolizer
status, adjusts those classifications for the effect of co-prescribed enzyme inhibitors and
inducers (phenoconversion), and tests whether the resulting phenoconverted phenotypes are
associated with antidepressant switching.

> The central question is whether accounting for drug-induced phenoconversion — rather than
> relying on genotype-predicted metabolizer status alone — recovers drug-specific associations
> with antidepressant switching that genotype-only classification misses.

---

## Overview

The analysis is built entirely in **R** inside **Jupyter notebooks**, run on the
**All of Us Researcher Workbench** (Controlled Tier dataset v8). Data are queried from
BigQuery via `bigrquery` and staged through the workspace Google Cloud Storage bucket.

At a high level the pipeline:

1. Extracts demographic, genomic, and drug-exposure data from the *All of Us* Controlled Tier.
2. Cleans antidepressant prescription records and builds treatment episodes, applying inclusion
   criteria and defining outcomes (e.g. continuation, discontinuation, switching) with handling
   for monotherapy versus combination therapy and short cross-tapers.
3. Identifies co-prescribed CYP modulators (inhibitors/inducers) and applies a phenoconversion
   algorithm that adjusts genotype-based metabolizer phenotypes across multiple temporal windows.
4. Runs descriptive and regression analyses linking phenoconverted CYP phenotypes to
   antidepressant switching, both per index antidepressant and pooled, with ancestry
   stratification and negative-control checks.

---

## Repository structure

```
CYP_Phenoconversion_AoU/
├── README.md
└── scripts/
    ├── Extract_Data/
    │   ├── Extract_Data.ipynb              # Query drug, demographic & area-level data from BigQuery
    │   └── Extract_Phenotypes.ipynb        # Extract person info (DOB, sex) & CYP metabolizer phenotypes
    ├── Create_Prescription_Phenotypes/
    │   ├── Prescription_Cleaning.ipynb     # Read & clean raw drug-exposure records
    │   ├── Prescription_Episodes.ipynb     # Construct prescription episodes
    │   └── CYP_Treatment_Phenotypes.ipynb  # Inclusion criteria, episode duration, switching outcomes
    ├── CYP_Phenoconversion_Adjustment/
    │   ├── CYP_Extract_Modulation.ipynb    # Identify co-prescribed CYP inhibitors/inducers
    │   └── CYP_Phenoconversion.ipynb       # Apply phenoconversion to CYP2C19/CYP2D6/CYP2B6
    └── Statistical_Analyses/
        ├── CYP_Descriptive_Analyses.ipynb        # Cohort descriptives & flow/alluvial plots
        ├── CYP_Index_Antidepressant_Switching.ipynb  # Drug-specific switching models
        └── CYP_Pooled_Switching.ipynb            # Pooled switching analyses
```

---

## Pipeline

The notebooks are intended to be run in the order below. Intermediate outputs are written to,
and read back from, the workspace bucket, so each stage depends on the previous one.

### 1. Extract data — `scripts/Extract_Data/`
- **`Extract_Data.ipynb`** — pulls drug-exposure and socioeconomic data for the study
  cohort from the *All of Us* Controlled Tier (v8) via BigQuery.
- **`Extract_Phenotypes.ipynb`** — extracts basic person-level information (date of birth, sex at
  birth) and CYP metabolizer phenotypes.

### 2. Build prescription & treatment phenotypes — `scripts/Create_Prescription_Phenotypes/`
- **`Prescription_Cleaning.ipynb`** — reads and cleans the raw drug records.
- **`Prescription_Episodes.ipynb`** — assembles cleaned prescriptions into continuous episodes.
- **`CYP_Treatment_Phenotypes.ipynb`** — applies study inclusion criteria (e.g. at least two
  antidepressant dispenses, minimum age at first dispense, study-period bounds), computes episode
  duration, classifies episodes as continuation or discontinuation, distinguishes monotherapy from
  combination therapy while allowing short start/end cross-tapers, and derives the switching
  outcomes used downstream.

### 3. Phenoconversion adjustment — `scripts/CYP_Phenoconversion_Adjustment/`
- **`CYP_Extract_Modulation.ipynb`** — identifies co-prescribed CYP2C19, CYP2D6, and CYP2B6
  inhibitors and inducers.
- **`CYP_Phenoconversion.ipynb`** — adjusts genotype-based metabolizer phenotypes to
  *phenoconverted* phenotypes based on concurrent modulator exposure, applied separately for each
  enzyme and across temporal exposure windows.

### 4. Statistical analyses — `scripts/Statistical_Analyses/`
- **`CYP_Descriptive_Analyses.ipynb`** — cohort descriptives and visualisations (including
  alluvial/flow diagrams).
- **`CYP_Index_Antidepressant_Switching.ipynb`** — models switching versus non-switching for each
  index antidepressant with sufficient sample size, stratified by genetically inferred ancestry,
  including negative-control medications to assess specificity.
- **`CYP_Pooled_Switching.ipynb`** — pooled switching analyses across antidepressants.

---

## Environment & requirements

This code was developed and run on the **All of Us Researcher Workbench** using an R kernel.
Reproducing it requires approved Controlled Tier access to *All of Us* and a configured
workspace; the notebooks reference workspace environment variables
(`WORKSPACE_BUCKET`, `GOOGLE_PROJECT`, `OWNER_EMAIL`) and `gsutil` for moving files between the
persistent disk and the workspace bucket.

---

## Data availability

Individual-level *All of Us* data cannot be redistributed. Access is available to approved
researchers through the [*All of Us* Researcher Workbench](https://www.researchallofus.org/).
This repository contains analysis code only; no participant data are included.

---

## Citation

If you use or adapt this code, please cite the preprint: 
Alicia Walker, Naomi Wray, Frank Klont et al. Pharmacotherapy context shapes CYP2C19, CYP2D6 and CYP2B6 effects on antidepressant switching, 10 June 2026, PREPRINT (Version 1) available at Research Square [https://doi.org/10.21203/rs.3.rs-9928103/v1]


---

