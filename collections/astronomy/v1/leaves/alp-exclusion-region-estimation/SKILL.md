---
name: alp-exclusion-region-estimation
description: Use when re-analyzing long-baseline Fermi-LAT observations (e.g. 12 years
  of PKS 2155-304 / 4FGL J2158.8-3013 or NGC 1275) to test whether ALP-induced spectral
  oscillation structure can be excluded, using irregularity statistics rather than
  spectral model fitting.
when_to_use_negative:
- When the goal is to detect a spectral oscillation signal (the method finds none
  — the NGC 1275 re-analysis found no spectral oscillation; it is an exclusion method)
- When fitting ALP models directly to the observed spectrum is desired — this skill
  intentionally replaces model fitting with irregularity calculation
- When a working, documented ALP-conversion tool with usage instructions is required
  — the alpconv code currently has no usage instructions and Chinese-language comments
tools:
- name: Fermi-LAT
  role: Source of the public multi-year photon event data and binned spectral extraction
    on which the irregularity analysis is run
- name: alpconv
  role: Calculates ALP-photon conversion probabilities over the (gaγ, ma) grid for
    each candidate parameter set
  repo: https://github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:34df6c4d50219b9298b8bf779731e3e6612fff6cfd4925d5521deeb5bc39990b
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# alp-exclusion-region-estimation

## Summary

A model-irregularity-based procedure for excluding axion-like particle (ALP) parameter points (coupling gaγ vs mass ma) from Fermi-LAT gamma-ray spectra. Instead of fitting spectra with ALP models, it computes the irregularity of the observed spectrum and compares each grid point against a null (no-ALP) baseline to flag 95% C.L. exclusions.

## When to use

Use when re-analyzing long-baseline Fermi-LAT observations (e.g. 12 years of PKS 2155-304 / 4FGL J2158.8-3013 or NGC 1275) to test whether ALP-induced spectral oscillation structure can be excluded, using irregularity statistics rather than spectral model fitting.

## When NOT to use

- When the goal is to detect a spectral oscillation signal (the method finds none — the NGC 1275 re-analysis found no spectral oscillation; it is an exclusion method)
- When fitting ALP models directly to the observed spectrum is desired — this skill intentionally replaces model fitting with irregularity calculation
- When a working, documented ALP-conversion tool with usage instructions is required — the alpconv code currently has no usage instructions and Chinese-language comments

## Inputs

- Multi-year public Fermi-LAT photon event data for the target source (e.g. 12 years for PKS 2155-304, 4FGL J2158.8-3013)
- Binned extracted gamma-ray spectrum of the source
- Grid of ALP couplings gaγ and masses ma
- Null (no-ALP) baseline irregularity distributions

## Outputs

- Exclusion-region table with per-parameter-set (gaγ, ma) excluded/excluded-not verdicts at 95% C.L.
- Exclusion-region figure in the (ma, gaγ) plane, saved to disk

## How to apply

Retrieve the multi-year public Fermi-LAT photon event data for the target blazar and perform binned spectral extraction. For each point on a grid of ALP couplings gaγ and masses ma, compute the ALP-photon conversion probability with the alpconv package. For each grid point, calculate the irregularity of the resulting oscillated spectrum — deliberately not fitting the observed spectrum with ALP models — and build null (no-ALP) baseline distributions of the irregularity statistic from the extracted spectrum. Compare each parameter-set irregularity against the null baseline and mark parameter sets excluded at the 95% confidence level; in the NGC 1275 re-analysis this ruled out gaγ > 3×10⁻¹² GeV⁻¹ around ma ∼ 1 neV. Assemble a per-parameter-set exclusion table and render the exclusion-region figure, saving both to disk.

## Related tools

- **Fermi-LAT** (Source of the public multi-year photon event data and binned spectral extraction on which the irregularity analysis is run)
- **alpconv** (Calculates ALP-photon conversion probabilities over the (gaγ, ma) grid for each candidate parameter set) — https://github.com/lyf222/alpconv

## Evaluation signals

- Exclusion verdicts are defined per parameter set against null (no-ALP) irregularity baselines — each flagged exclusion must trace to a 95% confidence-level comparison
- Reproduced result should rule out couplings gaγ > 3×10⁻¹² GeV⁻¹ around ma ∼ 1 neV for NGC 1275 (matching the published bound)
- Applying the same procedure to PKS 2155-304 should exclude the same ALP-induced spectral structure as the NGC 1275 analysis
- Output must include both the exclusion-region table (per-parameter-set verdicts) and the rendered exclusion-region figure saved to disk
- The analysis must compute irregularity rather than fit the observed spectrum with ALP models — presence of ALP spectral fitting indicates the wrong method was applied

## Limitations

- The alpconv package provides no usage instructions and code comments remain in Chinese (per the README), so reproducing the conversion-probability grid requires code inspection
- No spectral oscillation was found for NGC 1275 with this method — it yields exclusion regions, not detections
- The exclusion region depends on using an irregularity-based method; previous (fit-based) analyses gave different (less stringent) results
- Constraints from a single source are limited; combining observations of more sources (e.g. NGC 1275 with PKS 2155-304) improves them

## Evidence

- [abstract] irregularity method instead of fitting: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] published 95% C.L. bound: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [abstract] PKS 2155-304 combination: "We also show that the constraints can be further improved by combining the observation of PKS 2155-304."
- [abstract] no oscillation signal: "With the newly used method, we find no spectral oscillation for the NGC 1275"
- [readme] alpconv usability caveat: "(at present no intruction for use provides, chinese character comments remain in the code.)"
