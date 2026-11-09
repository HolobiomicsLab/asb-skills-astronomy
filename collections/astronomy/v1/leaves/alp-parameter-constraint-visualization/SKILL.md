---
name: alp-parameter-constraint-visualization
description: Use when re-analyzing Fermi-LAT observations (e.g. NGC 1275) to constrain
  ALP-photon coupling gaγ as a function of ALP mass ma, or when reproducing a 95%
  CL exclusion contour such as gaγ > 3×10⁻¹² GeV⁻¹ at ma ∼ 1 neV.
when_to_use_negative:
- When the analysis goal is to fit the observed spectrum with ALP models rather than
  compute spectral irregularity — the paper explicitly adopts the irregularity method
  'instead of fitting the observed spectrum with ALP models'.
- When the target source is not a gamma-ray point source with a published Fermi-LAT
  spectrum suitable for ALP-photon oscillation searches (e.g. no coherent magnetic-field
  model available via alpconv).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: alpconv
  role: Calculates ALP-photon conversion probabilities over the (ma, gaγ) grid that
    feed the irregularity-based exclusion analysis of Fermi-LAT spectra
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Instrument providing the gamma-ray observation of NGC 1275 whose spectrum
    is re-analyzed to search for ALP effects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:eb76b9c40d130354fd8d91947d8f8639392adaf40932ada8decfd9e17c896a34
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# alp-parameter-constraint-visualization

## Summary

Method for deriving and visualizing axion-like particle (ALP) exclusion regions in the (ma, gaγ) plane from Fermi-LAT gamma-ray spectra, using ALP-photon conversion probabilities (alpconv) and an irregularity-based spectral analysis instead of direct ALP model fitting.

## When to use

Use when re-analyzing Fermi-LAT observations (e.g. NGC 1275) to constrain ALP-photon coupling gaγ as a function of ALP mass ma, or when reproducing a 95% CL exclusion contour such as gaγ > 3×10⁻¹² GeV⁻¹ at ma ∼ 1 neV.

## When NOT to use

- When the analysis goal is to fit the observed spectrum with ALP models rather than compute spectral irregularity — the paper explicitly adopts the irregularity method 'instead of fitting the observed spectrum with ALP models'.
- When the target source is not a gamma-ray point source with a published Fermi-LAT spectrum suitable for ALP-photon oscillation searches (e.g. no coherent magnetic-field model available via alpconv).

## Inputs

- Fermi-LAT spectral observations of NGC 1275 (binned gamma-ray spectrum with energy-flux points)
- Grid of ALP masses ma (centered around ~1 neV) and coupling values gaγ (>= 3e-12 GeV^-1)
- Null (no-oscillation) baseline spectral model for the source

## Outputs

- 95% confidence level exclusion region in the (ma, gaγ) parameter plane (figure and/or table)
- Recorded excluded coupling bound (e.g. gaγ > 3×10⁻¹² GeV⁻¹ at ma ∼ 1 neV)
- Statement on presence/absence of spectral ALP oscillation for the source

## How to apply

Load the Fermi-LAT spectral observations of the target source (e.g. NGC 1275) and compute ALP-photon conversion probabilities with the alpconv package over a grid of ALP masses around ma ∼ 1 neV and couplings gaγ ≥ 3×10⁻¹² GeV⁻¹. Build null (no-oscillation) baseline spectra for comparison. Instead of fitting the observed spectrum with ALP models, apply the irregularity method, which calculates the irregularity of the spectrum to compare observed vs. ALP-model spectra. Scan the (ma, gaγ) grid and mark the region where the irregularity signal is excluded at 95% confidence level; for NGC 1275 the expected result is that gaγ > 3×10⁻¹² GeV⁻¹ around ma ∼ 1 neV is ruled out. Save the exclusion-contour figure/table and record the excluded coupling bound; constraints can be further improved by combining multiple sources such as PKS 2155-304.

## Related tools

- **alpconv** (Calculates ALP-photon conversion probabilities over the (ma, gaγ) grid that feed the irregularity-based exclusion analysis of Fermi-LAT spectra) — github.com/lyf222/alpconv
- **Fermi-LAT** (Instrument providing the gamma-ray observation of NGC 1275 whose spectrum is re-analyzed to search for ALP effects)

## Evaluation signals

- Reproduced exclusion region rules out couplings gaγ > 3×10⁻¹² GeV⁻¹ around ALP mass ma ∼ 1 neV at 95% confidence level, matching the paper's reported bound.
- The reproduced bound should be at least as stringent as previous (fitting-based) results for NGC 1275.
- The null (no-oscillation) baseline spectrum shows no irregularity signal inconsistent with the observed Fermi-LAT spectrum.
- Result states no spectral oscillation detected for NGC 1275, consistent with the paper's finding.
- Combining PKS 2155-304 with NGC 1275 should tighten (improve) the constraints rather than weaken them.

## Limitations

- The alpconv repository provides no usage instructions; code comments remain in Chinese, increasing integration effort.
- The exclusion region is limited to the mass/coupling grid analyzed (around ma ∼ 1 neV, gaγ >= 3×10⁻¹² GeV⁻¹); a much wider exclusion region would require taking more sources into account.
- Constraints from a single source (NGC 1275) are weaker than combined-source analyses; combining PKS 2155-304 is needed for further improvement.

## Evidence

- [abstract] irregularity method definition: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] 95% CL exclusion bound: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [abstract] no oscillation detected: "With the newly used method, we find no spectral oscillation for the NGC 1275"
- [intro] alpconv tool role: "alpconv calculating alp-photon conversion"
- [readme] alpconv repo limitations: "(at present no intruction for use provides, chinese character comments remain in the code.)"
