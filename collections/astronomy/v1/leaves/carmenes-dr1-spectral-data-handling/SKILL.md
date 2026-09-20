---
name: carmenes-dr1-spectral-data-handling
description: Use when reduced CARMENES VIS/NIR FITS spectra (e.g. car*vis_A.fits from
  the DR1 GTO sample) must be converted into per-spectrum radial velocities and activity-indicator
  time series via cross-correlation with a weighted binary mask.
when_to_use_negative:
- 'The DR1 high-level products already exist: 18,642 precise RVs for 345 targets were
  published in the release, so re-deriving RVs is only needed for custom re-analysis.'
- Input spectra are not reduced FITS stellar spectra with known absolute path naming;
  raccoonccf requires reduced spectra paths and an instrument ID (CARM_VIS, CARM_NIR,
  HARPS, HARPN, EXPRES).
- No suitable mask exists for the target's spectral type and vsini, and no template
  spectrum is available to build one with raccoonmask.
tools:
- name: raccoon
  role: Open-source python package implementing the CCF method (mask building, CCF
    computation, RV/activity-indicator derivation)
  repo: github.com/mlafarga/raccoon
- name: raccoonccf
  role: Entry-point script to compute the CCF of each observed spectrum against the
    mask
  repo: github.com/mlafarga/raccoon
- name: raccoonmask
  role: Entry-point script to build weighted binary masks from a stellar spectrum
    template
  repo: github.com/mlafarga/raccoon
- name: CARMENES
  role: Instrument whose VIS/NIR spectra (CARM_VIS, CARM_NIR) are the target inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:b294553d15381cbf8efad6510f20051904582aa426bfc95b9121da6580784921
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  - raccoon
  - raccoonccf
  - raccoonmask
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# carmenes-dr1-spectral-data-handling

## Summary

Derives radial velocities (RVs) and activity indicators from CARMENES DR1 stellar spectra using the raccoon CCF method: build weighted binary masks from a stellar spectrum template, cross-correlate spectra with the mask, and fit the CCF.

## When to use

Use when reduced CARMENES VIS/NIR FITS spectra (e.g. car*vis_A.fits from the DR1 GTO sample) must be converted into per-spectrum radial velocities and activity-indicator time series via cross-correlation with a weighted binary mask.

## When NOT to use

- The DR1 high-level products already exist: 18,642 precise RVs for 345 targets were published in the release, so re-deriving RVs is only needed for custom re-analysis.
- Input spectra are not reduced FITS stellar spectra with known absolute path naming; raccoonccf requires reduced spectra paths and an instrument ID (CARM_VIS, CARM_NIR, HARPS, HARPN, EXPRES).
- No suitable mask exists for the target's spectral type and vsini, and no template spectrum is available to build one with raccoonmask.

## Inputs

- Reduced CARMENES DR1 FITS spectra (e.g. car*vis_A.fits), as file names or a file listing absolute paths
- Stellar spectrum template FITS (e.g. SERVAL template) for building the mask via raccoonmask
- Weighted binary mask file (.mas) or a published mask ID such as J07274+052default
- Telluric mask (default masks available via --filtell default)

## Outputs

- Per-spectrum CCF profiles from raccoonccf
- Weighted binary mask file (TPL.mas) from raccoonmask
- Time-series table ccf_rv_activity.csv with RV and activity-indicator columns, one row per processed spectrum

## How to apply

Create a conda environment and install raccoon from source (`python -m pip install .`) because its fortran subroutines are compiled with numpy.f2py. First build a weighted binary mask with `raccoonmask` from a stellar spectrum template (e.g. a SERVAL template FITS), controlling line selection with parameters such as `--line_fwhmmin 2.00 --line_fwhmmax 30.00 --line_contrastminmin 0.06 --line_depthw_depthmaxquantile 0.6`, or reuse a published .mas mask matching the target's spectral type (e.g. J07274+052default for M3.5 V). Then compute CCFs with `raccoonccf`, passing the instrument (CARM_VIS or CARM_NIR), the mask, `--filtell default` to flag tellurics, `--rvshift header` to apply barycentric/drift corrections, `--fcorrorders obshighsnr` to avoid SED/extinction biases, and instrument-specific `--ords_use` (orders 10-51 for CARM_VIS, 1-52 subset for CARM_NIR). Fit each CCF to obtain RVs (optionally supplying `--rvabs` to locate the CCF minimum near the known absolute RV) and extract activity indicators from CCF properties. Finally assemble per-spectrum RVs and activity indicators into a time-series table (ccf_rv_activity.csv) for the DR1 sample of 362 targets / 19,633 spectra.

## Related tools

- **raccoon** (Open-source python package implementing the CCF method (mask building, CCF computation, RV/activity-indicator derivation)) — github.com/mlafarga/raccoon
- **raccoonccf** (Entry-point script to compute the CCF of each observed spectrum against the mask) — github.com/mlafarga/raccoon
- **raccoonmask** (Entry-point script to build weighted binary masks from a stellar spectrum template) — github.com/mlafarga/raccoon
- **CARMENES** (Instrument whose VIS/NIR spectra (CARM_VIS, CARM_NIR) are the target inputs)

## Examples

```
raccoonccf PATH/TO/OBS/car*vis_A.fits CARM_VIS J07274+052default --obj OBJ --filtell default --rvshift header --fcorrorders obshighsnr
```

## Evaluation signals

- ccf_rv_activity.csv exists, is a valid CSV, and contains RV and activity-indicator columns
- One row per processed DR1 spectrum, consistent with the DR1 sample of 362 targets / 19,633 spectra
- `raccoonccf -h` displays all options, confirming a correct installation (run outside the source directory)
- Per-target CCF outputs are named with the --obj argument and the CCF minimum lands near the supplied --rvabs absolute RV
- Derived RV counts are comparable to the published 18,642 precise RVs for 345 targets

## Limitations

- raccoon must be installed from source in a conda environment; fortran subroutines are compiled with numpy.f2py and the commented-out pip install path is not documented as working.
- Test `raccoonccf -h` only after leaving the source directory, or the installed script is shadowed.
- Spectra are usually not corrected for barycentric shifts or instrumental drifts, so --rvshift (e.g. header) must be supplied or the CCF/RVs will be biased.
- Available default masks cover only specific spectral types (M0.0 V to M7.0 V) with vsini <= 2.0 km/s (one mask at 2.9 km/s), so very fast rotators or out-of-range SpT lack ready-made masks.

## Evidence

- [intro] raccoon implements CCF in three steps: "`raccoon` is an open source python package the implements the cross-correlation function (CCF) method."
- [readme] mask building and CCF computation capabilities: "- build weighted binary masks from a stellar spectrum template
- compute the CCF of stellar spectra with a mask"
- [readme] installation via compiled fortran subroutines: [evidence span withheld — anchor status: external_doc]
- [readme] barycentric/drift correction requirement: [evidence span withheld — anchor status: external_doc]
- [abstract] DR1 sample size: "spanning from 2016 to 2020, during which 19 633 spectra for a sample of 362 targets were collected"
- [abstract] published high-level RV products: "high-level data products, including 18 642 precise RVs for 345 targets, were derived"
