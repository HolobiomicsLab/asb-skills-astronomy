---
name: template-matching-rv-extraction
description: Use when extracting RV time-series from ESPRESSO or HARPS S2D echelle
  spectra and you need a template-matching alternative to CCF methods with lower RV
  scatter (e.g. ~10% for M-type stars).
when_to_use_negative:
- You need per-order RV estimates combined by weighted average (the traditional template-matching
  mode) rather than a common RV shift across orders
- Input is not HARPS or ESPRESSO data — the paper states S-BART 'can currently be
  applied to HARPS and ESPRESSO data' only
- A full analysis of planetary and activity signals is the goal — the authors leave
  this for future work
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: S-BART
  role: 'Main pipeline: semi-Bayesian template-matching RV extraction from HARPS and
    ESPRESSO spectra'
  repo: https://github.com/iastro-pt/sBART
- name: Telfit
  role: Telluric absorption spectrum modeling (LBLRTM-based) used to create telluric
    templates; must be installed independently
  repo: https://github.com/kgullikson88/Telluric-Fitter
- name: pysynphot
  role: Dependency that may be needed if Telfit fails with certain errors
- name: ESO official pipeline
  role: Reference CCF RVs for comparison of RV scatter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:bbd90cf0da2011491fbad6800e4216db6ce144204c09922246d86c2b168f7095
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESO official pipeline
  - S-BART
  - SBART
  - Telfit
  - pysynphot
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# template-matching-rv-extraction

## Summary

Extract precise radial velocities via the semi-Bayesian template-matching approach (S-BART), in which a common RV shift describes the difference between each spectral order and a stellar template built from the available observations, with posteriors obtained through Laplace approximation.

## When to use

Use when extracting RV time-series from ESPRESSO or HARPS S2D echelle spectra and you need a template-matching alternative to CCF methods with lower RV scatter (e.g. ~10% for M-type stars).

## When NOT to use

- You need per-order RV estimates combined by weighted average (the traditional template-matching mode) rather than a common RV shift across orders
- Input is not HARPS or ESPRESSO data — the paper states S-BART 'can currently be applied to HARPS and ESPRESSO data' only
- A full analysis of planetary and activity signals is the goal — the authors leave this for future work

## Inputs

- ESPRESSO or HARPS S2D spectral frames (e.g. 33 ESPRESSO GTO targets)
- GDAS atmospheric profiles for TelFit telluric modeling
- ESO official pipeline (CCF) RVs for the same observations (for comparison)
- DACE secular acceleration values

## Outputs

- RV time-series text files with columns: BJD, RVc [km/s], RVc_ERR [km/s], SA [km/s], DRIFT [m/s], filename, frameIDs
- Stellar templates built from available observations
- Telluric templates from TelFit

## How to apply

Install SBART (pip install SBART, plus Telfit installed independently with pip install Telfit) and load ESPRESSO/HARPS S2D frames into the DataClass, applying frame-rejection conditions such as order-wise SNR checks, warning-keyword flags, and MAD flux-outlier flagging. Build a stellar template from the available observations and a telluric template with TelFit using GDAS atmospheric profiles. Run the RV extraction (RV_step) with the Laplace-approximation sampler, fitting a common RV shift across spectral orders against the stellar template and marginalizing over the continuum. Post-process with BERV correction (average relativistic terms, similar to the ESPRESSO DRS), instrumental drift correction, and secular acceleration (SA) from DACE, then write RV time-series text files with columns BJD, RVc [km/s], RVc_ERR [km/s], SA [km/s], DRIFT [m/s], filename, frameIDs. The common-RV-shift model is the distinguishing choice: instead of estimating an RV per spectral order and combining via weighted average, one RV shift describes all orders of a given spectrum, which the authors attribute performance gains to via template construction and telluric handling.

## Related tools

- **S-BART** (Main pipeline: semi-Bayesian template-matching RV extraction from HARPS and ESPRESSO spectra) — https://github.com/iastro-pt/sBART
- **Telfit** (Telluric absorption spectrum modeling (LBLRTM-based) used to create telluric templates; must be installed independently) — https://github.com/kgullikson88/Telluric-Fitter
- **pysynphot** (Dependency that may be needed if Telfit fails with certain errors)
- **ESO official pipeline** (Reference CCF RVs for comparison of RV scatter)

## Examples

```
pip install SBART Telfit
```

## Evaluation signals

- Median RV scatter decrease of ~10% for M-type and ~4% for K-type stars relative to the ESO CCF method on ESPRESSO GTO targets
- Median RV uncertainty of ~15 cm/s over 309 observations of 16 M-type targets with ESPRESSO
- On Barnard's star HARPS archival data, S-BART RV standard deviation should be smaller than that of HARPS-TERRA and SERVAL
- Output text files must contain the default columns BJD, RVc [km/s], RVc_ERR [km/s], SA [km/s], DRIFT [m/s], filename, frameIDs
- Nightly zero point (NZP) weighted scatter of ESPRESSO should fall below ~0.7 m/s, treated as an upper limit on attainable RV precision

## Limitations

- S-BART can currently be applied only to HARPS and ESPRESSO data
- Telfit and/or pysynphot can fail with known errors; GDAS profiles may be unavailable for some reference frames (a fallback for Telfit templates exists)
- Full analysis of the planetary and activity signals in the considered datasets is deferred to future work; the ~0.7 m/s NZP scatter is an upper limit on attainable precision
- Performance differences are attributed to template construction and telluric feature handling, so results depend on these choices

## Evidence

- [abstract] common RV shift model across orders: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations."
- [abstract] Laplace approximation posterior: "Posterior probability"
- [abstract] applicability limited to HARPS and ESPRESSO: "This methodology, named the Semi-Bayesian Approach for RVs with Template matching, S-BART, can currently be applied to HARPS and ESPRESSO data."
- [abstract] RV scatter decrease vs CCF: "We find a decrease in the median RV scatter of ~10 and ~4% for M- and K-type stars, respectively."
- [abstract] median RV uncertainty on ESPRESSO: "S-BART achieves a median uncertainty of ~15 cm s−1over 309 observations of 16 targets"
- [methods] pip installation: "SBART can be pip-installed:

    pip install SBART"
- [methods] Telfit installation: "To use `Telfit <https://github.com/kgullikson88/Telluric-Fitter>`_. it must be installed independently:

    pip install Telfit"
- [methods] frame rejection conditions: "Added a new condition to check order-wise SNR"
- [methods] BERV correction detail: "Added the average value for the relativistic terms in the BERV correction (similarly to ESPRESSO DRS)"
- [abstract] future work caveat: "We leave a full analysis of the planetary and activity signals present in the considered datasets for future work."
