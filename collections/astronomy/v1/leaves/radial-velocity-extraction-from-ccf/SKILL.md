---
name: radial-velocity-extraction-from-ccf
description: Use when reduced echelle spectra (e.g. CARMENES VIS/NIR FITS) need precise
  RVs and CCF-based activity-indicator time series, and no pipeline-produced RV product
  is available.
when_to_use_negative:
- The spectra already come with pipeline-derived RVs — the CARMENES DR1 already provides
  high-level products including 18,642 precise RVs for 345 targets
- You need an RV pipeline for an instrument outside the supported list (CARM_VIS,
  CARM_NIR, HARPS, HARPN, EXPRES)
- You need full spectral fitting of line profiles rather than the cross-correlation-function
  approach
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3315
tools:
- name: raccoon
  role: 'Open-source python (with numpy.f2py-compiled fortran subroutines) package
    implementing the CCF method: builds masks, computes CCFs, derives RVs and activity
    indicators'
  repo: github.com/mlafarga/raccoon
- name: raccoonccf
  role: Entry-point script that computes the CCF of stellar spectra with a mask, producing
    the RV/CCF outputs
  repo: github.com/mlafarga/raccoon
- name: raccoonmask
  role: Entry-point script that builds weighted binary masks from a stellar spectrum
    template
  repo: github.com/mlafarga/raccoon
- name: CARMENES
  role: High-accuracy radial-velocity spectrograph at Calar Alto whose VIS/NIR spectra
    are the primary input (CARMENES DR1)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:a11933bc460e2910f8b82875fa4b352f1426e95deb7c983c2e31887526d1b8c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  - raccoon
  - raccoonccf
  - raccoonmask
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# radial-velocity-extraction-from-ccf

## Summary

Derives precise radial velocities (RVs) and activity indicators from stellar spectra using the CCF method implemented in the raccoon python package, as applied to the CARMENES DR1 M-dwarf survey (19,633 spectra, 362 targets).

## When to use

Use when reduced echelle spectra (e.g. CARMENES VIS/NIR FITS) need precise RVs and CCF-based activity-indicator time series, and no pipeline-produced RV product is available.

## When NOT to use

- The spectra already come with pipeline-derived RVs — the CARMENES DR1 already provides high-level products including 18,642 precise RVs for 345 targets
- You need an RV pipeline for an instrument outside the supported list (CARM_VIS, CARM_NIR, HARPS, HARPN, EXPRES)
- You need full spectral fitting of line profiles rather than the cross-correlation-function approach

## Inputs

- Reduced stellar spectra (FITS), e.g. CARMENES DR1 car*vis_A.fits files or a text list of absolute file paths
- Stellar spectrum template (e.g. SERVAL template FITS) for raccoonmask mask building
- Spectral mask file (.mas) or bundled mask ID (e.g. J07274+052default)
- Instrument identifier (CARM_VIS, CARM_NIR, HARPS, HARPN, EXPRES)
- Telluric mask file (or --filtell default)

## Outputs

- CCF for each input spectrum (raccoonccf output files)
- Per-spectrum radial velocities and activity indicators assembled into a time-series table (e.g. ccf_rv_activity.csv)
- Weighted binary mask file (TPL.mas) when raccoonmask is run

## How to apply

Build a weighted binary mask from a stellar spectrum template with the raccoonmask entry point (e.g. for CARMENES VIS templates using --cont poly, --line_fwhmmin 2.00, --line_fwhmmax 30.00, --line_contrastminmin 0.06), or select one of the bundled masks by spectral type (SpT M0.0 V–M7.0 V, vsini <= 2.0 km/s). Then run raccoonccf on the reduced FITS spectra with the mandatory arguments fil_or_list_spec, inst (CARM_VIS, CARM_NIR, HARPS, HARPN, EXPRES), and filmask, adding --filtell default for telluric masking and --rvshift header for barycentric/drift corrections. Restrict to well-behaved orders via --ords_use (orders 10–51 for CARM_VIS; 1–52 subset for CARM_NIR) and use --fcorrorders obshighsnr so all observations share the same SED, avoiding CCF-profile biases from differing extinction. Fit the resulting CCFs to extract RVs and activity indicators, then assemble per-spectrum values into a time-series table. Rationale: the CCF method locates the CCF minimum (accelerated by supplying --rvabs, the target's absolute RV in km/s) and its profile properties carry both Doppler shift and stellar-activity information.

## Related tools

- **raccoon** (Open-source python (with numpy.f2py-compiled fortran subroutines) package implementing the CCF method: builds masks, computes CCFs, derives RVs and activity indicators) — github.com/mlafarga/raccoon
- **raccoonccf** (Entry-point script that computes the CCF of stellar spectra with a mask, producing the RV/CCF outputs) — github.com/mlafarga/raccoon
- **raccoonmask** (Entry-point script that builds weighted binary masks from a stellar spectrum template) — github.com/mlafarga/raccoon
- **CARMENES** (High-accuracy radial-velocity spectrograph at Calar Alto whose VIS/NIR spectra are the primary input (CARMENES DR1))

## Examples

```
raccoonccf PATH/TO/OBS/car*vis_A.fits CARM_VIS J07274+052default --obj OBJ --filtell default --rvshift header --fcorrorders obshighsnr --plot_sv --verbose
```

## Evaluation signals

- ccf_rv_activity.csv exists, is a valid CSV with RV and activity-indicator columns, and has one row per processed DR1 spectrum
- Row counts are consistent with the DR1 sample (19,633 spectra for 362 targets; 18,642 precise RVs for 345 targets)
- raccoonccf -h displays all script options, confirming a successful install (python 3 with numpy.f2py-compiled fortran)
- CCF minimum lands near the target's absolute RV when --rvabs is supplied (e.g. --rvabs 26.66 for J07446+035)
- Derived RV time series reproduce published CARMENES DR1 RV behavior for a known target (e.g. J07446+035)

## Limitations

- Spectra are usually not corrected for barycentric shifts or instrumental drifts, so --rvshift must be specified explicitly (e.g. --rvshift header)
- CCF profiles can be biased by differing SED/extinction across observations; mitigated with --fcorrorders obshighsnr (shared SED from high-SNR observations)
- Default masks are limited to CARMENES VIS/NIR and SpT M0.0 V–M7.0 V with vsini <= 2.0 km/s (one M6.0 V mask at 2.9 km/s)
- Installation from source requires leaving the source directory before testing raccoonccf; fortran subroutines must compile via numpy.f2py

## Evidence

- [intro] raccoon is an open source python package the implements the cross-correlation function (CCF) method.: "`raccoon` is an open source python package the implements the cross-correlation function (CCF) method."
- [methods] Entry points raccoonccf and raccoonmask are listed in the package documentation.: "### Entry points
- `raccoonccf`
- `raccoonmask`"
- [readme] The scripts are the easiest way to use the package.: [evidence span withheld — anchor status: external_doc]
- [readme] raccoonccf requires fil_or_list_spec, inst, and filmask as mandatory arguments.: [evidence span withheld — anchor status: external_doc]
- [readme] raccoon is mainly implemented in python 3 and uses fortran subroutines compiled with numpy.f2py.: [evidence span withheld — anchor status: external_doc]
- [abstract] The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived.: "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
