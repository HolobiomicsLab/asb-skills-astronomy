---
name: absorption-edge-parameter-extraction
description: Use when a raw XAS transmission spectrum (e.g. energy, i0, it columns
  in an ASCII file like fe.060) must be converted to absorbance and normalized around
  an absorption edge before XANES/EXAFS processing.
when_to_use_negative:
- Input is already a normalized mu(E) array or an Athena project file with normalization
  already applied (e.g. fe_athena.prj) — re-running pre_edge would re-fit an already-processed
  spectrum
- The analysis goal is EXAFS fitting to Feff paths or Fourier transforms rather than
  edge normalization — use feffit/xftf workflows instead
- The spectrum is not an absorption-edge measurement (e.g. XRF spectra or XRD images),
  which Larch handles with other tools
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3418
tools:
- name: larch
  role: Provides larch.xafs read_ascii and pre_edge for loading the spectrum and fitting
    edge normalization
  repo: https://github.com/xraypy/xraylarch
- name: Python
  role: Used to compute data.mu = -np.log(data.it/data.i0) and write the CSV output
  repo: https://github.com/xraypy/xraylarch
- name: ifeffit
  role: Reference implementation whose core normalization procedure Larch preserves
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:312647e559889517d481c76b96229e9fe4958df62f349f1b2c32c11134d258ca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Larch (xraylarch)
  - Python
  - ifeffit
  - larch
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# absorption-edge-parameter-extraction

## Summary

Extracts XAFS absorption-edge parameters (e0, edge_step, pre-edge and post-edge polynomial coefficients) and computes normalized mu(E) from a raw absorbance spectrum using Larch's pre_edge routine, preserving the IFEFFIT core normalization procedure.

## When to use

Use when a raw XAS transmission spectrum (e.g. energy, i0, it columns in an ASCII file like fe.060) must be converted to absorbance and normalized around an absorption edge before XANES/EXAFS processing.

## When NOT to use

- Input is already a normalized mu(E) array or an Athena project file with normalization already applied (e.g. fe_athena.prj) — re-running pre_edge would re-fit an already-processed spectrum
- The analysis goal is EXAFS fitting to Feff paths or Fourier transforms rather than edge normalization — use feffit/xftf workflows instead
- The spectrum is not an absorption-edge measurement (e.g. XRF spectra or XRD images), which Larch handles with other tools

## Inputs

- Raw ASCII XAFS data file (e.g. fe.060, Fe foil Fe K-edge) with energy, i0, and it columns
- Edge window parameters: pre1=-150, pre2=-30, norm1=150, norm2=850, nnorm=1

## Outputs

- Edge parameters e0 (eV) and edge_step from the pre_edge output group
- Pre-edge and post-edge polynomial coefficients
- Normalized mu(E) energy/mu columns written to CSV (e.g. pre_edge_fe060.csv)

## How to apply

Load the raw spectrum with larch.xafs read_ascii (e.g. fe.060 from Larch's example data directory) and compute absorbance as data.mu = -np.log(data.it/data.i0). Run larch.xafs pre_edge with the common parameterization pre1=-150, pre2=-30 (relative to e0) and norm1=150, norm2=850 with nnorm=1. Extract e0, edge_step, and the fitted pre-edge/post-edge polynomial coefficients from the output group, then form normalized mu(E) by removing the pre-edge trend and dividing by edge_step. This mirrors the IFEFFIT core normalization that Larch deliberately preserves. Write the edge parameters and normalized energy/mu columns to a CSV and validate that post-edge normalized mu settles near ~1.

## Related tools

- **larch** (Provides larch.xafs read_ascii and pre_edge for loading the spectrum and fitting edge normalization) — https://github.com/xraypy/xraylarch
- **Python** (Used to compute data.mu = -np.log(data.it/data.i0) and write the CSV output) — https://github.com/xraypy/xraylarch
- **ifeffit** (Reference implementation whose core normalization procedure Larch preserves)

## Examples

```
from larch.xafs import read_ascii, pre_edge; data = read_ascii('fe.060'); data.mu = -np.log(data.it/data.i0); pre_edge(data, pre1=-150, pre2=-30, nnorm=1, norm1=150, norm2=850)
```

## Evaluation signals

- Output CSV exists and contains e0 and edge_step fields with finite values
- Normalized mu(E) values settle near ~1 in the post-edge region (norm1=150 to norm2=850 window)
- e0 falls near the expected Fe K-edge region and edge_step is a positive finite jump
- Data file fe.060 was read with read_ascii and mu equals -ln(it/i0) with no negative or non-finite absorbance values
- Results consistent with IFEFFIT normalization (Larch preserves the core XAFS procedures with few algorithmic changes)

## Limitations

- Normalization windows (pre1/pre2, norm1/norm2, nnorm) are spectrum-specific; the -150/-30 and 150/850 eV values are calibrated for Fe-foil-like data and must be adjusted per element/edge
- Larch preserves IFEFFIT's core normalization procedures with few algorithmic changes, so results track IFEFFIT conventions rather than alternative normalization schemes

## Evidence

- [results] data = read_ascii('../xafsdata/fe.060'): "data = read_ascii('../xafsdata/fe.060')"
- [abstract] the initial release of larch preserves the core XAFS analysis procedures such as normalization, background subtraction: "the initial release of larch preserves the core XAFS analysis procedures such as normalization, background subtraction, Fourier transforms, fitting of XANES spectra"
- [results] pre_edge(grp, pre1=-150, pre2=-30, nnorm=1, norm1=150, norm2=850): "pre_edge(grp, pre1=-150, pre2=-30, nnorm=1, norm1=150, norm2=850)"
- [results] Cu foil pre_edge result: e0 = 8980.5 eV, edge_step = 2.729: [evidence span withheld — anchor status: not_found]
- [readme] Larch is an open-source library and set of applications for visualizing, processing, and analyzing X-ray absorption: "Larch is an open-source library and set of applications for visualizing, processing, and analyzing X-ray absorption and fluorescence spectroscopy data from synchrotron beamlines."
