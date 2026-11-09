---
name: dark-matter-halo-abundance-calculation
description: Use when you need to compute dark matter halo abundances (dndlnm) over
  ~10^10-10^16 h^-1 M_sun at a fixed redshift, or quantify how HMF fitting functions
  (PS, ST, Jenkins, Tinker, Warren, Watson, etc.) diverge from Sheth-Tormen.
when_to_use_negative:
- The goal is an exact simulation-calibrated HMF for a specific N-body box rather
  than a comparison of standard fitting-function forms
- Warm dark matter or non-standard dark-energy models outside hmf's supported set
  are required (the paper lists broader DM/DE models as future work)
- Haloes are defined by a finder other than Friends-of-Friends and the Watson FoF-specific
  calibration would be misapplied
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hmf
  role: Python engine that computes the HMF, exposes the MassFunction object and fitting-function
    dictionary, and implements the cascading update() method
  repo: https://github.com/halomod/hmf
- name: HMFcalc
  role: Web frontend to hmf for interactive exploration of cosmology, redshift and
    fitting-function effects on the HMF
- name: CAMB
  role: Computes the transfer functions consumed by hmf for the power-spectrum calculation
  repo: https://github.com/joezuntz/pycamb
- name: Python
  role: Implementation language of hmf/HMFcalc and environment for looping over fits
    and plotting ratios
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:a51f3c234b77c33e3d64c881c04e218169199fe80565078daf45714eb1c923b9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - HMFcalc
  - Python
  - hmf
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dark-matter-halo-abundance-calculation

## Summary

Calculates differential halo mass functions (dndlnm) with the hmf Python package and compares multiple fitting functions f(sigma) against the Sheth-Tormen baseline at z=0, reproducing the HMFcalc Figure 2 comparison.

## When to use

Use when you need to compute dark matter halo abundances (dndlnm) over ~10^10-10^16 h^-1 M_sun at a fixed redshift, or quantify how HMF fitting functions (PS, ST, Jenkins, Tinker, Warren, Watson, etc.) diverge from Sheth-Tormen.

## When NOT to use

- The goal is an exact simulation-calibrated HMF for a specific N-body box rather than a comparison of standard fitting-function forms
- Warm dark matter or non-standard dark-energy models outside hmf's supported set are required (the paper lists broader DM/DE models as future work)
- Haloes are defined by a finder other than Friends-of-Friends and the Watson FoF-specific calibration would be misapplied

## Inputs

- Cosmological parameters (default hmf values) with sigma_8 normalisation
- Mass grid from 10^10 to 10^16 h^-1 M_sun at z=0
- Transfer function computed by CAMB or imported from a CAMB-format file
- Fitting-function string identifiers: ST, PS, Jenkins, Reed03, Warren, Reed07, Tinker, Crocce, Courtin, Bhattacharya, Angulo, Watson FoF, Watson

## Outputs

- dndlnm column per fitting function over the mass grid
- 13 ratio columns of each fitting function's dndlnm divided by the Sheth-Tormen dndlnm
- CSV table of mass grid, dndlnm columns, and ratio columns; optional matplotlib plot of ratios versus mass reproducing Figure 2

## How to apply

Instantiate a MassFunction object from the hmf package with default cosmology, sigma_8 normalisation, and a mass grid spanning 10^10-10^16 h^-1 M_sun at z=0. Evaluate dndlnm for the 'ST' fitting-function identifier as the baseline. Loop over the remaining fits (PS, Jenkins, Reed03, Warren, Reed07, Tinker, Crocce, Courtin, Bhattacharya, Angulo, Watson FoF, Watson), changing only the fitting-function parameter via update() so the cascading dependency tracker recalculates only dndlnm (the cascading method cuts processing time by ~50% when each parameter changes once, and >94% when only redshift changes). Divide each fit's dndlnm by the ST curve per mass bin, write mass, dndlnm and ratio columns to CSV, and optionally plot ratios versus mass to reproduce Figure 2. Transfer functions come from CAMB (or a CAMB-format file), are interpolated with linear splines in log-space, the power spectrum P0(k)=k^n*T^2 is normalised via sigma_8, and the linear growth factor is applied to P(k) and sigma(M,z) before evaluating f(sigma).

## Related tools

- **hmf** (Python engine that computes the HMF, exposes the MassFunction object and fitting-function dictionary, and implements the cascading update() method) — https://github.com/halomod/hmf
- **HMFcalc** (Web frontend to hmf for interactive exploration of cosmology, redshift and fitting-function effects on the HMF)
- **CAMB** (Computes the transfer functions consumed by hmf for the power-spectrum calculation) — https://github.com/joezuntz/pycamb
- **Python** (Implementation language of hmf/HMFcalc and environment for looping over fits and plotting ratios)

## Examples

```
from hmf import MassFunction; mf = MassFunction(Mmin=10, Mmax=16, dlnm=0.01); dndlnm_st = mf.dndlnm; mf.update(hmf_model='Tinker'); ratio = mf.dndlnm / dndlnm_st
```

## Evaluation signals

- Output table contains one dndlnm column per fitting function plus exactly 13 ratio columns
- Ratio values are order-unity across most of the 10^10-10^16 h^-1 M_sun mass range
- Ratio deviations are concentrated at the high-mass end, qualitatively consistent with published Figure 2
- Updating only the fitting-function parameter via update() leaves the mass grid and cosmology unchanged (cascading dependency tracker recalculates only dndlnm)
- dndlnm values reproduce a low-mass slope near M^-1.8 expected for CDM

## Limitations

- Some fitting functions have limited applicable mass ranges, so limits must be placed on each function as appropriate (as in Figure 2)
- pycamb is outdated and no longer works with recent CAMB versions; modern CAMB ships its own Python bindings
- Cascading update savings depend on how many parameters change; benefits shrink if many parameters are varied simultaneously
- hmf is designed for standard CDM-style models; broader dark matter and dark energy models are deferred to future work

## Evidence

- [methods] update() method semantics: [evidence span withheld — anchor status: not_found]
