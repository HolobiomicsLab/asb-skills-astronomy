---
name: cosmological-table-export
description: Use when you need to reproduce and save a tabulated dark matter halo
  mass function (M vs dn/dlnM) from hmf, e.g. to share HMF values for a given cosmology,
  fitting function and mass range as a portable CSV file.
when_to_use_negative:
- You need a cumulative HMF (n(>M)) rather than the differential dn/dlnM — use the
  ngtm property instead
- You need custom cosmologies, WDM models or non-default redshifts/mass ranges — the
  default MassFunction instantiation covers only PLANCK15, z=0, 10^10-10^15 Msun/h
  with the Tinker (2008) fitting function
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3299
tools:
- name: hmf
  role: Python engine that computes the halo mass function (transfer functions, normalisation
    to sigma_8, fitting function) and exposes dndlnm for tabulation
  repo: https://github.com/halomod/hmf
- name: CAMB
  role: Underlying code used by hmf to compute the transfer functions for the linear
    power spectrum
  repo: https://github.com/joezuntz/pycamb
- name: Python
  role: Implementation language for hmf and for writing the CSV export
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:bf3ee8128d4e789dea834231065d0a5487701b38ee6d15011175c002e02f7c13
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - Python
  - hmf
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cosmological-table-export

## Summary

Export a computed differential halo mass function (dn/dlnM) as a two-column CSV table over a defined mass grid, using the hmf Python package's MassFunction object as the source of the mass values and the dndlnm property.

## When to use

Use when you need to reproduce and save a tabulated dark matter halo mass function (M vs dn/dlnM) from hmf, e.g. to share HMF values for a given cosmology, fitting function and mass range as a portable CSV file.

## When NOT to use

- You need a cumulative HMF (n(>M)) rather than the differential dn/dlnM — use the ngtm property instead
- You need custom cosmologies, WDM models or non-default redshifts/mass ranges — the default MassFunction instantiation covers only PLANCK15, z=0, 10^10-10^15 Msun/h with the Tinker (2008) fitting function

## Inputs

- hmf MassFunction instance (default parameters: Tinker 2008 fitting function, PLANCK15 cosmology, z=0)
- mass grid M in units of Msun/h (default span 10^10 to 10^15 Msun/h)
- differential mass function array dndlnm from the MassFunction object

## Outputs

- CSV file hmf_dndlnm.csv with a header row and two columns: M (Msun/h) and dn/dlnM

## How to apply

Install the hmf Python package (pip install hmf), which wraps CAMB transfer-function evaluation via pycamb, and instantiate the MassFunction class with all default parameters: this gives the Tinker (2008) fitting function between 10^10 and 10^15 Msun/h at z=0 for the PLANCK15 cosmology, with the power spectrum normalised to sigma_8 using a top-hat window function. Extract the mass grid (in Msun/h) and the differential mass function property dndlnm from the instantiated object; these arrays are the two columns of the output table. Write them to a CSV file (hmf_dndlnm.csv) with header rows identifying each column. Optionally cross-check values against HMFcalc online or genmf/MF_Code, since hmf was validated against those codes with excellent consistency.

## Related tools

- **hmf** (Python engine that computes the halo mass function (transfer functions, normalisation to sigma_8, fitting function) and exposes dndlnm for tabulation) — https://github.com/halomod/hmf
- **CAMB** (Underlying code used by hmf to compute the transfer functions for the linear power spectrum) — https://github.com/joezuntz/pycamb
- **Python** (Implementation language for hmf and for writing the CSV export)

## Examples

```
from hmf import MassFunction; mf = MassFunction(); import csv; rows = zip(mf.m, mf.dndlnm); csv.writer(open('hmf_dndlnm.csv','w')).writerows([('M [Msun/h]','dn/dlnM')] + rows)
```

## Evaluation signals

- The output table contains exactly two columns, M and dn/dlnM, with header rows
- The mass grid spans the full default 10^10 to 10^15 Msun/h range
- The power spectrum was normalised to sigma_8 (mass variance on the 8 h^-1 Mpc scale at z=0) using the top-hat window function
- dn/dlnM values fall in the expected range for the PLANCK15 cosmology at z=0, consistent with hmf's validation against genmf, MF_Code and the yt-project HMF calculator
- CDM abundance behaviour holds: halo number density rises with decreasing mass approximately as M^-1.8

## Limitations

- The default instantiation is fixed to PLANCK15 cosmology, z=0, Tinker (2008) fitting function and 10^10-10^15 Msun/h; other settings require explicit parameter updates via the update() method
- If a transfer-function file is supplied instead of computing via CAMB, it must be produced by CAMB or in CAMB format
- Transfer-function interpolation uses linear splines in log-space between k-bounds, which may affect accuracy at the k-range edges unless extrapolate_with_eh=True is used

## Evidence

- [methods] which wraps CAMB transfer-function evaluation via pycamb: "We use the public Code for Anisotropies in the Microwave Background (CAMB) (Lewis et al., 2000) to compute our transfer functions."
- [methods] the differential and cumulative mass functions (dndlnm and ngtm): "the differ- ential and cumulative mass functions (dndlnm and ngtm)"
- [methods] Normalisation to sigma_8: "Ppkq Ð ´ σ 8 ˆ σ 8 ¯ 2 P 0 pkq Ź Normalise"
- [readme] hmf is the python halo mass function calculator backing the export: "``hmf`` is a python application that provides a flexible and simple way to calculate the Halo Mass Function for a range of varying parameters."
- [methods] transfer function file format constraint: "This file must be either produced by CAMB or in CAMB format."
