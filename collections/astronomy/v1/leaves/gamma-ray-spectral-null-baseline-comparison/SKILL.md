---
name: gamma-ray-spectral-null-baseline-comparison
description: Use when re-analyzing Fermi-LAT (or similar) blazar spectra for ALP-photon
  conversion effects with an irregularity method, where a no-oscillation reference
  spectrum must be built instead of fitting ALP models to the observed spectrum.
when_to_use_negative:
- When the goal is to fit the observed spectrum directly with ALP models — this skill
  deliberately avoids model fitting in favor of irregularity quantification
- When the source spectrum is not from a photon-attenuation-regime observation (ALP
  conversion constraints here rely on gamma-ray spectra; the article additionally
  notes PKS 2155-304 must be analyzed for combined constraints, not NGC 1275 alone
  for its bound)
edam_operation: http://edamontology.org/operation_2950
tools:
- name: alpconv
  role: Calculates ALP-photon conversion probabilities used to generate oscillation
    spectra that are compared against the null (no-oscillation) baseline
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Instrument providing the observed gamma-ray spectral data of NGC 1275 from
    which the null baseline and irregularity statistics are derived
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:5194b24d4a2a05f984dc71db45d6cf98ad1ea954528b157188e47f33f30f47ba
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# gamma-ray-spectral-null-baseline-comparison

## Summary

Constructs a null (no-oscillation) baseline gamma-ray spectrum from Fermi-LAT observations so ALP-induced spectral irregularities can be detected by comparison rather than by model fitting. This underpins the irregularity-based re-analysis of NGC 1275 that yields coupling constraints around ma ~ 1 neV.

## When to use

Use when re-analyzing Fermi-LAT (or similar) blazar spectra for ALP-photon conversion effects with an irregularity method, where a no-oscillation reference spectrum must be built instead of fitting ALP models to the observed spectrum.

## When NOT to use

- When the goal is to fit the observed spectrum directly with ALP models — this skill deliberately avoids model fitting in favor of irregularity quantification
- When the source spectrum is not from a photon-attenuation-regime observation (ALP conversion constraints here rely on gamma-ray spectra; the article additionally notes PKS 2155-304 must be analyzed for combined constraints, not NGC 1275 alone for its bound)

## Inputs

- Fermi-LAT spectral observations of NGC 1275 (gamma-ray energy spectrum with uncertainties)
- Grid of ALP-photon conversion probability values from alpconv, sampled over ma around 1 neV and gaγ >= 3e-12 GeV^-1

## Outputs

- Null (no-oscillation) baseline spectrum for the observed source
- Exclusion contour in the (ma, gaγ) plane at 95% confidence level
- Excluded coupling bound (e.g., gaγ > 3×10⁻¹² GeV⁻¹ around ma ~ 1 neV)

## How to apply

Load the Fermi-LAT spectral observations of the target source (e.g., NGC 1275) and build the null baseline assuming no ALP-photon conversion. Using alpconv, compute ALP-photon conversion probabilities over a grid of ALP masses around ma ~ 1 neV and coupling values gaγ >= 3e-12 GeV^-1. Rather than fitting the observed spectrum with ALP models, adopt the irregularity method: quantify the irregularity (deviation from smoothness) of the observed spectrum and compare it against the null baseline to test for spectral oscillation. If no oscillation is detected, set exclusion limits in the (ma, gaγ) plane — for NGC 1275 this rules out gaγ > 3×10⁻¹² GeV⁻¹ around ma ~ 1 neV at 95% confidence level. Rationale: fitting spectra with ALP models can overstate sensitivity; the irregularity-based comparison against a null baseline avoids that bias and produced more stringent bounds than previous analyses.

## Related tools

- **alpconv** (Calculates ALP-photon conversion probabilities used to generate oscillation spectra that are compared against the null (no-oscillation) baseline) — github.com/lyf222/alpconv
- **Fermi-LAT** (Instrument providing the observed gamma-ray spectral data of NGC 1275 from which the null baseline and irregularity statistics are derived)

## Evaluation signals

- The derived 95% CL exclusion region must rule out couplings gaγ > 3×10⁻¹² GeV⁻¹ around ALP mass ma ~ 1 neV, matching the published bound
- The null baseline must show no ALP-induced spectral oscillation features; residual irregularity of observed vs. baseline should be consistent with statistical fluctuations
- The exclusion contour should be at least as stringent as the previous (fitting-based) NGC 1275 results, per the reported 'more stringent than previous results' outcome
- Conversion probability grid from alpconv must span the ma ~ 1 neV neighborhood and gaγ >= 3e-12 GeV^-1 to reproduce the reported constraint region

## Limitations

- alpconv provides no usage instructions and code comments remain in Chinese, complicating reuse of the conversion-probability calculations
- The article's discussion section contains only a changelog placeholder, so extended methodological detail beyond abstract/intro is unavailable
- Constraints from a single source are limited; the authors state that combining PKS 2155-304 and more sources is needed for a much wider exclusion region

## Evidence

- [abstract] Null-baseline comparison rationale: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] Exclusion bound reproduced by the null-baseline comparison: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [readme] alpconv role and limitation: "at present no intruction for use provides, chinese character comments remain in the code."
- [intro] Conversion-probability tool grounding: "alpconv calculating alp-photon conversion"
