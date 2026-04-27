# OmniFold Publication Tools

Research-oriented utilities and documentation for preparing OmniFold weight
outputs for publication and downstream reuse.

## Purpose

This repository provides:
- a structured gap analysis of available OmniFold HDF5 files,
- a machine-readable metadata schema (`spec/metadata.yaml`),
- schema design rationale (`docs/schema_design.md`),
- robust weighted-histogram utilities with uncertainty handling,
- tests covering numerical and data-quality edge cases,
- a minimal working prototype of a publication package API.

The goal is to make event-level OmniFold outputs easier to validate, compare,
and reuse by analysts who did not run the original training pipeline.

---

## Repository Structure

| File / Directory | Description |
|---|---|
| `docs/` | Design artifacts such as the gap analysis and schema rationale |
| `spec/` | Metadata schema and histogram utility used by the prototype |
| `scripts/` | Exploration and one-off inspection scripts |
| `omnifold_publication/` | Prototype publication package API (writer, reader, validator) |
| `examples/` | Usage demos, including roundtrip validation and histogram plotting |
| `tests/` | pytest suite for histogram logic, roundtrip correctness, and validation |
| `data/` | Local OmniFold HDF5 files (ignored from git tracking) |

---

## Setup
```bash
python3 -m pip install -r requirements.txt
```

The pinned minimum version constraints in `requirements.txt` improve
reproducibility across reviewer environments.

---

## Run Code

**Weighted histogram example:**
```bash
python3 examples/example_plot.py
```

Reads `data/multifold.h5`, computes a weighted `pT_ll` histogram using
`weights_nominal`, and saves `examples/example_histogram.png`.

**Publication package roundtrip:**
```bash
python3 examples/package_roundtrip.py
```

Reads `data/multifold.h5`, writes a minimal publication package to
`artifacts/demo_nominal/` (Parquet + metadata), reloads it, and verifies
that the reloaded histogram matches the original to numerical precision.

**Reproduce a packaged histogram:**
```bash
python3 examples/reproduce_histogram.py
```

Loads `artifacts/demo_nominal/`, computes a nominal weighted histogram, and
prints a compact closure summary. Run `examples/package_roundtrip.py` first if
the demo package has not been created.

**Command-line inspection:**
```bash
python3 -m omnifold_publication summary artifacts/demo_nominal/
python3 -m omnifold_publication validate artifacts/demo_nominal/
python3 -m omnifold_publication inspect artifacts/demo_nominal/
```

**Proposal-style package API:**
```python
from omnifold_publication import load_package

pkg = load_package("artifacts/demo_nominal/")
df = pkg.load_events(columns=["pT_ll"])
w = pkg.get_weights()
replicas = pkg.list_systematics()
```

---

## Prototype: `omnifold_publication`

The `omnifold_publication` package is a minimal proof-of-concept for the
publication API proposed in the GSoC project.

**Modules:**

| Module | Description |
|---|---|
| `writer.py` | Reads OmniFold HDF5 output and writes a publication package (Parquet + metadata YAML) |
| `reader.py` | Loads metadata and event data from a publication package; supports both function and class-based access |
| `validation.py` | Checks schema compliance, required columns, event count consistency, and file integrity |
| `cli.py` | Provides lightweight `inspect`, `validate`, and `summary` commands |

**Package layout written by `writer.py`:**
```
artifacts/demo_nominal/
    events.parquet       # observables + weights (columnar, partitioned)
    metadata.yaml        # format version, provenance, weight families, normalization
```

**Key design decisions:**
- Parquet is used as the primary storage format: columnar layout enables
  efficient per-observable and per-weight-family access without loading the
  full event record.
- The writer selects a canonical nominal weight (`weights_nominal`) and one
  replica column (`weights_ensemble_0`) to demonstrate the minimal/full
  package split described in the proposal.
- Validation is decoupled from reading, so packages can be checked
  independently of the analysis workflow.

This prototype covers a compact package tier and now includes Phase 2 hooks
for metadata-declared systematics, iteration-aware weights, event alignment
checks, normalization checks, and a small CLI. The remaining full-project scope
is broader HEPData export and richer experiment provenance.

For lightweight proposal snippets, `numpy.histogram` may be used for
readability. For real weighted analyses, uncertainty handling, and plotting,
use [`spec/weighted_histogram.py`](/Users/aashirvad/omnifold-gsoc-eval/spec/weighted_histogram.py).

---

## Run Tests
```bash
pytest
```

The test suite includes:
- core histogram correctness and uncertainty propagation,
- robustness checks for realistic physics-data issues (NaNs, negative weights,
  shape mismatches, empty arrays),
- roundtrip correctness (write package → reload → compare histograms),
- validation failure cases (missing files, missing columns, wrong event counts),
- metadata-driven systematics, iteration weights, normalization, closure, and CLI behavior.
