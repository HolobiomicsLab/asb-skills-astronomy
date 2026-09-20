---
name: rv-scatter-comparison-analysis
description: Use when you have RV time series from two extraction methods (e.g., S-BART
  template matching vs ESO DRS CCF) on the same instrument frames and need a scatter-based
  precision comparison per stellar type.
when_to_use_negative:
- Comparing against another template-matching pipeline rather than the CCF method
  — for HARPS data S-BART is instead validated against HARPS-TERRA and SERVAL.
- Analyzing planetary or activity signals — the authors explicitly leave this for
  future work; this skill only measures RV scatter/precision.
- Only single-instrument RVs are available with no reference method RVs — no comparison
  baseline exists.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SBART
  role: Produces the template-matching RV time series (pip install SBART) whose weighted
    scatter is compared against CCF
  repo: https://github.com/iastro-pt/sBART
- name: ESO official pipeline
  role: Source of the reference CCF-method RVs for the same ESPRESSO observations
- name: Telfit
  role: Builds the telluric templates (with GDAS atmospheric profiles) used before
    RV extraction so telluric handling does not bias the comparison
  repo: https://github.com/kgullikson88/Telluric-Fitter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:4bd3038639399ee76cec213f042dee731302f40416622bbb4fe65e270c71989e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESO official pipeline
  - S-BART
  - SBART
  - Telfit
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# rv-scatter-comparison-analysis

## Summary

Compare per-target weighted RV scatter from a template-matching pipeline (S-BART) against the ESO CCF pipeline RVs on ESPRESSO GTO observations, reporting the median scatter decrease separately for M- and K-type stars. Used to quantify precision gains of template matching over cross-correlation.

## When to use

Use when you have RV time series from two extraction methods (e.g., S-BART template matching vs ESO DRS CCF) on the same instrument frames and need a scatter-based precision comparison per stellar type.

## When NOT to use

- Comparing against another template-matching pipeline rather than the CCF method — for HARPS data S-BART is instead validated against HARPS-TERRA and SERVAL.
- Analyzing planetary or activity signals — the authors explicitly leave this for future work; this skill only measures RV scatter/precision.
- Only single-instrument RVs are available with no reference method RVs — no comparison baseline exists.

## Inputs

- ESPRESSO GTO S2D frames for 33 targets (after frame rejection: order-wise SNR check, warning keyword flags, MAD flux-outlier flagging)
- S-BART RV time-series text files (BJD, RVc [km/s], RVc_ERR [km/s], SA [km/s], DRIFT [m/s], filename, frameIDs)
- ESO official pipeline CCF RVs for the same observations
- Stellar templates built from available observations; telluric templates from TelFit with GDAS atmospheric profiles

## Outputs

- Per-target weighted RV scatter for S-BART and for CCF
- Median RV scatter decrease per stellar type (~10% M-type, ~4% K-type)
- Per-target comparison table of S-BART vs CCF RV scatter and median RV uncertainties

## How to apply

Run the RV extraction (RV_step) with S-BART's Laplace-approximation sampler, fitting a common RV shift across spectral orders against the stellar template, then apply the BERV correction (average relativistic terms as in the ESPRESSO DRS), instrumental drift correction, and secular acceleration (SA) correction from DACE, writing time-series files with BJD, RVc [km/s], RVc_ERR [km/s], SA [km/s], DRIFT [m/s], filename, frameIDs columns. Load the ESO official pipeline CCF RVs for the same observations. For each of the 33 ESPRESSO GTO targets, compute the weighted RV scatter of the S-BART time series and of the CCF time series, then take the ratio per target. Finally take the median decrease across targets separately for M-type and K-type stars, so that sensitivity of any single target does not dominate the comparison. Judge correctness by reproducing ~10% median decrease for M-type and ~4% for K-type stars, and by S-BART M-type RVs reaching a median uncertainty of ~15 cm/s over 309 observations of 16 targets.

## Related tools

- **SBART** (Produces the template-matching RV time series (pip install SBART) whose weighted scatter is compared against CCF) — https://github.com/iastro-pt/sBART
- **ESO official pipeline** (Source of the reference CCF-method RVs for the same ESPRESSO observations)
- **Telfit** (Builds the telluric templates (with GDAS atmospheric profiles) used before RV extraction so telluric handling does not bias the comparison) — https://github.com/kgullikson88/Telluric-Fitter

## Examples

```
pip install SBART; pip install Telfit  # then run RV_step on ESPRESSO S2D frames and compare weighted scatter vs ESO CCF RVs per target
```

## Evaluation signals

- Median RV scatter decrease reproduces ~10% for M-type and ~4% for K-type stars relative to CCF
- S-BART M-type ESPRESSO RVs reach a median uncertainty of ~15 cm/s over 309 observations of 16 targets
- S-BART and CCF RV time series are computed on the identical frame sets after identical rejection (order-wise SNR, warning KW flags, MAD flux outliers)
- Output text files contain the default columns BJD, RVc [km/s], RVc_ERR [km/s], SA [km/s], DRIFT [m/s], filename, frameIDs
- Scatter is computed as weighted RV scatter per target, then median taken per spectral type — not pooled across targets

## Limitations

- Weighted NZP scatter of ESPRESSO below ~0.7 m/s should be taken as an upper limit on the RV precision attainable with ESPRESSO data, bounding interpretation of scatter gains.
- Performance differences vs CCF are attributed to template construction and telluric handling, so results depend on TelFit/GDAS telluric template quality.
- Full analysis of planetary and activity signals present in the datasets is deferred to future work; scatter comparison alone does not validate astrophysical signal recovery.
- Telfit/pysynphot may fail with known errors (fallback needed for reference frames without downloadable GDAS profiles).

## Evidence

- [abstract] median decrease per stellar type: "We find a decrease in the median RV scatter of ~10 and ~4% for M- and K-type stars, respectively."
- [abstract] median uncertainty: "S-BART achieves a median uncertainty of ~15 cm s−1over 309 observations of 16 targets"
- [abstract] common RV shift fit: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations."
- [abstract] attribution of differences: "We believe this is due to differences in the construction of the stellar template and the handling of telluric features."
- [abstract] NZP upper limit: [evidence span withheld — anchor status: not_found]
- [methods] BERV correction: "Added the average value for the relativistic terms in the BERV correction (similarly to ESPRESSO DRS)"
- [methods] frame rejection conditions: "Added a new condition to check order-wise SNR"
- [readme] TelFit installation: "To use `Telfit <https://github.com/kgullikson88/Telluric-Fitter>`_. it must be installed independently:

    pip install Telfit"
