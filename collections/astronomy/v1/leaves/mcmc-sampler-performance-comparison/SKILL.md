---
name: mcmc-sampler-performance-comparison
description: 'Use when benchmarking an ensemble MCMC sampler against a self-tuning
  Metropolis–Hastings sampler: you have autocorrelation times tau, burn-in length
  b, walker count L and sequential steps n, and need to find at what sampling effort
  epsilon <= 0.1.'
when_to_use_negative:
- 'When the sampler uses Metropolis–Hastings only: the paper notes it ''is dominated
  by overheads'' in parallelised MCMC, so the ensemble-sampler epsilon formula with
  walker count L does not directly apply.'
- When you already have a converged posterior sample and only need parameter inference
  — this skill is a sampling-effort benchmark, not an inference workflow.
- When scaling analysis (wall time vs core count) is the goal rather than statistical
  error convergence; the paper treats these as separate analyses (e.g. the 11% scaling
  overhead at alpha = 0.89).
edam_operation: http://edamontology.org/operation_2238
tools:
- name: Python
  role: numpy is used to evaluate epsilon = |2*tau/((n-b)*L)|^(1/2) at every (L, n)
    grid point; matplotlib is used to plot epsilon versus n with the epsilon = 0.1
    reference line
- name: CosmoHammer
  role: Python framework embedding emcee (affine invariant ensemble sampler) whose
    sampling error convergence is compared against self-tuning CosmoMC
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: CosmoMC
  role: Widely used Metropolis–Hastings-based Fortran MCMC program serving as the
    self-tuning baseline whose tuning endpoint is n = 2000
- name: acor
  role: Python wrapper estimating the autocorrelation time tau of time series data,
    which feeds the epsilon calculation
  repo: https://github.com/dfm/acor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:2c5602857e132ab0484377429031b06200587192c80ce927e66ebf50213c2ed8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  repo_url: https://github.com/cosmo-ethz/CosmoHammer
  tools:
  - CosmoHammer
  - CosmoMC
  - Python
  - acor
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mcmc-sampler-performance-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitatively compare the statistical error epsilon of an ensemble MCMC sampler (CosmoHammer/emcee) against a self-tuning Metropolis–Hastings sampler (CosmoMC) by evaluating epsilon = |2*tau/((n-b)*L)|^(1/2) over a grid of walker counts L and sequential steps n, using autocorrelation times tau from the paper's Table 2.

## When to use

Use when benchmarking an ensemble MCMC sampler against a self-tuning Metropolis–Hastings sampler: you have autocorrelation times tau, burn-in length b, walker count L and sequential steps n, and need to find at what sampling effort epsilon <= 0.1.

## When NOT to use

- When the sampler uses Metropolis–Hastings only: the paper notes it 'is dominated by overheads' in parallelised MCMC, so the ensemble-sampler epsilon formula with walker count L does not directly apply.
- When you already have a converged posterior sample and only need parameter inference — this skill is a sampling-effort benchmark, not an inference workflow.
- When scaling analysis (wall time vs core count) is the goal rather than statistical error convergence; the paper treats these as separate analyses (e.g. the 11% scaling overhead at alpha = 0.89).

## Inputs

- Autocorrelation times tau from Table 2 of arXiv:1212.1721 for the (L, n) grid
- Burn-in/tuning length b from the paper's benchmark configuration
- Self-tuning CosmoMC tuning endpoint (n = 2000)
- Walker counts L (e.g. L = 10 and larger) and sequential step counts n

## Outputs

- Array/table of epsilon values per (L, n) grid point
- Figure epsilon_convergence_curves.png showing epsilon vs n for CosmoHammer and self-tuning CosmoMC with a reference line at epsilon = 0.1
- Step count at which CosmoHammer first reaches epsilon <= 0.1

## How to apply

Retrieve the autocorrelation times tau from Table 2 of the CosmoHammer paper (arXiv:1212.1721) for the grid of walker counts L and sequential step counts n, and fix the burn-in length b plus the self-tuning CosmoMC tuning endpoint at n = 2000 per the paper's benchmark configuration. Use Python (numpy) to evaluate epsilon = |2*tau/((n-b)*L)|^(1/2) at every (L, n) grid point. Plot (matplotlib) epsilon versus n for CosmoHammer and for self-tuning CosmoMC on a single axes, drawing a horizontal reference line at epsilon = 0.1. The decision point is whether the CosmoHammer curve drops to epsilon <= 0.1 at a sequential step count below n = 2000, i.e. before CosmoMC finalises its proposal tuning; for as few as 10 walkers CosmoHammer reaches the 10% error regime before the self-tuning CosmoMC run even finalises its tuning. Save the resulting figure as epsilon_convergence_curves.png.

## Related tools

- **Python** (numpy is used to evaluate epsilon = |2*tau/((n-b)*L)|^(1/2) at every (L, n) grid point; matplotlib is used to plot epsilon versus n with the epsilon = 0.1 reference line)
- **CosmoHammer** (Python framework embedding emcee (affine invariant ensemble sampler) whose sampling error convergence is compared against self-tuning CosmoMC) — https://github.com/cosmo-ethz/CosmoHammer
- **CosmoMC** (Widely used Metropolis–Hastings-based Fortran MCMC program serving as the self-tuning baseline whose tuning endpoint is n = 2000)
- **acor** (Python wrapper estimating the autocorrelation time tau of time series data, which feeds the epsilon calculation) — https://github.com/dfm/acor

## Examples

```
import numpy as np; eps = np.sqrt(np.abs(2*tau/((n-b)*L)))  # tau from Table 2, burn-in b, walkers L, steps n
```

## Evaluation signals

- The CosmoHammer curve drops to epsilon <= 0.1 at a sequential step count below n = 2000, i.e. before the self-tuning CosmoMC run finalises its tuning at n = 2000.
- For L = 10 walkers, CosmoHammer reaches the 10% error regime before CosmoMC finalises its tuning (the specific claim of Fig. 3).
- epsilon decreases as 1/sqrt((n-b)*L): doubling the effective sample size (n-b)*L reduces epsilon by a factor of sqrt(2).
- The horizontal reference line at epsilon = 0.1 is present and the figure is saved as epsilon_convergence_curves.png.
- Burn-in b is subtracted from n in the denominator for both samplers, consistent with the paper's benchmark burn-in/tuning lengths.

## Limitations

- Scaling saturates: once there are more processes than walkers, processes idle, which limits the effective parallelism of the ensemble sampler.
- An 11% scaling overhead when doubling cores (alpha = 0.89) means wall-time gains are not perfectly linear, affecting the cost side of the error-vs-effort comparison.
- The comparison depends on the specific benchmark configuration (WMAP likelihood, burn-in lengths from Table 2, tuning endpoint n = 2000); results do not automatically transfer to other likelihoods or samplers.

## Evidence

- [other] Fig. 3 shows that for as few as 10 walkers, Cosmo Hammer reaches the 10% error regime before the self-tuning CosmoMC run even finalises its tuning at n = 2000.: [evidence span withheld — anchor status: not_found]
- [readme] Autocorrelation-time estimation tool that supplies tau for the epsilon formula.: [evidence span withheld — anchor status: external_doc]
