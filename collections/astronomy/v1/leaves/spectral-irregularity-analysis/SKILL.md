---
name: spectral-irregularity-analysis
description: Use when re-analyzing Fermi-LAT AGN observations for ALP-induced spectral
  oscillations and you need irregularity-based 95% CL exclusion limits around ma ~
  1 neV instead of ALP model fitting.
when_to_use_negative:
- When the goal is likelihood-based fitting of the observed spectrum with ALP models
  — this skill deliberately replaces spectral fitting with irregularity calculation
- When no magnetic field model (turbulent or regular ICMF/GMF) along the sightline
  is available, since conversion probabilities in alpconv depend on the field configuration
- When the data are not gamma-ray spectra of extragalactic sources suitable for ALP-photon
  mixing (e.g., Galactic plane sightlines or non-Fermi-LAT spectral products without
  energy-binned structure)
edam_operation: http://edamontology.org/operation_2950
tools:
- name: alpconv
  role: Calculates ALP-photon conversion probabilities along the source sightlines
    over the (gaγ, ma) grid, for both turbulent and regular magnetic field configurations,
    feeding the irregularity statistic
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Provides the gamma-ray spectral observations of NGC 1275 and PKS 2155-304
    that the irregularity analysis is applied to
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:c4869c6641e6dd9928788ed3a83a6d5fd8419454a82836c537a1c95c6da82ac8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_2950
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectral-irregularity-analysis

## Summary

A model-agnostic re-analysis method for Fermi-LAT gamma-ray spectra of AGN (NGC 1275, PKS 2155-304) that constrains axion-like particle (ALP) parameters by computing spectral irregularity instead of fitting observed spectra with ALP models, yielding 95% confidence-level exclusion contours in the (ma, gaγ) plane.

## When to use

Use when re-analyzing Fermi-LAT AGN observations for ALP-induced spectral oscillations and you need irregularity-based 95% CL exclusion limits around ma ~ 1 neV instead of ALP model fitting.

## When NOT to use

- When the goal is likelihood-based fitting of the observed spectrum with ALP models — this skill deliberately replaces spectral fitting with irregularity calculation
- When no magnetic field model (turbulent or regular ICMF/GMF) along the sightline is available, since conversion probabilities in alpconv depend on the field configuration
- When the data are not gamma-ray spectra of extragalactic sources suitable for ALP-photon mixing (e.g., Galactic plane sightlines or non-Fermi-LAT spectral products without energy-binned structure)

## Inputs

- Fermi-LAT binned spectral event data for NGC 1275 (or 12-year PKS 2155-304 / 4FGL J2158.8-3013 observations)
- ALP-photon conversion probability grids from alpconv over (gaγ, ma) for turbulent and/or regular magnetic field configurations
- Null (no-ALP) baseline spectra and their irregularity distributions

## Outputs

- 95% confidence-level exclusion regions in the (ma, gaγ) plane
- Exclusion-contour figures and per-parameter-set verdict tables (e.g., combined_exclusion_region.csv, regular_field_limits.csv)
- Comparison records quantifying single-source vs. combined and turbulent vs. regular-field limit changes (e.g., combined_vs_single_bounds.csv, regular_vs_turbulent_comparison.json)

## How to apply

Load the Fermi-LAT binned spectral observations of the source (e.g., NGC 1275, or 12 years of PKS 2155-304 / 4FGL J2158.8-3013 data). Compute ALP-photon conversion probabilities with alpconv over a grid of ALP masses (focused around ma ~ 1 neV) and couplings (gaγ >= 3e-12 GeV^-1), optionally switching alpconv from the turbulent-field baseline to a regular ICMF/GMF magnetic field configuration as a variant. Rather than fitting the observed spectrum with ALP models, calculate the irregularity of the spectrum — both for the observed data and for model spectra at each (gaγ, ma) grid point — and build null (no-ALP) baseline distributions of the irregularity statistic as the reference. Flag parameter sets as excluded at the 95% confidence level when their irregularity deviates from the null baseline; multiple sources (NGC 1275 + PKS 2155-304) can be combined into a joint irregularity statistic to further tighten the constraint.

## Related tools

- **alpconv** (Calculates ALP-photon conversion probabilities along the source sightlines over the (gaγ, ma) grid, for both turbulent and regular magnetic field configurations, feeding the irregularity statistic) — github.com/lyf222/alpconv
- **Fermi-LAT** (Provides the gamma-ray spectral observations of NGC 1275 and PKS 2155-304 that the irregularity analysis is applied to)

## Evaluation signals

- The derived exclusion region must rule out couplings gaγ > 3×10⁻¹² GeV⁻¹ around ALP mass ma ~ 1 neV at 95% confidence level, matching the paper's reported bound
- Per-parameter-set verdicts must be assigned by comparing each grid point's irregularity against the null (no-ALP) baseline distribution at the 95% CL threshold
- When combining sources, the joint exclusion region must strictly enlarge (improve upon) both single-source regions
- The regular-field variant limits should be expressed as 95% CL exclusions comparable in stringency to the turbulent-field baseline
- No spectral oscillation signal should be reported for NGC 1275 under this method, consistent with the paper's null finding

## Limitations

- alpconv provides no usage instructions and code comments remain in Chinese, so setting up the package may require reading the referenced arXiv:1802.08420 paper and the code directly
- Results depend on the assumed magnetic field configuration: the paper explicitly tested how limits change between the turbulent-field baseline and a regular magnetic field model
- The paper reports no spectral oscillation found for NGC 1275 with this method; the skill constrains couplings rather than detecting a signal

## Evidence

- [abstract] irregularity method replaces spectral fitting: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] reported 95% CL exclusion bound: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [intro] alpconv role in the pipeline: "alpconv calculating alp-photon conversion"
- [readme] alpconv usability caveat: "(at present no intruction for use provides, chinese character comments remain in the code.)"
