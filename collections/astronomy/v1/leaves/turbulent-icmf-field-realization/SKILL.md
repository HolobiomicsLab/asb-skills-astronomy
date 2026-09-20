---
name: turbulent-icmf-field-realization
description: Use when computing photon–ALP/axion-like-particle conversion probabilities
  along a line of sight that crosses a galaxy cluster, where the ICMF must be modeled
  as a turbulent 1–10 μG field with Gaussian statistics rather than a uniform field.
when_to_use_negative:
- The source is outside a galaxy cluster so no ICMF segment exists along the line
  of sight (the paper treats JET and IGMF contributions as negligible, leaving only
  GMF conversion)
- The analysis requires fitting the observed Fermi-LAT spectrum with ALP models rather
  than propagating through magnetic-field realizations (this paper instead calculates
  spectral irregularity)
tools:
- name: alpconv
  role: Computes the photon–ALP and ALP–photon conversion probabilities through the
    turbulent ICMF, GMF, and negligible JET/IGMF environments over the Fermi-LAT energy
    range
  repo: https://github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Defines the observational energy range over which conversion probabilities
    are calculated and validated for the NGC 1275 source
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:28ccbf08b59663af5632d0c7c591a6205d3a595f6e09fd803353196bf8f0f05e
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# turbulent-icmf-field-realization

## Summary

Generates turbulent intracluster magnetic-field (ICMF) realizations of order 1–10 μG with Gaussian turbulence statistics, used as the dominant photon–ALP conversion medium between NGC 1275 and the Galactic field, within the alpconv propagation framework.

## When to use

Use when computing photon–ALP/axion-like-particle conversion probabilities along a line of sight that crosses a galaxy cluster, where the ICMF must be modeled as a turbulent 1–10 μG field with Gaussian statistics rather than a uniform field.

## When NOT to use

- The source is outside a galaxy cluster so no ICMF segment exists along the line of sight (the paper treats JET and IGMF contributions as negligible, leaving only GMF conversion)
- The analysis requires fitting the observed Fermi-LAT spectrum with ALP models rather than propagating through magnetic-field realizations (this paper instead calculates spectral irregularity)

## Inputs

- ALP parameter grid of coupling g_aγ (GeV⁻¹) and mass m_a (neV-scale)
- ICMF field strength range 1–10 μG and Gaussian turbulence statistics assumptions
- Fermi-LAT energy range for NGC 1275 observations

## Outputs

- Photon–ALP conversion probability as a function of energy, averaged over turbulent ICMF realizations
- Structured data files storing probability-vs-energy arrays for every sampled (g_aγ, m_a) point
- Figures of conversion probability versus energy for representative parameter sets

## How to apply

Define the four magnetic-field environments along the source-to-observer path (negligible JET, negligible IGMF, turbulent ICMF of 1–10 μG, regular GMF), consistent with the alpconv package conventions, since the paper finds the JET and IGMF contributions negligible for the considered ALP parameters and Fermi-LAT energy range. Sample realizations of the ICMF assuming Gaussian turbulence statistics, set up the photon–ALP mixing equations, and integrate them numerically over the Fermi-LAT energy range for each ALP parameter point (g_aγ, m_a) on a grid spanning couplings near 3×10⁻¹² GeV⁻¹ and neV-scale masses near 1 neV. Run multiple ICMF realizations for each parameter point and average the conversion probability over the Gaussian turbulence. Store probability-vs-energy arrays per parameter set and validate that probabilities remain in [0, 1] and are finite across the full energy range.

## Related tools

- **alpconv** (Computes the photon–ALP and ALP–photon conversion probabilities through the turbulent ICMF, GMF, and negligible JET/IGMF environments over the Fermi-LAT energy range) — https://github.com/lyf222/alpconv
- **Fermi-LAT** (Defines the observational energy range over which conversion probabilities are calculated and validated for the NGC 1275 source)

## Evaluation signals

- Every computed conversion probability value lies within the physical range [0, 1]
- Probability-vs-energy arrays are finite and cover the full Fermi-LAT energy range for all sampled (g_aγ, m_a) points
- Curves exhibit the expected energy-dependent oscillatory structure
- Numerical integration of the mixing equations converges over the Fermi-LAT energy range
- Averaging over multiple ICMF realizations smooths realization-specific fluctuations, consistent with Gaussian turbulence statistics

## Limitations

- No usage instructions are provided; the alpconv code comments remain in Chinese
- The turbulent-field result is realization-dependent, so multiple ICMF realizations must be run and averaged; a single realization is not representative
- The ICMF is the only non-negligible cluster-scale medium in this setup: JET and IGMF contributions are treated as negligible for the considered ALP parameters and Fermi-LAT energy range

## Evidence

- [abstract] rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level"
- [readme] no usage instructions / Chinese comments: "(at present no intruction for use provides, chinese character comments remain in the code.)"
- [readme] alpconv purpose: "calculating alp-photon conversion"
- [intro] PKS 2155-304 reference bounds: "New bounds on axionlike particles from the fermi large area telescope observation of PKS 2155-304"
