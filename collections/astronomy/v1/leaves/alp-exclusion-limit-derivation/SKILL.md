---
name: alp-exclusion-limit-derivation
description: Use when re-analyzing a Fermi-LAT observation (e.g. NGC 1275) to constrain
  ALP-photon coupling gaγ as a function of ALP mass ma, or when testing how exclusion
  limits change under different ICMF/GMF magnetic field configurations (turbulent
  vs regular).
when_to_use_negative:
- When the goal is to fit the observed spectrum with ALP models — the paper's method
  deliberately replaces spectral fitting with an irregularity-based calculation
- When a spectral oscillation signal has already been detected and the goal is detection/significance
  estimation rather than exclusion-limit derivation
- When the source data is not a Fermi-LAT gamma-ray observation compatible with the
  alpconv conversion-probability pipeline
tools:
- name: Fermi-LAT
  role: Source of the gamma-ray observation of NGC 1275 that is re-analyzed to search
    for ALP effects and constrain ALP parameters
- name: alpconv
  role: Calculates ALP-photon conversion probabilities over the (ma, gaγ) grid for
    the configured magnetic field variant
  repo: github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:41945afa026058c20b780d1d9a5d59487899c039e63d112c9c7555d9e79c8d81
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# alp-exclusion-limit-derivation

## Summary

Derives 95% confidence-level exclusion limits for axion-like particle (ALP) coupling (gaγ) versus mass (ma) from Fermi-LAT gamma-ray spectra using an irregularity-based method with ALP-photon conversion probabilities computed via the alpconv package, rather than fitting observed spectra with ALP models.

## When to use

Use when re-analyzing a Fermi-LAT observation (e.g. NGC 1275) to constrain ALP-photon coupling gaγ as a function of ALP mass ma, or when testing how exclusion limits change under different ICMF/GMF magnetic field configurations (turbulent vs regular).

## When NOT to use

- When the goal is to fit the observed spectrum with ALP models — the paper's method deliberately replaces spectral fitting with an irregularity-based calculation
- When a spectral oscillation signal has already been detected and the goal is detection/significance estimation rather than exclusion-limit derivation
- When the source data is not a Fermi-LAT gamma-ray observation compatible with the alpconv conversion-probability pipeline

## Inputs

- Fermi-LAT spectral/likelihood data for the target source (e.g. NGC 1275 observation)
- Baseline exclusion limits from a prior analysis run (e.g. turbulent-field limits) for comparison
- Magnetic field configuration specification (turbulent ICMF baseline or regular ICMF/GMF model)

## Outputs

- 95% confidence-level exclusion limits in the (ma, gaγ) plane (e.g. regular_field_limits.csv)
- Comparison record of excluded regions across magnetic field variants (e.g. regular_vs_turbulent_comparison.json)

## How to apply

Load the Fermi-LAT spectral/likelihood data for the target source (e.g. NGC 1275) and configure the alpconv package (github.com/lyf222/alpconv) to calculate ALP-photon conversion probabilities over the (ma, gaγ) grid for the chosen magnetic field setup (turbulent baseline or regular ICMF/GMF variant). Instead of fitting the observed spectrum with ALP models, apply the paper's alternative method that calculates the irregularity of the spectrum induced by ALP-photon oscillations, and derive 95% confidence-level exclusion limits in the (ma, gaγ) plane from the irregularity statistic. If comparing magnetic field variants, tabulate the new limits and quantify the change in the excluded region relative to the turbulent-field baseline; the NGC 1275 result should rule out couplings gaγ > 3×10⁻¹² GeV⁻¹ around ma ∼ 1 neV at 95% confidence level. Constraints can be further improved by combining multiple sources (e.g. PKS 2155-304), yielding a wider exclusion region.

## Related tools

- **Fermi-LAT** (Source of the gamma-ray observation of NGC 1275 that is re-analyzed to search for ALP effects and constrain ALP parameters)
- **alpconv** (Calculates ALP-photon conversion probabilities over the (ma, gaγ) grid for the configured magnetic field variant) — github.com/lyf222/alpconv

## Examples

```
python run_alpconv.py --source NGC1275 --field regular --grid ma,ga_gammagamma; # then apply irregularity method to derive 95% CL limits
```

## Evaluation signals

- Derived limits are expressed as 95% confidence-level exclusions in the (ma, gaγ) plane
- Reproduced stringency matches the paper: couplings gaγ > 3×10⁻¹² GeV⁻¹ around ALP mass ma ∼ 1 neV are ruled out
- Comparison record quantifies the change in the excluded region relative to the turbulent-field baseline rather than only absolute limits
- Conversion probabilities span a grid over ALP mass (ma) and coupling (gaγ), not a single-point calculation
- The method computes spectral irregularity rather than performing ALP model fits to the observed spectrum

## Limitations

- The alpconv repository provides no usage instructions and code comments remain in Chinese, so setup requires reading the referenced paper (arXiv:1802.08420, PKS 2155-304) for methodology
- No spectral oscillation was found for NGC 1275 with this method — the output is an exclusion limit, not a detection
- Exclusion regions are limited by the single-source analysis; a much wider exclusion region requires combining more sources (e.g. PKS 2155-304)

## Evidence

- [abstract] abstract: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [intro] intro: "alpconv calculating alp-photon conversion"
- [readme] readme: "(at present no intruction for use provides, chinese character comments remain in the code.)"
