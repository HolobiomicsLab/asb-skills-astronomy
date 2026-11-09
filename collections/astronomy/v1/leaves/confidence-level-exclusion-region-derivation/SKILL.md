---
name: confidence-level-exclusion-region-derivation
description: Use when Fermi-LAT spectral data of a blazar sightline (e.g. NGC 1275,
  PKS 2155-304) must be scanned over (ma, gaγ) to build a confidence-level exclusion
  region for axion-like particle couplings.
when_to_use_negative:
- The analysis goal is a ALP signal detection/fitting of an observed spectrum with
  ALP models rather than an irregularity-based non-fitting exclusion limit
- Only a single grid point or a fixed (ma, gaγ) value is of interest — a full coupling–mass
  grid scan is required to delimit a region
- The input spectrum is not from a Fermi-LAT blazar observation with a known ALP-relevant
  sightline (conversion along extragalactic/Galactic magnetic fields)
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: alpconv
  role: Computes the ALP-photon conversion probabilities along the source sightline
    over the (ma, gaγ) grid, feeding the irregularity statistic used to build the
    exclusion region
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Provides the observed gamma-ray spectral data of NGC 1275 (and PKS 2155-304)
    that the irregularity statistic is evaluated against
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:88052c8d9476736220f5ea78e0cf3773672b823fb18ea0871587256dcbbc0452
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# confidence-level-exclusion-region-derivation

## Summary

Derives 95% CL exclusion regions in the (ma, gaγ) ALP parameter plane by scanning a coupling–mass grid and applying an irregularity-based (non-fitting) statistic to Fermi-LAT gamma-ray spectra, using alpconv to compute ALP-photon conversion probabilities.

## When to use

Use when Fermi-LAT spectral data of a blazar sightline (e.g. NGC 1275, PKS 2155-304) must be scanned over (ma, gaγ) to build a confidence-level exclusion region for axion-like particle couplings.

## When NOT to use

- The analysis goal is a ALP signal detection/fitting of an observed spectrum with ALP models rather than an irregularity-based non-fitting exclusion limit
- Only a single grid point or a fixed (ma, gaγ) value is of interest — a full coupling–mass grid scan is required to delimit a region
- The input spectrum is not from a Fermi-LAT blazar observation with a known ALP-relevant sightline (conversion along extragalactic/Galactic magnetic fields)

## Inputs

- Fermi-LAT observed gamma-ray spectrum of the target source (e.g. NGC 1275)
- Grid of ALP mass (ma) and coupling (gaγ) values, e.g. ma around 1 neV, gaγ >= 3e-12 GeV^-1
- Null no-oscillation baseline spectrum
- ALP-photon conversion probabilities along the source sightline (from alpconv)

## Outputs

- 95% CL exclusion region in the (ma, gaγ) plane (contour figure/table, e.g. combined_exclusion_region.csv)
- Tabulated single-source vs. combined coupling bounds (e.g. combined_vs_single_bounds.csv)
- Excluded coupling bound statement (e.g. gaγ > 3×10⁻¹² GeV⁻¹ around ma ~ 1 neV)

## How to apply

Load the observed Fermi-LAT spectrum of the source and compute ALP-photon conversion probabilities with alpconv over a grid of ALP masses around ma ~ 1 neV and couplings gaγ >= 3e-12 GeV^-1. Build null (no-oscillation) baseline spectra for comparison, then apply the irregularity-based method — which calculates the irregularity of the spectrum instead of fitting the observed spectrum with ALP models — to compare observed vs. model spectra on each grid point. Convert the irregularity statistic into an exclusion decision using the 95% confidence level threshold to delimit the excluded region in the (ma, gaγ) plane. For combined constraints, evaluate the irregularity statistic jointly over multiple sources' sightlines (e.g. NGC 1275 + PKS 2155-304) so the combined exclusion region strictly contains each single-source region. Save the exclusion-contour grid and tabulate the excluded coupling bound (here gaγ > 3×10⁻¹² GeV⁻¹ around ma ~ 1 neV).

## Related tools

- **alpconv** (Computes the ALP-photon conversion probabilities along the source sightline over the (ma, gaγ) grid, feeding the irregularity statistic used to build the exclusion region) — github.com/lyf222/alpconv
- **Fermi-LAT** (Provides the observed gamma-ray spectral data of NGC 1275 (and PKS 2155-304) that the irregularity statistic is evaluated against)

## Evaluation signals

- The reproduced exclusion region must rule out couplings gaγ > 3×10⁻¹² GeV⁻¹ around ALP mass ma ~ 1 neV at 95% confidence level
- The combined (multi-source) exclusion region must strictly enlarge/contain both single-source regions
- No spurious spectral oscillation should be claimed for NGC 1275 — the expected result is 'no spectral oscillation'
- The exclusion grid must cover a coupling–mass scan with the 95% CL threshold applied consistently across grid points
- Result should be at least as stringent as previous (fitting-based) results for the same source

## Limitations

- No usage instructions are provided for alpconv; code comments remain in Chinese, so interpreting the tool requires the referenced paper (arXiv:1802.08420)
- The irregularity method is non-fitting: it tests spectral irregularity rather than model fitting, so its interpretation differs from likelihood-based ALP analyses
- Wider exclusion regions require additional sources; a single sightline limits the constrained parameter-space area

## Evidence

- [abstract] 95% CL exclusion bound: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [abstract] irregularity method definition: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] multi-source combination: "We also show that the constraints can be further improved by combining the observation of PKS 2155-304."
- [intro] conversion computation tool: "alpconv calculating alp-photon conversion"
- [readme] tool usability limitation: "at present no intruction for use provides, chinese character comments remain in the code."
