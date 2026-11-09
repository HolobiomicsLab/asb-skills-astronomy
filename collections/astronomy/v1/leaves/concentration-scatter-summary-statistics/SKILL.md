---
name: concentration-scatter-summary-statistics
description: Use when you have per-cluster posterior samples of parametric mass-model
  parameters (c200, M200) and need population-level statistics (mean concentration,
  scatter) and deviation thresholds to validate a mass model.
when_to_use_negative:
- When individual cluster profile diversity is the target rather than population-level
  averages — the article notes more shape diversity than NFW can capture, which summary
  statistics mask
- When the input mass profiles are parametric rather than nonparametric reconstructions,
  since the <10% deviation criterion is defined against the nonparametric log-normal-mixture
  profiles
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: hydromass
  role: Provides the parametric (NFW, Einasto) reconstruction with PyMC3/HMC whose
    per-cluster posteriors feed the aggregation; invoked via hydromass.mhyd
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Backend for Hamiltonian Monte Carlo sampling of the parametric mass-model
    posteriors
- name: numpy
  role: Computes means, scatter, and fractional deviations over posterior samples
    and radial bins
- name: scipy
  role: Dependency of hydromass used for numerical routines in profile evaluation
- name: astropy
  role: Cosmology and unit handling when expressing c200, M200, and R200
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:83023762ac93b2fc0d699ed46cbb087829d1cec217c5fa4c3dc852b872880b6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/domeckert/hydromass
  tools:
  - astropy
  - hydromass
  - numpy
  - pymc3
  - scipy
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# concentration-scatter-summary-statistics

## Summary

Aggregates per-cluster NFW (and Einasto) fit parameters for the 12 X-COP clusters into summary statistics — mean concentration, scatter, and fractional deviation from the nonparametric hydrostatic mass profiles — and benchmarks them against the ΛCDM concentration–mass prediction.

## When to use

Use when you have per-cluster posterior samples of parametric mass-model parameters (c200, M200) and need population-level statistics (mean concentration, scatter) and deviation thresholds to validate a mass model.

## When NOT to use

- When individual cluster profile diversity is the target rather than population-level averages — the article notes more shape diversity than NFW can capture, which summary statistics mask
- When the input mass profiles are parametric rather than nonparametric reconstructions, since the <10% deviation criterion is defined against the nonparametric log-normal-mixture profiles

## Inputs

- Posterior samples of NFW fit parameters (c200, M200) for the 12 X-COP clusters from hydromass.mhyd PyMC3/HMC runs
- Posterior samples of Einasto fit parameters (c200, M200, shape parameter alpha) for the same clusters
- Nonparametric hydrostatic mass profiles (log-normal mixture reconstruction) for the 12 X-COP clusters
- ΛCDM concentration–mass relation prediction for comparison

## Outputs

- Per-cluster NFW and Einasto parameters table
- Summary statistics: average fractional deviation, mean NFW concentration, and scatter relative to the ΛCDM c–M prediction
- CSV file nfw_einasto_fit_summary.csv

## How to apply

Load the per-cluster posterior samples produced by hydromass.mhyd parametric reconstruction (HMC via PyMC3) for the NFW and Einasto fits of the 12 X-COP clusters. For each cluster, compute the fractional deviation between the NFW/Einasto fit and the nonparametric log-normal-mixture mass profile over the fitted radial range out to R200, and evaluate against the <10% criterion. Aggregate the per-cluster NFW concentrations into a mean and scatter. Compare the mean concentration and its scatter to the ΛCDM concentration–mass prediction to assess consistency with the dark-matter-only framework. Write per-cluster parameters and summary statistics (average deviation, mean concentration, scatter vs. the ΛCDM c–M relation) to nfw_einasto_fit_summary.csv.

## Related tools

- **hydromass** (Provides the parametric (NFW, Einasto) reconstruction with PyMC3/HMC whose per-cluster posteriors feed the aggregation; invoked via hydromass.mhyd) — https://github.com/domeckert/hydromass
- **pymc3** (Backend for Hamiltonian Monte Carlo sampling of the parametric mass-model posteriors)
- **numpy** (Computes means, scatter, and fractional deviations over posterior samples and radial bins)
- **scipy** (Dependency of hydromass used for numerical routines in profile evaluation)
- **astropy** (Cosmology and unit handling when expressing c200, M200, and R200)

## Examples

```
import numpy as np; devs = np.abs(m_nfw / m_np - 1.0); print(np.mean(c200), np.std(c200), np.max(devs[np.asarray(r) < r200]))
```

## Evaluation signals

- Average fractional deviation between NFW fit and nonparametric profile is less than 10% over a wide radial range
- Mean NFW concentration and its scatter agree with the ΛCDM concentration–mass prediction
- Summary CSV nfw_einasto_fit_summary.csv contains 12 per-cluster rows with c200, M200, alpha (Einasto), and deviation columns
- Deviations are computed consistently out to R200 for all clusters
- Einasto fits recover a shape parameter alpha alongside c200 and M200 for every cluster

## Limitations

- There is more diversity in the shape of individual profiles than can be captured by the NFW model, so population averages hide per-cluster deviations
- The <10% deviation claim holds over a wide but finite radial range, not the full profile
- The version of hydromass used is not stated, so exact function signatures may differ across releases

## Evidence

- [abstract] NFW deviations <10% criterion: "the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range"
- [abstract] ΛCDM concentration agreement: "The average NFW concentration and its scatter agree very well with the prediction of the ΛCDM framework"
- [readme] Parametric model support in hydromass: "Parametric mass model reconstruction including Navarro-Frenk-White, Einasto and several other popular mass models, with automatic or custom priors"
- [methods] HMC sampling machinery: "Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3"
- [abstract] Profile diversity limitation: "there appears to be more diversity in the shape of individual profiles than can be captured by the NFW model"
