---
name: null-hypothesis-baseline-simulation
description: Use when testing a gamma-ray spectrum (e.g., Fermi-LAT observations of
  NGC 1275 or PKS 2155-304) for ALP-photon conversion features by comparing an irregularity
  statistic against a simulated no-ALP null distribution, rather than fitting ALP
  models to the spectrum.
when_to_use_negative:
- When the goal is a likelihood or model fit of ALP parameters directly to the observed
  spectrum — this skill's method explicitly replaces fitting the observed spectrum
  with ALP models by calculating the spectrum's irregularity.
- When the observed spectrum already shows a confirmed spectral oscillation signal
  to characterize rather than to exclude — the method is designed to flag exclusion
  (the article found no spectral oscillation for NGC 1275).
- When the ALP-photon conversion probability cannot be computed for the source's magnetic-field
  environment with the available tools (e.g., no alpconv-compatible setup).
edam_operation: http://edamontology.org/operation_3578
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: alpconv
  role: Computes ALP-photon conversion probabilities over the gaγ–ma grid used to
    generate the oscillated spectra whose irregularity is compared to the null baseline
  repo: https://github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Source of the public photon event data from which the binned gamma-ray spectrum
    and its irregularity statistic are derived
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:4cc06c1491eabc88670acb27707f6a7416bf69d671b3c750a4c051908d4b2a7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lyf222/alpconv
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_3578
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# null-hypothesis-baseline-simulation

## Summary

Builds null (no-ALP) baseline distributions of a spectral irregularity statistic from Fermi-LAT gamma-ray spectra, so ALP-induced spectral oscillations can be flagged without fitting ALP models directly. Applied in the NGC 1275 and PKS 2155-304 re-analyses to exclude couplings gaγ > 3×10⁻¹² GeV⁻¹ near ma ∼ 1 neV at 95% confidence.

## When to use

Use when testing a gamma-ray spectrum (e.g., Fermi-LAT observations of NGC 1275 or PKS 2155-304) for ALP-photon conversion features by comparing an irregularity statistic against a simulated no-ALP null distribution, rather than fitting ALP models to the spectrum.

## When NOT to use

- When the goal is a likelihood or model fit of ALP parameters directly to the observed spectrum — this skill's method explicitly replaces fitting the observed spectrum with ALP models by calculating the spectrum's irregularity.
- When the observed spectrum already shows a confirmed spectral oscillation signal to characterize rather than to exclude — the method is designed to flag exclusion (the article found no spectral oscillation for NGC 1275).
- When the ALP-photon conversion probability cannot be computed for the source's magnetic-field environment with the available tools (e.g., no alpconv-compatible setup).

## Inputs

- Binned Fermi-LAT photon event data / extracted spectral energy distribution for the target source (e.g., 12-year LAT observations of PKS 2155-304, 4FGL J2158.8-3013, or NGC 1275)
- Grid of ALP coupling values gaγ and ALP masses ma over which conversion probabilities are evaluated

## Outputs

- Null (no-ALP) baseline distributions of the spectral irregularity statistic
- Per-parameter-set exclusion verdicts at 95% confidence level (exclusion-region table)
- Exclusion-region figure in the gaγ–ma plane

## How to apply

Extract a binned Fermi-LAT photon spectrum (e.g., 12 years of public LAT events for 4FGL J2158.8-3013). For each grid point of ALP coupling gaγ and mass ma, compute the ALP-photon conversion probability with the alpconv package, then calculate the irregularity of the resulting oscillated spectrum instead of fitting the observed spectrum with ALP models. Generate null-baseline distributions of the irregularity statistic under the no-ALP hypothesis to define the reference against which spectral irregularities are judged. Flag each (gaγ, ma) parameter set as excluded when its irregularity falls outside the null baseline at the 95% confidence level, and assemble the exclusion-region table and figure. The rationale is that an irregularity-based comparison avoids model-fitting degeneracies and, per the article, yields bounds more stringent than previous ALP-fitting results.

## Related tools

- **alpconv** (Computes ALP-photon conversion probabilities over the gaγ–ma grid used to generate the oscillated spectra whose irregularity is compared to the null baseline) — https://github.com/lyf222/alpconv
- **Fermi-LAT** (Source of the public photon event data from which the binned gamma-ray spectrum and its irregularity statistic are derived)

## Evaluation signals

- The null-baseline irregularity distributions should be reproducible under repeated no-ALP simulation with stable percentiles defining the 95% confidence threshold.
- Parameter sets flagged excluded should satisfy: their irregularity lies beyond the null baseline at 95% CL; excluded region should reproduce gaγ > 3×10⁻¹² GeV⁻¹ near ma ∼ 1 neV for NGC 1275.
- The exclusion-region table should contain a per-(gaγ, ma) verdict for every grid point with no gaps.
- Applying the same procedure to PKS 2155-304 should exclude the same ALP-induced spectral structure as the NGC 1275 analysis; combining both sources should widen the exclusion region.
- Conversion probabilities from alpconv should be physically bounded (0 ≤ P(a→γ) ≤ 1) across the grid.

## Limitations

- The alpconv repository provides no usage instructions and its code comments remain in Chinese, complicating reuse of the conversion-probability calculations.
- The method judges spectral irregularity against a simulated null rather than fitting; its power depends on the fidelity of the null-baseline distributions and the binned spectral extraction.
- Constraints improve only when multiple sources are combined; a single source yields a limited exclusion region (wider regions require more sources taken into account).

## Evidence

- [abstract] irregularity method replaces ALP model fitting: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] 95% CL exclusion threshold and coupling bound: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [abstract] multi-source widening of exclusion region: "We suggest that with more sources taken into account, we could obtain a much wider exclusion region."
- [abstract] no-oscillation null result: "With the newly used method, we find no spectral oscillation for the NGC 1275"
- [readme] alpconv tool limitations: "(at present no intruction for use provides, chinese character comments remain in the code.)"
