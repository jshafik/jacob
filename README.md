# Wyss-Coray Rotarod Analysis

Reproducible R workflows for the C57BL/6J body-weight, Rotarod, longitudinal
performance-group, and planned proteomics analyses.

This repository contains code and input specifications only. Mouse-level data,
mass-spectrometry data, sample identifiers, and generated results are excluded
from Git.

## Analysis overview

The behavioral workflow:

1. reads exact mouse-by-timepoint matches of Rotarod average latency and body
   weight;
2. calculates body-weight and Rotarod deviations within each age;
3. reports within-age Spearman correlations with Benjamini-Hochberg correction;
4. fits a mixed-effects ANCOVA using raw Rotarod seconds, body weight, age, sex,
   and a random intercept for mouse;
5. compares raw seconds, seconds per gram, and model-adjusted residual scores;
6. classifies Low, Medium, and Top performers from age-specific quartiles; and
7. tests longitudinal group persistence with kappa, ordinal regression, exact
   tests, and age-stratified permutation tests.

Raw latency is the primary outcome. Seconds per gram is retained as a requested
sensitivity analysis. The regression-adjusted residual is the preferred
secondary weight-adjusted measure because simple ratio correction can create
mathematical coupling with body weight.

The proteomics workflow:

- reads UniProt accession from mass-spec column B and protein name from column P;
- removes missing annotations and common contaminant/reverse/site-only flags;
- treats nonpositive, blank, and nonnumeric abundance values as missing;
- log2 transforms and median-centers each sample;
- joins samples to mouse ID, age, and interval/cohort through an explicit map;
- excludes groups with fewer than four samples;
- requires at least 50% valid observations per protein within each tested group;
- calculates Spearman correlations for raw, seconds-per-gram, and adjusted
  performance; and
- applies Benjamini-Hochberg correction within group and phenotype before
  producing a clustered correlation matrix and heatmap.

## Requirements

- R 4.2 or newer
- R packages: nlme and MASS
- R package readxl only when the mass-spec input is Excel

Install missing packages:

    install.packages(c("nlme", "MASS", "readxl"))

## Reproduce the behavioral analysis

1. Copy the template:

       copy config\config.example.R config\config.R

   On macOS or Linux:

       cp config/config.example.R config/config.R

2. Put the two derived inputs described in data/README.md in a private local
   directory. The default is data/derived.

3. Review paths and analysis settings in config/config.R.

4. From the repository root, run:

       Rscript R/01_rotarod_analysis.R config/config.R

5. Review the CSV and text outputs described in results/README.md.

The random seed and permutation counts are recorded in the configuration file,
so reruns are deterministic apart from platform-level numerical differences.

## Run the proteomics analysis

1. Obtain the mass-spec workbook and verify that column B is UniProt accession
   and column P is protein name.
2. Copy config/proteomics_sample_map_template.csv to
   config/proteomics_sample_map.csv.
3. Enter one row for every abundance column. A 6-month sample in the 3-to-6
   cohort and a 6-month sample in the 6-to-9 cohort remain separate through the
   Cohort_or_interval field.
4. Set proteomics paths in config/config.R.
5. Run:

       Rscript R/02_proteomics_pipeline.R config/config.R

The pipeline deliberately does not impute missing protein values. It records
dropout through N and Missing_fraction, then filters by the configured minimum
valid fraction.

## Repository structure

    R/
      01_rotarod_analysis.R
      02_proteomics_pipeline.R
    config/
      config.example.R
      proteomics_sample_map_template.csv
    data/
      README.md
    results/
      README.md

## Interpretation cautions

- Age-specific 30-month estimates are descriptive because only four matched
  observations are available.
- Five of twelve 3-month observations reached the 300-second ceiling.
- Performance groups are relative age-specific quartiles, not clinical states.
- "Improved" and "declined" refer to group movement and do not prove biological
  improvement or decline.
- Protein correlations are exploratory and require independent validation.

## Data governance

Do not commit raw or derived mouse-level data. The .gitignore excludes all
contents under data, generated results, the working configuration, and the
completed sample map while retaining the documentation and templates.
