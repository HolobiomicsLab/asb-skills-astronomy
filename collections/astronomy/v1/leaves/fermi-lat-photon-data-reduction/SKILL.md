---
name: fermi-lat-photon-data-reduction
description: Use when the analysis goal is to constrain axion-like particle (ALP)
  parameters from gamma-ray spectra of a blazar and requires binned spectral extraction
  from multi-year (e.g., 12-year) Fermi-LAT photon event data.
when_to_use_negative:
- The observed spectrum is to be fitted directly with ALP models — the NGC 1275 re-analysis
  explicitly adopts the irregularity-based method instead of model fitting
- No public Fermi-LAT photon event data are available for the target source, so no
  binned spectrum can be extracted
tools:
- name: Fermi-LAT
  role: Source of the public photon event data (12 years for PKS 2155-304) used for
    binned spectral extraction in the ALP search
- name: alpconv
  role: Calculates ALP-photon conversion probabilities over the gaγ–ma grid from the
    extracted spectrum
  repo: https://github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:f0872949a57818baeb47a8488bce91d10faa1e4a2b3205d9fcf54e74e5acec7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lyf222/alpconv
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fermi-lat-photon-data-reduction

## Summary

Retrieves and reduces multi-year public Fermi-LAT photon event data for a blazar (e.g., PKS 2155-304 / 4FGL J2158.8-3013) into binned spectra that feed ALP-photon conversion analyses with alpconv.

## When to use

Use when the analysis goal is to constrain axion-like particle (ALP) parameters from gamma-ray spectra of a blazar and requires binned spectral extraction from multi-year (e.g., 12-year) Fermi-LAT photon event data.

## When NOT to use

- The observed spectrum is to be fitted directly with ALP models — the NGC 1275 re-analysis explicitly adopts the irregularity-based method instead of model fitting
- No public Fermi-LAT photon event data are available for the target source, so no binned spectrum can be extracted

## Inputs

- 12 years of public Fermi-LAT photon event data for the target blazar (e.g., PKS 2155-304 / 4FGL J2158.8-3013)
- ALP parameter grid of couplings gaγ and masses ma

## Outputs

- Binned spectral extraction of the source from the Fermi-LAT event data
- ALP-photon conversion probabilities per (gaγ, ma) grid point
- Null (no-ALP) baseline distributions of the spectral irregularity statistic
- Exclusion-region table with per-parameter-set verdicts at 95% confidence level
- Exclusion-region figure saved to disk

## How to apply

Retrieve 12 years of public Fermi-LAT photon event data for the target source (e.g., PKS 2155-304, 4FGL J2158.8-3013) and perform binned spectral extraction as the first pipeline stage. Pass the resulting binned spectrum to the alpconv package, which calculates ALP-photon conversion probabilities over a grid of couplings gaγ and masses ma. Rather than fitting the observed spectrum with ALP models, compute the irregularity of the observed spectrum for each grid point and build null (no-ALP) baseline distributions of the irregularity statistic. Compare each parameter-set irregularity against the null baseline and flag parameter sets as excluded at the 95% confidence level, mirroring the NGC 1275 re-analysis methodology. Assemble per-parameter-set verdicts into an exclusion-region table and figure.

## Related tools

- **Fermi-LAT** (Source of the public photon event data (12 years for PKS 2155-304) used for binned spectral extraction in the ALP search)
- **alpconv** (Calculates ALP-photon conversion probabilities over the gaγ–ma grid from the extracted spectrum) — https://github.com/lyf222/alpconv

## Evaluation signals

- The null (no-ALP) baseline irregularity distributions are constructed before any exclusion verdicts are assigned
- Exclusion verdicts are reported per (gaγ, ma) parameter set at the 95% confidence level, reproducing the benchmark result that gaγ > 3×10⁻¹² GeV⁻¹ is ruled out around ma ∼ 1 neV
- The exclusion-region table and exclusion-region figure are both assembled and saved to disk
- The irregularity statistic is computed on the observed spectrum itself, not via ALP model fits
- The exclusion region obtained should be consistent with (or more stringent than) previous PKS 2155-304 ALP bounds when cross-checked

## Limitations

- The alpconv package provides no usage instructions and its code comments remain in Chinese, hindering reuse
- Exclusion regions are strongest near ma ∼ 1 neV; constraints depend on combining multiple sources — a wider exclusion region requires taking more sources into account

## Evidence

- [abstract] irregularity-based method instead of model fitting: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] 95% CL benchmark exclusion: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [abstract] PKS 2155-304 as an independent constraint: "We also show that the constraints can be further improved by combining the observation of PKS 2155-304."
- [intro] alpconv purpose: "alpconv calculating alp-photon conversion"
- [readme] alpconv usability caveat: "(at present no intruction for use provides, chinese character comments remain in the code.)"
