---
name: oscillatory-spectrum-visualization
description: Use when computed ALP–photon conversion probabilities over the Fermi-LAT
  energy band must be plotted as probability-vs-energy curves to inspect oscillatory
  structure for sampled (g_aγ, m_a) points.
when_to_use_negative:
- When the goal is a quantitative ALP parameter exclusion contour rather than qualitative
  inspection of oscillatory spectra — the paper's irregularity-based method, not probability-curve
  plots, delivers the confidence-level constraints.
- When propagation environments outside the paper's scope (non-negligible JET or IGMF
  fields, or masses far from ~1 neV) dominate the mixing, since alpconv settings here
  assume negligible JET/IGMF.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: alpconv
  role: Computes the ALP–photon conversion probability as a function of energy through
    ICMF and GMF environments, whose oscillatory energy dependence is then plotted
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Source of the NGC 1275 gamma-ray observation defining the energy range over
    which oscillatory spectra are evaluated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:9cfc83541926e66568867f1801d1233a6c8abb24461bcf03af34616f8208ef56
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# oscillatory-spectrum-visualization

## Summary

Visualize photon–ALP conversion probability as an energy-dependent oscillatory spectrum (probability vs energy) computed with alpconv across ICMF and GMF magnetic-field environments, as used in the Fermi-LAT NGC 1275 ALP analysis to check for spectral irregularity signals.

## When to use

Use when computed ALP–photon conversion probabilities over the Fermi-LAT energy band must be plotted as probability-vs-energy curves to inspect oscillatory structure for sampled (g_aγ, m_a) points.

## When NOT to use

- When the goal is a quantitative ALP parameter exclusion contour rather than qualitative inspection of oscillatory spectra — the paper's irregularity-based method, not probability-curve plots, delivers the confidence-level constraints.
- When propagation environments outside the paper's scope (non-negligible JET or IGMF fields, or masses far from ~1 neV) dominate the mixing, since alpconv settings here assume negligible JET/IGMF.

## Inputs

- Conversion-probability-vs-energy arrays for each sampled (g_aγ, m_a) point (structured data file)
- Fermi-LAT energy range for NGC 1275 observations

## Outputs

- Conversion probability vs energy plots for representative ALP parameter sets (saved figure files)
- Structured data file of probability arrays for all sampled parameter points

## How to apply

First compute conversion probabilities with alpconv for a grid of ALP parameters in (g_aγ, m_a) near the neV-scale masses constrained by the NGC 1275 analysis, treating JET and IGMF environments as negligible and propagating through a turbulent ICMF of 1–10 μG (Gaussian turbulence, averaged over multiple realizations) plus a regular GMF component. Then store each conversion-probability-vs-energy array for the full Fermi-LAT energy range in a structured data file before plotting. Plot probability versus energy for representative parameter sets so the energy-dependent oscillatory structure is visible; this visualization is the qualitative basis for judging whether spectral oscillations (irregularity) appear in the spectrum. Save the resulting figure files per parameter set. Validate that every plotted probability lies in [0, 1] and that curves are finite and cover the full energy range.

## Related tools

- **alpconv** (Computes the ALP–photon conversion probability as a function of energy through ICMF and GMF environments, whose oscillatory energy dependence is then plotted) — github.com/lyf222/alpconv
- **Fermi-LAT** (Source of the NGC 1275 gamma-ray observation defining the energy range over which oscillatory spectra are evaluated)

## Evaluation signals

- Every computed conversion probability lies within the physical range [0, 1]
- Probability arrays are finite and cover the full Fermi-LAT energy range for all sampled (g_aγ, m_a) points
- Plotted curves exhibit the expected energy-dependent oscillatory structure
- Multiple turbulent ICMF realizations are averaged consistently (Gaussian turbulence statistics) so curves are stable
- Figure files are saved for every representative parameter set plotted

## Limitations

- alpconv currently provides no instructions for use and Chinese-character comments remain in the code
- The paper reports no quantitative acceptance threshold for the oscillatory visualization; only qualitative QC checks apply
- JET and IGMF environments are treated as negligible for the considered ALP parameters and Fermi-LAT energy range, so the visualization is only valid under that approximation

## Evidence

- [full_text] workflow environments: [evidence span withheld — anchor status: not_found]
- [full_text] plot and QC: [evidence span withheld — anchor status: not_found]
- [full_text] QC threshold: [evidence span withheld — anchor status: not_found]
- [abstract] no oscillation finding: "With the newly used method, we find no spectral oscillation for the NGC 1275"
- [readme] tool role: "calculating alp-photon conversion"
