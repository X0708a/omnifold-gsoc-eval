# Schema Design for OmniFold Publication Metadata

## Purpose

The metadata schema makes OmniFold outputs reusable by people who did not run the original training pipeline.
It documents file-level capabilities (observables, weight families, event counts) so users can choose correct inputs and uncertainty procedures.

## Core Schema Blocks

- `files`: file path, event count, and available weight families.
- `observables`: canonical observable column names.
- `weights.required_core`: baseline columns for central-value histograms (`weight_mc`, `weights_nominal`).
- `weights.uncertainty_families`: replica-driven uncertainty families.
- `weights.detector_theory_families`: detector/theory/background/luminosity weights.

### Explicit Systematic Weight Families

The schema explicitly declares uncertainty-related families, including:

- `weights_nominal` (central-value reference weight)
- `weights_ensemble_*`
- `weights_bootstrap_mc_*`
- `weights_bootstrap_data_*`

These correspond to uncertainty propagation procedures used in OmniFold analyses:

- ensemble replicas for model/reweighting variation,
- MC bootstrap replicas for MC statistical uncertainty,
- data bootstrap replicas for data statistical uncertainty.

Explicit declaration is important because downstream users can programmatically discover what uncertainty propagation is possible for each file, instead of guessing from column names.

## How a User Who Did Not Run the Analysis Uses the Metadata

Typical workflow:

1. **Load the HDF5 dataset** (for example `data/multifold.h5`) and inspect available columns.
2. **Read `metadata.yaml`** to identify expected observables, event counts, and declared weight families.
3. **Identify observables and the nominal weight** (`weights_nominal`, with `weight_mc` where needed for normalization setup).
4. **Identify systematic/uncertainty variations** from `weights_ensemble_*`, `weights_bootstrap_mc_*`, `weights_bootstrap_data_*`, and detector/theory families.
5. **Compute histograms with correct normalization**, applying the metadata-defined weight strategy consistently across nominal and variation histograms.

This workflow gives reproducible histogram production even for users who only receive files + metadata at publication time.
