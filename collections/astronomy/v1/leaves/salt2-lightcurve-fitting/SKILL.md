---
name: salt2-lightcurve-fitting
description: Use when you need per-object SALT2 stretch and colour from multi-band
  SN Ia light curves, or when you must locate the redshift above which survey selection
  effects bias fitted light-curve parameters.
when_to_use_negative:
- The light curves are not SN Ia photometry or do not span enough of the rise and
  decline to constrain SALT2 stretch and colour
- The goal is raw survey/dataset generation (Target, Survey, DataSet construction)
  rather than light-curve parameter fitting
- The sample has already been restricted to a volume- and selection-unbiased regime,
  so a redshift-dependence study would be redundant
tools:
- name: sncosmo
  role: 'SALT2 light-curve fitting: recovers stretch (x1) and colour (c) from simulated
    g, r, i band light curves; skysurvey relies on sncosmo for bandpass and lightcurve
    generations'
  repo: https://sncosmo.readthedocs.io/en/stable/
- name: skysurvey
  role: Provides the simulated ZTF SN Ia DR2 dataset (dset.data with selection-corrected
    fluxes) on which the SALT2 fits and redshift-dependence study are run
  repo: https://github.com/MickaelRigault/skysurvey
- name: Python
  role: Implementation environment for the fitting and redshift-binning analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:c1b5620147708b8df5a552564b8061ba2fc9e29422a1463188bd4b6e4ef7647c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - skysurvey
  - sncosmo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# salt2-lightcurve-fitting

## Summary

Fit the SALT2 light-curve model to simulated or observed SN Ia photometry (g, r, i bands) with sncosmo to recover stretch (x1) and colour (c) per object, then study their redshift dependence. This is the method used to define the ZTF SN Ia DR2 volume-limited sample boundary (z ≤ 0.06, ~1000 SNe Ia).

## When to use

Use when you need per-object SALT2 stretch and colour from multi-band SN Ia light curves, or when you must locate the redshift above which survey selection effects bias fitted light-curve parameters.

## When NOT to use

- The light curves are not SN Ia photometry or do not span enough of the rise and decline to constrain SALT2 stretch and colour
- The goal is raw survey/dataset generation (Target, Survey, DataSet construction) rather than light-curve parameter fitting
- The sample has already been restricted to a volume- and selection-unbiased regime, so a redshift-dependence study would be redundant

## Inputs

- Multi-band (g, r, i) SN Ia light-curve photometry with flux uncertainties, e.g. a skysurvey dset.data table of simulated observations
- Redshift per SN Ia object
- ZTF SN Ia DR2 selection criteria
- Noise model parameters: error floors 2.5%/3.5%/6% (g/r/i) and sky-noise correction factors 1.23/1.17/1.20 (g/r/i)

## Outputs

- Per-object SALT2 parameter table (x0, x1, c, with uncertainties)
- Stretch (x1) and colour (c) versus redshift binned plots
- Deduced volume-limited redshift limit (expected: z ≤ 0.06) and the corresponding ~1000-SNe-Ia subsample

## How to apply

Load the simulated or observed multi-band light curves (for skysurvey simulations these are in dset.data, produced via DataSet.from_targets_and_survey). For each object, fit the SALT2 model with sncosmo (skysurvey relies on sncosmo for bandpass and lightcurve generations) in the g, r, and i bands, recovering the stretch (x1), colour (c), and normalization (x0). Apply the ZTF SN Ia DR2 selection criteria to the sample before/alongside fitting, and account for the simulation's noise model: error floors of 2.5% (g), 3.5% (r), 6% (i) of the flux level and sky-noise correction factors 1.23 (g), 1.17 (r), 1.20 (i). Bin fitted x1 and c versus redshift across the DR2 redshift range and identify the redshift above which selection effects bias the parameters; below that limit, the stretch and colour distributions must show no redshift trend. The deduced boundary should reproduce the reported z ≤ 0.06 criterion defining the ~1000-SNe-Ia volume-limited sample.

## Related tools

- **sncosmo** (SALT2 light-curve fitting: recovers stretch (x1) and colour (c) from simulated g, r, i band light curves; skysurvey relies on sncosmo for bandpass and lightcurve generations) — https://sncosmo.readthedocs.io/en/stable/
- **skysurvey** (Provides the simulated ZTF SN Ia DR2 dataset (dset.data with selection-corrected fluxes) on which the SALT2 fits and redshift-dependence study are run) — https://github.com/MickaelRigault/skysurvey
- **Python** (Implementation environment for the fitting and redshift-binning analysis)

## Examples

```
import sncosmo; model = sncosmo.Model(source='salt2'); res, fitted_lc = sncosmo.fit_lc(lc, model, ['x0', 'x1', 'c'])
```

## Evaluation signals

- The deduced volume-limited redshift limit equals the reported z ≤ 0.06 criterion
- Within the volume-limited region, binned fitted stretch (x1) and colour (c) show no redshift trend
- The volume-limited subsample contains about 1000 SNe Ia
- Fitted flux uncertainties agree with measured uncertainties only when the g/r/i sky-noise corrections (1.23/1.17/1.20) and error floors (2.5%/3.5%/6%) are applied to the simulated dataset
- The SALT2 parameter table is complete (x0, x1, c per object) for all objects passing the ZTF SN Ia DR2 selection criteria

## Limitations

- The z ≤ 0.06 boundary is specific to the ZTF SN Ia DR2 selection criteria; a different survey or selection requires re-deriving the limit
- The redshift-dependence result is valid only for the simulation's noise model: sky-noise correction factors of 1.23 (g), 1.17 (r), 1.20 (i) and error floors of 2.5%, 3.5%, and 6% of the flux level in g, r, i
- Fitted parameters above the volume limit are biased by selection effects and must not be used for unbiased stretch/colour population studies

## Evidence

- [abstract] a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample: "a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample"
- [abstract] We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06: "We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06"
- [abstract] an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [readme] skysurvey relies on sncosmo for bandpass and lightcurve generations.: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
- [abstract] the simulated fluxes and associated uncertainties agre well with the measured uncertainties: "After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties"
