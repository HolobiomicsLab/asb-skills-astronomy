---
name: alp-photon-conversion-probability-calculation
description: Use when re-analyzing Fermi-LAT spectra of AGN (e.g., NGC 1275, PKS 2155-304)
  to search for ALP-induced spectral oscillations and need energy-dependent photon–ALP
  conversion probabilities across a (g_aγ, m_a) parameter grid.
when_to_use_negative:
- The goal is fitting the observed spectrum with ALP models — this paper's method
  instead calculates the irregularity of the spectrum, and alpconv only supplies the
  conversion-probability ingredient
- The propagation environment or ALP mass/coupling range differs substantially from
  the Fermi-LAT neV-scale, g_aγ ≳ 3×10⁻¹² GeV⁻¹ regime for which JET/IGMF fields were
  found negligible
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: alpconv
  role: Calculates ALP–photon conversion probabilities as a function of energy over
    the sampled (g_aγ, m_a) grid, supporting turbulent and regular ICMF/GMF field
    configurations
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Source of the gamma-ray spectra (NGC 1275, PKS 2155-304) whose energy range
    defines the domain over which conversion probabilities are computed
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:fc54c137a7aa57d74131043cfa77e07762b9274e7083ceecc55f6bf85227b666
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# alp-photon-conversion-probability-calculation

## Summary

Computes axion-like-particle (ALP)–photon conversion probabilities along astrophysical sightlines using the alpconv package, modeling JET, ICMF, IGMF, and GMF magnetic-field environments over the Fermi-LAT energy range for ALP parameter grids in (g_aγ, m_a).

## When to use

Use when re-analyzing Fermi-LAT spectra of AGN (e.g., NGC 1275, PKS 2155-304) to search for ALP-induced spectral oscillations and need energy-dependent photon–ALP conversion probabilities across a (g_aγ, m_a) parameter grid.

## When NOT to use

- The goal is fitting the observed spectrum with ALP models — this paper's method instead calculates the irregularity of the spectrum, and alpconv only supplies the conversion-probability ingredient
- The propagation environment or ALP mass/coupling range differs substantially from the Fermi-LAT neV-scale, g_aγ ≳ 3×10⁻¹² GeV⁻¹ regime for which JET/IGMF fields were found negligible

## Inputs

- ALP parameter grid of couplings g_aγ (GeV⁻¹) and masses m_a (neV scale)
- Magnetic-field environment configuration (turbulent ICMF 1–10 μG, regular GMF, negligible JET/IGMF)
- Fermi-LAT energy range for which conversion probabilities are evaluated

## Outputs

- ALP–photon conversion probability as a function of energy for each sampled (g_aγ, m_a) point
- Structured data file of probability-vs-energy arrays for all sampled parameter sets
- Plots of conversion probability versus energy for representative parameter sets

## How to apply

Define the four propagation environments following alpconv conventions: negligible JET and IGMF fields for the considered ALP parameters, a turbulent ICMF of 1–10 μG with Gaussian turbulence statistics, and a regular GMF component. Set up the photon–ALP mixing equations, verify numerical convergence over the Fermi-LAT energy range, and sample a grid of ALP parameter points in (g_aγ, m_a) spanning neV-scale masses and couplings near g_aγ ~ 3×10⁻¹² GeV⁻¹. For each grid point, run alpconv to obtain conversion probability vs. energy, averaging over multiple realizations of the turbulent ICMF; optionally switch to a regular ICMF/GMF configuration as a robustness variant. Store probability-vs-energy arrays per parameter set for downstream irregularity-based exclusion analysis.

## Related tools

- **alpconv** (Calculates ALP–photon conversion probabilities as a function of energy over the sampled (g_aγ, m_a) grid, supporting turbulent and regular ICMF/GMF field configurations) — github.com/lyf222/alpconv
- **Fermi-LAT** (Source of the gamma-ray spectra (NGC 1275, PKS 2155-304) whose energy range defines the domain over which conversion probabilities are computed)

## Evaluation signals

- Every computed probability value lies within the physical range [0, 1]
- Probability arrays are finite and cover the full Fermi-LAT energy range for all sampled (g_aγ, m_a) points
- Curves exhibit the expected energy-dependent oscillatory structure
- Downstream exclusion analysis reproduces ruling out g_aγ > 3×10⁻¹² GeV⁻¹ around m_a ∼ 1 neV at 95% confidence level
- Numerical integration of the mixing equations converges over the Fermi-LAT energy range

## Limitations

- The alpconv repository provides no usage instructions and code comments remain in Chinese, per the README
- The paper reports no quantitative acceptance threshold for the conversion-probability QC step, so validation is a qualitative check
- Turbulent ICMF realizations are stochastic; multiple realizations averaged over Gaussian turbulence are needed for stable results

## Evidence

- [abstract] propagation environments: "we re-analyze the Fermi-LAT observation of NGC 1275 to search for axion-like particle (ALP) effects and constrain ALP parameters"
- [intro] alpconv tool: "alpconv calculating alp-photon conversion"
- [readme] README caveat: "(at present no intruction for use provides, chinese character comments remain in the code.)"
- [abstract] exclusion bound: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [abstract] analysis method: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
