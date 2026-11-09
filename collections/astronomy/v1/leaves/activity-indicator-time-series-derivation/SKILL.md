---
name: activity-indicator-time-series-derivation
description: Use when you have reduced FITS echelle spectra (e.g. CARMENES VIS/NIR,
  HARPS, HARPN, EXPRES) and need per-observation radial velocities plus activity indicators
  assembled into a time-series table via the CCF method.
when_to_use_negative:
- Input is already an RV/activity time-series table rather than raw reduced spectra
  — re-derivation with the CCF is unnecessary
- Spectra are low-resolution or single-order data where line-list masking and CCF
  fitting are not applicable
- You need template-free RVs (e.g. SERVAL-style forward modelling) instead of the
  mask-based CCF method
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: raccoon
  role: 'Open-source Python package implementing the CCF method: builds masks, computes
    CCFs, and derives RVs and activity indicators'
  repo: https://github.com/mlafarga/raccoon
- name: raccoonccf
  role: Entry-point script that cross-correlates each observed spectrum with the mask
    to produce the CCF used for RV/activity extraction
  repo: https://github.com/mlafarga/raccoon
- name: raccoonmask
  role: Entry-point script that builds weighted binary masks from a stellar spectrum
    template
  repo: https://github.com/mlafarga/raccoon
- name: CARMENES
  role: 'Source instrument of the VIS/NIR echelle spectra (CARMENES DR1: 19,633 spectra
    of 362 targets) processed with this pipeline'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:1c44de690b8c71a7c71aba4b4c5b30dbe3365d882ebb42dce9ed9be1e231969f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mlafarga/raccoon
  tools:
  - CARMENES
  - Python
  - raccoon
  - raccoonccf
  - raccoonmask
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# activity-indicator-time-series-derivation

## Summary

Derives radial velocities and activity-indicator time series from echelle stellar spectra by cross-correlating each spectrum with a weighted binary mask using the raccoon Python package, as applied to CARMENES DR1 data.

## When to use

Use when you have reduced FITS echelle spectra (e.g. CARMENES VIS/NIR, HARPS, HARPN, EXPRES) and need per-observation radial velocities plus activity indicators assembled into a time-series table via the CCF method.

## When NOT to use

- Input is already an RV/activity time-series table rather than raw reduced spectra — re-derivation with the CCF is unnecessary
- Spectra are low-resolution or single-order data where line-list masking and CCF fitting are not applicable
- You need template-free RVs (e.g. SERVAL-style forward modelling) instead of the mask-based CCF method

## Inputs

- Reduced FITS echelle spectra (e.g. CARMENES VIS car*vis_A.fits files or a text list of file paths)
- Stellar spectrum template (e.g. SERVAL template FITS) for mask construction, or a pre-built .mas weighted binary mask
- Instrument identifier (CARM_VIS, CARM_NIR, HARPS, HARPN, EXPRES)
- Optional: telluric mask file (--filtell default), target absolute RV in km/s, list of spectral orders

## Outputs

- CCF per observed spectrum (from raccoonccf, e.g. spec_ccf.dat / OBJ.ccfpar.dat outputs)
- Fitted radial velocities and activity indicators per spectrum
- Time-series table ccf_rv_activity.csv with RV and activity-indicator columns, one row per processed spectrum

## How to apply

First build a weighted binary mask from a stellar spectrum template with the raccoonmask entry point (e.g. --cont poly, --contfiltmax 400, --line_fwhmmin 2.00 --line_fwhmmax 30.00, --line_contrastminmin 0.06, --line_depthw_depthmaxquantile 0.6), or reuse a shipped mask matched to the target spectral type (M0.0 V through M7.0 V, vsini <= 2.0 km/s). Then run raccoonccf with the mandatory arguments fil_or_list_spec (list or file of reduced FITS spectra), instrument id (CARM_VIS, CARM_NIR, HARPS, HARPN, or EXPRES), and the mask id or path; for CARMENES VIS use the recommended orders 10-51, --filtell default for tellurics, --rvshift header for barycentric/drift corrections, and --fcorrorders obshighsnr to use one SED for all observations and avoid CCF-profile biases from differential extinction. Optionally pass --rvabs with the target's absolute RV (e.g. 26.66 km/s for J07446+035) to locate the CCF minimum faster. Finally fit each spectrum's CCF to extract RVs and activity indicators and assemble them into a per-spectrum time-series table (ccf_rv_activity.csv).

## Related tools

- **raccoon** (Open-source Python package implementing the CCF method: builds masks, computes CCFs, and derives RVs and activity indicators) — https://github.com/mlafarga/raccoon
- **raccoonccf** (Entry-point script that cross-correlates each observed spectrum with the mask to produce the CCF used for RV/activity extraction) — https://github.com/mlafarga/raccoon
- **raccoonmask** (Entry-point script that builds weighted binary masks from a stellar spectrum template) — https://github.com/mlafarga/raccoon
- **CARMENES** (Source instrument of the VIS/NIR echelle spectra (CARMENES DR1: 19,633 spectra of 362 targets) processed with this pipeline)

## Examples

```
raccoonccf PATH/TO/OBS/car*vis_A.fits CARM_VIS J07274+052default --obj OBJ --filtell default --rvshift header --fcorrorders obshighsnr --ords_use 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 --plot_sv --verbose
```

## Evaluation signals

- ccf_rv_activity.csv exists, parses as valid CSV, and contains both RV and activity-indicator columns
- One row per processed input spectrum, consistent with the DR1 sample scale (362 targets / 19,633 spectra; 18,642 precise RVs for 345 targets)
- raccoonccf -h runs successfully after installation, confirming the compiled Fortran/numpy.f2py subroutines and entry points are in place
- CCF minimum lands near the expected absolute RV when --rvabs is supplied (e.g. ~26.66 km/s for J07446+035)
- Using --fcorrorders obshighsnr yields CCF profiles free of biases from differing extinction between observations

## Limitations

- raccoon is mainly Python 3 but relies on Fortran subroutines compiled with numpy.f2py, so installation requires a suitable compiler environment (conda env with pip)
- Spectra are usually not corrected for barycentric shifts or instrumental drifts; --rvshift must be supplied to obtain these corrections
- Spectral-order selection matters: only specific orders are recommended per instrument (e.g. 10-51 for CARM_VIS, a sparse list for CARM_NIR)
- Differing extinction/SED between observations can bias the CCF profile unless --fcorrorders obshighsnr is used

## Evidence

- [readme] It is able to build weighted binary masks from a stellar spectrum template, compute the CCF of stellar spectra with a mask, derive radial velocities (RVs) and activity indicators from the CCF: "`raccoon` is an open source python package the implements the cross-correlation function (CCF) method."
- [readme] raccoonccf requires fil_or_list_spec (reduced FITS spectra), inst (instrument id), and filmask (mask id or file), with optional --rvabs, --filtell default, --rvshift, --fcorrorders obshighsnr and instrument-specific order lists: [evidence span withheld — anchor status: external_doc]
- [readme] Use the same SED for all observations. Avoids biases in the CCF profile due to e.g. different extinction values: [evidence span withheld — anchor status: external_doc]
- [abstract] DR1 delivered 19,633 spectra for 362 targets and 18,642 precise RVs for 345 targets as high-level data products: "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
- [readme] raccoon is mainly Python 3 with Fortran subroutines compiled via numpy.f2py; install from source with pip and test with raccoonccf -h: [evidence span withheld — anchor status: external_doc]
