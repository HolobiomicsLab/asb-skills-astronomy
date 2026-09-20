---
name: zinc-k-edge-chik-extraction
description: Use when raw Zn K-edge (or analogous edge) XAS absorption data — mu vs
  energy from an ASCII file or Athena project — must be converted to chi(k) via autobk
  background subtraction before Fourier transforms or Feff-path fitting.
when_to_use_negative:
- Data are already processed chi(k) or chi(R) spectra (e.g., from an Athena export)
  — re-running autobk would double-subtract the background
- The goal is XANES edge fitting, linear combination, or PCA on normalized mu(E) rather
  than EXAFS chi(k) extraction
- The spectrum is an XRF map or XRD image, which Larch handles with other modules,
  not autobk
tools:
- name: Larch (xraylarch)
  role: Provides read_ascii/read_athena for loading the Zn K-edge data and autobk
    for background subtraction to extract chi(k)
  repo: https://github.com/xraypy/xraylarch
- name: Python
  role: Scripting environment in which the Larch XAFS functions (autobk, xftf) are
    imported and chained
  repo: https://github.com/xraypy/xraylarch
- name: ifeffit
  role: Predecessor package whose background-subtraction/Fourier algorithms Larch's
    autobk pipeline preserves with few changes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:c9312c6f5508ce3e778d6ce5273f1d3ade32f842b2565a22878defc17be4e045
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Larch (xraylarch)
  - Python
  - ifeffit
  - larch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# zinc-k-edge-chik-extraction

## Summary

Extracts the k-space EXAFS signal chi(k) from raw Zn K-edge absorption data (mu vs energy) using Larch's autobk background-subtraction routine, after loading with read_ascii or read_athena. This chi(k) is the required input for downstream Feff path-sum fitting (ff2chi/feffit) of spectra such as cubic ZnSe.

## When to use

Use when raw Zn K-edge (or analogous edge) XAS absorption data — mu vs energy from an ASCII file or Athena project — must be converted to chi(k) via autobk background subtraction before Fourier transforms or Feff-path fitting.

## When NOT to use

- Data are already processed chi(k) or chi(R) spectra (e.g., from an Athena export) — re-running autobk would double-subtract the background
- The goal is XANES edge fitting, linear combination, or PCA on normalized mu(E) rather than EXAFS chi(k) extraction
- The spectrum is an XRF map or XRD image, which Larch handles with other modules, not autobk

## Inputs

- Raw Zn K-edge XAS data file (ASCII mu vs energy, e.g. read via read_ascii)
- Athena project file (.prj) containing XAFS groups, read via read_athena
- autobk parameters such as rbkg=1.0 and optional clamp_hi/clamp_lo settings

## Outputs

- Larch data group containing E0, edge_step, and the background-subtracted chi(k) array
- chi(k) suitable for xftf Fourier transforms or feffit path-sum fitting

## How to apply

Load the Zn K-edge XAFS data (e.g., the ZnSe spectrum) with Larch's read_ascii for ASCII columns or read_athena for .prj Athena projects, then run pre_edge/autobk to determine E0 and edge step and to remove the smooth background, yielding chi(k) stored in the data group. Use autobk with a physically motivated rbkg (e.g., rbkg=1.0, as in the article's autobk example) and check the sensitivity of the resulting chi(k) to autobk parameters such as clamp_hi (values 0, 50, and 200 were tested in the article). The extracted chi(k) is then ready for Fourier transforms (xftf with kmin/kmax/window settings) or for feffit fitting over kmin=3, kmax=12, kweight=2 with a Kaiser window (dk=4) and R-range 1.0–3.0 Å. Rationale: Larch deliberately preserves the core IFEFFIT XAFS procedures (normalization, background subtraction, Fourier transforms) with few algorithmic changes, while adding Python-based scripting for reproducible multi-step processing.

## Related tools

- **Larch (xraylarch)** (Provides read_ascii/read_athena for loading the Zn K-edge data and autobk for background subtraction to extract chi(k)) — https://github.com/xraypy/xraylarch
- **Python** (Scripting environment in which the Larch XAFS functions (autobk, xftf) are imported and chained) — https://github.com/xraypy/xraylarch
- **ifeffit** (Predecessor package whose background-subtraction/Fourier algorithms Larch's autobk pipeline preserves with few changes)

## Examples

```
from larch.xafs import autobk; autobk(dat.energy, dat.mu, rbkg=1.0, group=dat)
```

## Evaluation signals

- The data group contains a valid E0 and edge_step after pre_edge/autobk (cf. the article's Cu foil check: e0=8980.5 eV, edge_step=2.729)
- chi(k) is defined over the fitting k-range (kmin=3 to kmax=12 Å^-1, kweight=2) with no residual background oscillation at low k
- autobk result is stable under clamp_hi sensitivity variation (0, 50, 200) and consistent rbkg (1.0)
- Downstream feffit over R-range 1.0–3.0 Å reaches a low R-factor (ZnSe example: R-factor ≈ 0.00283, chi-square ≈ 246.4), indicating the extracted chi(k) supports a good path-sum fit

## Limitations

- autobk background removal is parameter-sensitive: the article tested clamp_hi values of 0, 50, and 200, so clamp and rbkg choices must be justified per dataset
- FT window parameters (window type, dk) alter the k-weighting/apodization of chi(k); the article compared hanning, parzen, welch, kaiser, and gaussian windows with dk=4
- The article notes a changelog fix for 'turning CIF structures into Feff inputs and running Feff', so upstream Feff path generation may need a recent xraylarch version

## Evidence

- [results] Loading and autobk import: [evidence span withheld — anchor status: not_found]
