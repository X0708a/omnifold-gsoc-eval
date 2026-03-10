# OmniFold HDF5 Gap Analysis

## Main Finding: Weight Information Is Highly Asymmetric Across Files

The critical gap is not observable coverage, but **weight coverage**.

| File | Events | Approx. weight columns | What is available |
|---|---:|---:|---|
| `multifold.h5` | 418,014 | ~175 | `weights_nominal`, `weights_ensemble_*`, `weights_bootstrap_mc_*`, `weights_bootstrap_data_*`, and many detector/theory/background/luminosity weights. |
| `multifold_sherpa.h5` | 326,430 | ~27 | `weight_mc`, `weights_nominal`, `weights_bootstrap_mc_*`; missing ensemble and data-bootstrap families. |
| `multifold_nonDY.h5` | 433,397 | 2 | `weight_mc`, `weights_nominal` only. |

## Why This Matters

Uncertainty estimation in OmniFold commonly depends on replica-style weights (ensemble + bootstrap families).

- `multifold.h5` supports full replica-based uncertainty propagation.
- `multifold_sherpa.h5` supports only partial uncertainty propagation.
- `multifold_nonDY.h5` does not support replica-based uncertainty estimation by itself.

So users working only with Sherpa/nonDY cannot reproduce the full uncertainty treatment available from `multifold.h5`.

## Observable Coverage (Not the Main Gap)

The core 24 observables are aligned across files (`pT_ll`, lepton kinematics, track-jet features, `Ntracks_trackj1`, `Ntracks_trackj2`).
This means cross-file friction is driven primarily by missing weight families, not by missing physics features.

## `target_dd` Interpretation

`target_dd` appears in `multifold.h5` and is absent from the reduced files.

Most likely it is a **data-driven training target/label** used in OmniFold-related reweighting steps (consistent with the presence of `weights_dd`).
Its semantics are not documented in-file, so publication metadata should explicitly define:

- meaning and valid range,
- whether it is training-only or intended for downstream analysis,
- how it should be used alongside `weights_dd`.

## Practical Guidance

Use `multifold.h5` as the reference input for publication-level uncertainty propagation.
Treat `multifold_sherpa.h5` and `multifold_nonDY.h5` as reduced/specialized inputs unless auxiliary files provide missing weight families.
