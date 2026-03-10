# OmniFold Publication Tools

Utilities and metadata for preparing OmniFold HDF5 outputs for publication and downstream reuse.

## Repository Contents

- `gap_analysis.md`: concise gap analysis with emphasis on cross-file weight asymmetry.
- `metadata.yaml`: machine-readable metadata for files, observables, and weight families.
- `schema_design.md`: design rationale and user workflow for metadata-driven analysis.
- `weighted_histogram.py`: weighted histogram computation and plotting utilities.
- `explore_h5.py`: HDF5 schema/preview helper.
- `tests/`: unit tests for weighted histogram utilities.

## Quick Example

Run:

```bash
python3 example_plot.py
```

This will load `data/multifold.h5`, build a weighted histogram for `pT_ll` using `weights_nominal`, and save:

- `example_histogram.png`
