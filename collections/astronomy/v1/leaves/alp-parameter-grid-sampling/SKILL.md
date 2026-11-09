---
name: alp-parameter-grid-sampling
description: 'Use when reconstructing ALP parameter exclusions from Fermi-LAT spectra
  of NGC 1275-like sources: you need conversion-probability curves P_γ→a(E) on a (g_aγ,
  m_a) grid spanning couplings down to ~3×10⁻¹² GeV⁻¹ and ma ∼ 1 neV.'
when_to_use_negative:
- The analysis goal is spectral-model fitting of the observed Fermi-LAT spectrum rather
  than irregularity-based / grid-sampled conversion-probability evaluation — the paper
  explicitly adopts the irregularity statistic instead of fitting spectra with ALP
  models
- The source environment requires non-negligible JET or IGMF magnetic fields, outside
  the NGC 1275 assumptions of negligible JET/IGMF and 1–10 μG ICMF
- ALP masses far from the neV scale constrained by the NGC 1275 analysis, outside
  the sampled grid span
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: alpconv
  role: Numerically integrates photon–ALP mixing equations through JET/ICMF/GMF/IGMF
    environments to produce conversion-probability-vs-energy curves per (g_aγ, m_a)
    grid point
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Defines the observational energy range over which conversion probabilities
    are computed and validated for the NGC 1275 re-analysis
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:394c07ec35da466c9259bbde8be285ce5bcf4b8cada29d705abb3073bd6460bc
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
yes_rate: 1.0
n_claims: 8
---

# alp-parameter-grid-sampling

## Summary

Sample a grid of axion-like-particle parameters (g_aγ, m_a) around the neV mass scale and compute photon-ALP conversion probability vs. energy with the alpconv package over the Fermi-LAT energy range, for the NGC 1275 line-of-sight magnetic-field environments (negligible JET/IGMF, turbulent ICMF of 1–10 μG, and GMF).

## When to use

Use when reconstructing ALP parameter exclusions from Fermi-LAT spectra of NGC 1275-like sources: you need conversion-probability curves P_γ→a(E) on a (g_aγ, m_a) grid spanning couplings down to ~3×10⁻¹² GeV⁻¹ and ma ∼ 1 neV.

## When NOT to use

- The analysis goal is spectral-model fitting of the observed Fermi-LAT spectrum rather than irregularity-based / grid-sampled conversion-probability evaluation — the paper explicitly adopts the irregularity statistic instead of fitting spectra with ALP models
- The source environment requires non-negligible JET or IGMF magnetic fields, outside the NGC 1275 assumptions of negligible JET/IGMF and 1–10 μG ICMF
- ALP masses far from the neV scale constrained by the NGC 1275 analysis, outside the sampled grid span

## Inputs

- ALP parameter grid points (g_aγ, m_a) spanning GeV⁻¹ couplings and neV-scale masses
- Line-of-sight magnetic-field environment definitions (JET, ICMF 1–10 μG turbulent, GMF, IGMF)
- Fermi-LAT energy range definition
- alpconv package from github.com/lyf222/alpconv

## Outputs

- ALP–photon conversion probability vs. energy arrays for every sampled (g_aγ, m_a) point
- Structured data file storing conversion-probability arrays per parameter set
- Plots of conversion probability versus energy for representative parameter sets

## How to apply

Define the four magnetic-field environments along the propagation path following alpconv conventions: negligible JET field, negligible IGMF, a turbulent ICMF of order 1–10 μG with Gaussian turbulence statistics, and a regular Galactic magnetic field component. Set up the photon–ALP mixing equations and verify numerical integration converges over the Fermi-LAT energy range. Sample the (g_aγ, m_a) grid spanning couplings and neV-scale masses motivated by the NGC 1275 analysis, and for each grid point run alpconv to compute the ALP–photon conversion probability as a function of energy, averaging over multiple realizations of the turbulent ICMF. Serialize the probability-vs-energy arrays per parameter set (e.g., for downstream pseudo-count generation of 500 realizations × 100 pseudo spectra per parameter set).

## Related tools

- **alpconv** (Numerically integrates photon–ALP mixing equations through JET/ICMF/GMF/IGMF environments to produce conversion-probability-vs-energy curves per (g_aγ, m_a) grid point) — github.com/lyf222/alpconv
- **Fermi-LAT** (Defines the observational energy range over which conversion probabilities are computed and validated for the NGC 1275 re-analysis)

## Examples

```
python alpconv_driver.py --gmin 1e-13 --gmax 1e-11 --mmin 0.1neV --mmax 10neV --icmf 1-10uG --energy-range fermi-lat
```

## Evaluation signals

- Every computed probability value lies within the physical range [0, 1]
- Probability arrays are finite and cover the full Fermi-LAT energy range for all sampled (g_aγ, m_a) points
- Curves exhibit the expected energy-dependent oscillatory structure in P_γ→a(E)
- Numerical integration of the photon–ALP mixing equations converges over the Fermi-LAT energy range
- Downstream pseudo-count step finds exactly 50,000 counts cubes per ALP parameter set, confirming each grid point yielded a usable probability array

## Limitations

- No quantitative acceptance threshold is reported by the paper for probability-array QC, so only qualitative checks (range [0,1], finiteness, oscillatory structure) apply
- The alpconv repository provides no usage instructions and code comments remain in Chinese, hindering reuse
- JET and IGMF environments are treated as negligible for the considered ALP parameters and Fermi-LAT energy range; other assumptions may not transfer to other sources
- Turbulent ICMF realizations require averaging over Gaussian turbulence statistics, introducing realization-dependent variance in the probability curves

## Evidence

- [methods] The propagation calculation treats the JET and IGMF environments as negligible for the considered ALP parameters and Fermi-LAT energy range, while the ICMF has a typical field strength in the range of 1 - 10 μG.: [evidence span withheld — anchor status: not_found]
- [methods] Sample a grid of ALP parameter points in (g_aγ, m_a) spanning the couplings and neV-scale masses constrained by the NGC 1275 analysis.: [evidence span withheld — anchor status: not_found]
- [abstract] rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level"
- [intro] alpconv calculating alp-photon conversion: "alpconv calculating alp-photon conversion"
- [readme] (at present no intruction for use provides, chinese character comments remain in the code.): "(at present no intruction for use provides, chinese character comments remain in the code.)"
