---
name: healpix-map-binning
description: Use when a scanned timestream with per-sample sky coordinates (e.g. TIME/PHI/THETA/PSI
  columns in an HDF5 file) must be projected onto a HEALPix Nside 256/512 map, either
  as a plain-binned baseline or as the binning operator for destriping.
when_to_use_negative:
- 'The systematic effect is gain variation or spin-synchronous contamination: destriping/binning
  alone can be ineffective or even harmful, and such effects must be handled before
  map-making.'
- The scanning strategy lacks frequent crossing points between baselines, so baseline
  offsets cannot be constrained by pixel scatter.
- The goal is to remove high-frequency (white) noise rather than low-frequency correlated
  noise; binning/destriping acts as a high-pass filter and does not whiten pixel noise.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: dst
  role: Python parallel destriper that applies the HEALPix binning operator and solves
    for baselines; run as 'python dst.py ch6_256.cfg'
  repo: http://github.com/zonca/dst
- name: HEALPix
  role: Pixelization scheme used for the binned map grid (Nside 256 in ch6_256.cfg;
    Nside 512 in the chi2 validation) and for anafast spectra
- name: HDF5
  role: Storage format of the input timestream (TIME, PHI, THETA, PSI, FLAG, TEMP,
    Q, U) read before binning
- name: MPI
  role: 'Parallel binning: data are binned locally on each process, then local maps
    are summed to the global map via Trilinos'
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:a5426f31fda05bc70b62aef8c02445a36471e7f1969e2871cc04607422a8b067
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HEALPix
  - MPI
  - Python
  - Trilinos
  - dst
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# healpix-map-binning

## Summary

Binning polarimeter timestreams (Q/U samples with pointing PHI/THETA) onto a HEALPix sky map, which serves both as the map-making baseline (plain binning) and as the binning operator B inside the destriping equation of the dst destriper.

## When to use

Use when a scanned timestream with per-sample sky coordinates (e.g. TIME/PHI/THETA/PSI columns in an HDF5 file) must be projected onto a HEALPix Nside 256/512 map, either as a plain-binned baseline or as the binning operator for destriping.

## When NOT to use

- The systematic effect is gain variation or spin-synchronous contamination: destriping/binning alone can be ineffective or even harmful, and such effects must be handled before map-making.
- The scanning strategy lacks frequent crossing points between baselines, so baseline offsets cannot be constrained by pixel scatter.
- The goal is to remove high-frequency (white) noise rather than low-frequency correlated noise; binning/destriping acts as a high-pass filter and does not whiten pixel noise.

## Inputs

- Single HDF5 timestream file (e.g. ch6_fill_3d.h5) with columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U
- dst configuration file ch6_256.cfg specifying the HEALPix Nside 256 grid
- Baseline length setting (2000 samples = ~60 s at 33.4 Hz)

## Outputs

- HEALPix binned (or destriped) map of I, Q, U (and hit counts) in HEALPix FITS format
- Residual maps (output minus input signal) for reduced chi2 computation
- chi2_table.csv with reduced chi2 for I/Q x binned/destriped

## How to apply

Build the pointing matrix P mapping each timestream sample to its HEALPix pixel (Nside 256 as in ch6_256.cfg, or Nside 512 for the residual chi2 test) and apply the binning operator B by averaging all samples hitting each pixel. In dst, data are binned locally on each MPI process and the local maps are distributed and summed to a global map via Trilinos. Plain binning of a timestream containing 1/f correlated noise leaves low-frequency residuals (reduced chi2 of residual maps ~1.052/1.026 for I/Q at Nside 512), while binning after baseline subtraction reaches the white-noise level (chi2 ~1.0002/1.0012), so use binning both as the control and inside the destriper. For validation, subtract the input smoothed signal (WMAP 7-year Q band smoothed by 1 degree) from the output map and compute the reduced chi2 of the residual map.

## Related tools

- **dst** (Python parallel destriper that applies the HEALPix binning operator and solves for baselines; run as 'python dst.py ch6_256.cfg') — http://github.com/zonca/dst
- **HEALPix** (Pixelization scheme used for the binned map grid (Nside 256 in ch6_256.cfg; Nside 512 in the chi2 validation) and for anafast spectra)
- **HDF5** (Storage format of the input timestream (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) read before binning)
- **MPI** (Parallel binning: data are binned locally on each process, then local maps are summed to the global map via Trilinos)

## Examples

```
python dst.py ch6_256.cfg  # serial; or: mpirun -np 3 python dst.py ch6_256.cfg
```

## Evaluation signals

- Reduced chi2 of the binned-map residual (output minus 1-degree-smoothed input signal) reproduces the paper's values within a small tolerance: binned I=1.052, Q=1.026 at Nside 512.
- Destriped-map residuals reduce to the white-noise level: reduced chi2 I=1.0002, Q=1.0012, confirming destriping removes correlated noise with minimal impact on the signal.
- For a signal-only timestream, the solved baselines are null to machine precision (max |baseline| < 1e-15 K, ~1e-16), so the binned map is unbiased by spurious offsets.
- Output map files exist as HEALPix FITS with the expected Q and U fields at the configured Nside.
- Local bins from each MPI process sum to a global map whose pixels are uniquely owned (no double counting) when running under mpirun.

## Limitations

- Plain binning leaves low-frequency correlated-noise residuals in the map (chi2 above 1); destriping must be layered on top to reach the white-noise level.
- Systematic effects such as thermally induced gain variations near sunrise/sunset contaminate the map and must be masked out before binning.
- No version numbers are given for HEALPix/Python/Trilinos/HDF5, limiting reproducibility of the exact binning pipeline.

## Evidence

- [intro] The baselines are estimated iteratively by binning the data on a sky map and minimizing the scatter among different measurements of each pixel: [evidence span withheld — anchor status: not_found]
- [intro] pixelized using the HEALPix scheme: [evidence span withheld — anchor status: not_found]
- [methods] parallel binning via Trilinos: [evidence span withheld — anchor status: not_found]
- [results] chi2 validation target values: [evidence span withheld — anchor status: not_found]
