---
name: regular-gmf-component-modeling
description: Use when computing photon-ALP conversion probabilities for an extragalactic
  source like NGC 1275 where the propagation chain is dominated by the cluster magnetic
  field (ICMF, ~1–10 μG, turbulent) plus a regular GMF component.
when_to_use_negative:
- When the analysis target requires modeling conversion inside the source jet or the
  intergalactic magnetic field — this paper's setup explicitly treats JET and IGMF
  contributions as negligible for the considered ALP parameters and Fermi-LAT energy
  range.
- When the goal is the spectral-analysis step (irregularity-based method or ALP-model
  spectrum fitting on Fermi-LAT data) rather than the propagation/conversion-probability
  calculation.
- When a different, non-regular GMF model is required — this skill follows the alpconv
  package's regular-GMF conventions specifically.
tools:
- name: alpconv
  role: Numerically integrates the photon–ALP mixing equations through the GMF/ICMF
    environments and computes the conversion probability as a function of energy for
    each (g_aγ, m_a) grid point.
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Defines the observational energy range over which the propagation calculation
    and conversion probabilities must be computed and validated.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:9e9cc999e1f157c99119b3f7a487436c55dbcbc9d2816f91e941b9526a17ae2c
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# regular-gmf-component-modeling

## Summary

Sets up the regular Galactic magnetic field (GMF) component of the photon–ALP propagation calculation, alongside the turbulent ICMF (1–10 μG), with the JET and IGMF environments treated as negligible for NGC 1275 parameters in the Fermi-LAT energy range.

## When to use

Use when computing photon-ALP conversion probabilities for an extragalactic source like NGC 1275 where the propagation chain is dominated by the cluster magnetic field (ICMF, ~1–10 μG, turbulent) plus a regular GMF component.

## When NOT to use

- When the analysis target requires modeling conversion inside the source jet or the intergalactic magnetic field — this paper's setup explicitly treats JET and IGMF contributions as negligible for the considered ALP parameters and Fermi-LAT energy range.
- When the goal is the spectral-analysis step (irregularity-based method or ALP-model spectrum fitting on Fermi-LAT data) rather than the propagation/conversion-probability calculation.
- When a different, non-regular GMF model is required — this skill follows the alpconv package's regular-GMF conventions specifically.

## Inputs

- ALP parameter grid points (g_aγ, m_a) spanning couplings and neV-scale masses constrained by the NGC 1275 analysis
- Magnetic-field environment definitions: regular GMF component, turbulent ICMF (~1–10 μG) with Gaussian turbulence statistics, negligible JET and IGMF
- Fermi-LAT energy range grid
- alpconv package (github.com/lyf222/alpconv)

## Outputs

- ALP-photon conversion-probability-vs-energy arrays for every sampled (g_aγ, m_a) point, stored in a structured data file
- Figures of conversion probability versus energy for representative parameter sets

## How to apply

Define the four magnetic-field environments following the alpconv package conventions: a negligible JET field, a negligible IGMF, a turbulent ICMF of order 1–10 microgauss with Gaussian turbulence statistics, and a regular Galactic magnetic field (GMF) component. Set up the photon–axion-like-particle mixing equations for propagation through these environments and verify the numerical integration converges over the Fermi-LAT energy range. For each sampled ALP parameter point (g_aγ, m_a) on the neV-scale mass grid, run alpconv — using multiple realizations of the turbulent ICMF where needed and averaging over the Gaussian turbulence — to obtain the conversion probability as a function of energy. Store the probability-vs-energy arrays per parameter set, and validate that all values lie in [0, 1], arrays are finite over the full energy range, and curves show the expected energy-dependent oscillatory structure.

## Related tools

- **alpconv** (Numerically integrates the photon–ALP mixing equations through the GMF/ICMF environments and computes the conversion probability as a function of energy for each (g_aγ, m_a) grid point.) — github.com/lyf222/alpconv
- **Fermi-LAT** (Defines the observational energy range over which the propagation calculation and conversion probabilities must be computed and validated.)

## Evaluation signals

- Every computed conversion probability lies within the physical range [0, 1].
- All probability-vs-energy arrays are finite and cover the full Fermi-LAT energy range for every sampled (g_aγ, m_a) point.
- Curves exhibit the expected energy-dependent oscillatory structure.
- Numerical integration over the propagation environments converges over the Fermi-LAT energy range.
- Turbulent ICMF contributions are averaged over multiple Gaussian-turbulence realizations rather than a single realization.

## Limitations

- The paper reports no quantitative acceptance threshold for the conversion-probability QC; only qualitative checks (range [0, 1], finiteness, oscillatory structure) apply.
- alpconv provides no usage instructions and code comments remain in Chinese.
- JET and IGMF environments are treated as negligible, so the skill is not valid for source parameters where those environments matter.

## Evidence

- [intro] alpconv conventions: "alpconv calculating alp-photon conversion"
- [abstract] Fermi-LAT energy range: "we re-analyze the Fermi-LAT observation of NGC 1275 to search for axion-like particle (ALP) effects"
- [readme] no usage instructions: "(at present no intruction for use provides, chinese character comments remain in the code.)"
- [readme] reference paper for alpconv: "please see the below paper for detailed references."
