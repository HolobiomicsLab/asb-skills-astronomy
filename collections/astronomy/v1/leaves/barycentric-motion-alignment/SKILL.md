---
name: barycentric-motion-alignment
description: 'Use when constructing an S-BART stellar template from multiple HARPS
  or ESPRESSO S2D observations: spectra taken at different BERV values must first
  be shifted to a common barycentric frame before order-wise co-addition.'
when_to_use_negative:
- A stellar template for the target sub-Instrument already exists in the templates
  folder and is loadable by SBART — reuse it rather than re-aligning and re-co-adding
  frames
- The task is telluric template creation or telluric handling (Telfit/GDAS-profile
  based), not stellar template construction from observations
- Input frames fail the order-wise SNR condition or are flagged by warning keyword
  flags — reject them via the frame-selection step instead of aligning them into the
  template
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: S-BART
  role: Pipeline whose SBART DataClass applies BERV corrections and co-adds selected
    S2D frames order-by-order into the stellar template per sub-Instrument
  repo: https://github.com/iastro-pt/sBART
- name: HARPS
  role: Instrument providing the reduced S2D archival observations (e.g. Barnard's
    star) that are barycentrically aligned and co-added
- name: ESPRESSO
  role: Instrument whose DRS BERV convention (average relativistic terms) S-BART follows
    when correcting barycentric motion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:b096c621e656db001d1cec519420efd28085f7482184631f6c44a494cdb5090e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESPRESSO
  - HARPS
  - S-BART
  - SBART
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# barycentric-motion-alignment

## Summary

Aligns reduced HARPS/ESPRESSO S2D spectral frames into a common barycentric rest frame — applying BERV corrections including the average relativistic terms (as in ESPRESSO DRS) — so that selected observations can be co-added order-by-order into the single S-BART stellar template that underpins its semi-Bayesian RV estimation.

## When to use

Use when constructing an S-BART stellar template from multiple HARPS or ESPRESSO S2D observations: spectra taken at different BERV values must first be shifted to a common barycentric frame before order-wise co-addition.

## When NOT to use

- A stellar template for the target sub-Instrument already exists in the templates folder and is loadable by SBART — reuse it rather than re-aligning and re-co-adding frames
- The task is telluric template creation or telluric handling (Telfit/GDAS-profile based), not stellar template construction from observations
- Input frames fail the order-wise SNR condition or are flagged by warning keyword flags — reject them via the frame-selection step instead of aligning them into the template

## Inputs

- Reduced S2D spectral observations from HARPS or ESPRESSO (33 ESPRESSO GTO targets / Barnard's star HARPS archival data in the paper)
- BERV values per observation, including relativistic terms for the barycentric correction
- Instrumental drift values (DRIFT column, in km/s) for RV correction
- Frame-selection metadata: order-wise SNR values and warning keyword flags

## Outputs

- Stellar template FITS files per sub-Instrument, written to the templates folder
- Text files summarizing included and rejected observations
- Metric plots of the amount of pixels rejected in each spectral order
- Barycentrically aligned, order-wise co-added stellar spectra feeding the semi-Bayesian RV estimation

## How to apply

Load the reduced S2D frames into the SBART DataClass, then reject frames failing order-wise SNR checks or carrying warning keyword flags, and flag flux-outlier pixels with the MAD-based criterion before any co-addition. Correct each surviving frame for barycentric Earth RV (BERV), applying the average value of the relativistic terms in the BERV correction similarly to the ESPRESSO DRS, so all spectra share one barycentric reference. Combine the aligned observations order-by-order into a single stellar template per sub-Instrument with SBART's stellar-template creation routine, choosing the NORMALIZATION_MODE used to normalize the stellar spectra. The rationale is that template construction is credited for S-BART's smaller RV scatter versus HARPS-TERRA and SERVAL, so frame selection and frame alignment errors propagate directly into RV precision. Write the template to disk as FITS files in the templates folder, together with text files listing included and rejected data, and produce metric plots of rejected pixels per spectral order.

## Related tools

- **S-BART** (Pipeline whose SBART DataClass applies BERV corrections and co-adds selected S2D frames order-by-order into the stellar template per sub-Instrument) — https://github.com/iastro-pt/sBART
- **HARPS** (Instrument providing the reduced S2D archival observations (e.g. Barnard's star) that are barycentrically aligned and co-added)
- **ESPRESSO** (Instrument whose DRS BERV convention (average relativistic terms) S-BART follows when correcting barycentric motion)

## Examples

```
pip install SBART
```

## Evaluation signals

- The stellar template FITS file exists in the templates folder and is loadable by SBART
- Per-order pixel-rejection metric plots were produced for each sub-Instrument, allowing diagnosis of chromatic issues or outlier observations
- Text files accompanying the template correctly list the included and rejected frames (SNR and warning-flag rejections applied before co-addition)
- BERV correction applied the average relativistic terms, matching the ESPRESSO DRS convention
- Resulting RV standard deviation is at or below that of HARPS-TERRA and SERVAL on the same dataset (e.g. Barnard's star HARPS data)

## Limitations

- Frame-selection choices (order-wise SNR checks, warning keyword flags, MAD flux-outlier flagging, NORMALIZATION_MODE) directly shape the template and thus the RVs; misconfiguration degrades precision
- Telfit needs a fallback to its own templates when GDAS profiles cannot be downloaded for a reference frame, affecting the telluric side of the pipeline
- The paper leaves a full analysis of the planetary and activity signals present in the datasets for future work, so template-level RV scatter is not a full scientific validation
- Telluric-feature handling differences (alongside template construction) are believed to drive performance differences versus HARPS-TERRA and SERVAL

## Evidence

- [abstract] Template shift assumes common RV: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations"
- [abstract] Template construction credited for performance: "We believe this is due to differences in the construction of the stellar template and the handling of telluric features."
- [methods] BERV relativistic terms follow ESPRESSO DRS: "Added the average value for the relativistic terms in the BERV correction (similarly to ESPRESSO DRS)"
- [methods] Frame selection via MAD outlier flagging: "Allow to use MAD to flag flux outliers."
- [readme] s-BART as template-matching implementation: "s-BART: A semi-Bayesian implementation of template matching for precise Radial Velocities"
