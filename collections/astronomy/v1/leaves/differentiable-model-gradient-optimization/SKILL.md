---
name: differentiable-model-gradient-optimization
description: Use when you need to approximate individual halo mass growth curves (log
  halo mass vs. cosmic time) from merger-tree catalogs, e.g. TNG host halos or subhalos
  above 10^11 Msun, with a compact analytic parameterization.
when_to_use_negative:
- You need Monte Carlo realizations of cosmologically representative halo histories
  rather than fits to actual merger-tree MAHs — that is a separate capability of the
  diffmah code
- Halos have present-day mass below 10^11 Msun, where the demonstrated ~0.1 dex accuracy
  for t > 1 Gyr was not established
- Your histories are at very early times t < 1 Gyr, where the model's demonstrated
  accuracy does not hold
edam_operation: http://edamontology.org/operation_2428
tools:
- name: diffmah
  role: Differentiable 3-parameter model for the mass assembly history of individual
    and populations of dark matter halos; provides the model being fit
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: JAX
  role: Autodiff library underlying diffmah; supplies the gradient-based optimization
    used to minimize log-mass residuals
  repo: https://jax.readthedocs.io/en/latest/
- name: numpy
  role: Array handling for halo mass accretion histories extracted from merger trees
- name: Python >=3.9
  role: Runtime environment required by diffmah
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:fefbb265995c640155f881fa0f791456a929f80c4cbdf0b87fbb35fd716e487d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python
  - Python >=3.9
  - diffmah
  - numpy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# differentiable-model-gradient-optimization

## Summary

Fit the 3-parameter diffmah power-law model (early-time power-law index, transition time, transition smoothness) to individual halo mass accretion histories using JAX-based gradient optimization, achieving ~0.1 dex log-mass accuracy for t > 1 Gyr.

## When to use

Use when you need to approximate individual halo mass growth curves (log halo mass vs. cosmic time) from merger-tree catalogs, e.g. TNG host halos or subhalos above 10^11 Msun, with a compact analytic parameterization.

## When NOT to use

- You need Monte Carlo realizations of cosmologically representative halo histories rather than fits to actual merger-tree MAHs — that is a separate capability of the diffmah code
- Halos have present-day mass below 10^11 Msun, where the demonstrated ~0.1 dex accuracy for t > 1 Gyr was not established
- Your histories are at very early times t < 1 Gyr, where the model's demonstrated accuracy does not hold

## Inputs

- Halo merger tree catalogs (e.g. from TNG hydrodynamical simulation or gravity-only cosmological simulations)
- Per-halo mass accretion histories: log halo mass vs. cosmic time arrays
- Halo selection: host halos and subhalos with present-day mass > 10^11 Msun

## Outputs

- Best-fit 3-parameter diffmah parameters per halo (early-time power-law index, transition time, transition smoothness)
- Per-halo fit diagnostics (e.g. log-mass residuals, dex accuracy for t > 1 Gyr)
- Assembled parameter/diagnostics table such as diffmah_fits_tng.csv

## How to apply

Load halo merger tree catalogs and select host halos and subhalos with present-day mass greater than 10^11 solar masses; extract each halo's mass accretion history (log halo mass vs. cosmic time) as numpy arrays. Fit the 3-parameter diffmah power-law model — early-time power-law index, transition time, and transition smoothness — to each history by minimizing log-mass residuals with a JAX-based gradient optimizer, exploiting JAX autodiff so the model self-consistently captures mean and variance of halo mass accretion rate across cosmic time. Evaluate per-halo fit quality, retaining ~0.1 dex typical accuracy for t > 1 Gyr as the acceptance criterion, and assemble best-fit parameters plus fit diagnostics into a table such as diffmah_fits_tng.csv. For large halo samples, use the repository's history_fitting_script.py to fit many MAHs in parallel with mpi4py.

## Related tools

- **diffmah** (Differentiable 3-parameter model for the mass assembly history of individual and populations of dark matter halos; provides the model being fit) — https://github.com/ArgonneCPAC/diffmah/
- **JAX** (Autodiff library underlying diffmah; supplies the gradient-based optimization used to minimize log-mass residuals) — https://jax.readthedocs.io/en/latest/
- **numpy** (Array handling for halo mass accretion histories extracted from merger trees)
- **Python >=3.9** (Runtime environment required by diffmah)

## Examples

```
conda create -c conda-forge -n diffit python=3.9 numpy jax pytest ipython jupyter matplotlib scipy h5py diffmah
```

## Evaluation signals

- Per-halo log-mass residuals show typical accuracy of ~0.1 dex for t > 1 Gyr
- Fitted halos all satisfy the selection criterion: present-day mass > 10^11 Msun, including both host halos and subhalos
- The fit reproduces both gravity-only simulation trees and TNG hydrodynamical simulation trees within the same acceptance criterion
- Output CSV (e.g. diffmah_fits_tng.csv) contains exactly 3 best-fit parameters plus fit diagnostics per halo
- Residual approximation errors show negligible correlation with the large-scale density field (no residual assembly-bias signal)

## Limitations

- The ~0.1 dex accuracy is demonstrated only for t > 1 Gyr; earlier times are not covered by the acceptance criterion
- Accuracy was validated only for halos of present-day mass greater than 10^11 Msun; extrapolation to lower-mass halos is not supported by the reported findings
- No changelog is available for the diffmah repository, and discussion-section documentation is limited to repository source metadata

## Evidence

- [abstract] abstract: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [methods] methods: "The latest release of diffmah is available on pip and conda-forge:

       conda install -c conda-forge diffmah"
