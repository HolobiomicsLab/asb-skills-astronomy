---
name: log-mass-residual-accuracy-evaluation
description: Use when you have per-halo diffmah -fit parameters and the original merger-tree
  mass histories and need to quantify fit quality via log-mass residuals against the
  0.1 dex accuracy benchmark.
when_to_use_negative:
- Evaluating fit accuracy at t <= 1 Gyr, where the paper's 0.1 dex accuracy guarantee
  does not apply
- Halos below the present-day mass threshold of 10^11 solar masses, which are outside
  the paper's validation sample
- Populations-level statistics (mean MAH, assembly diversity) rather than per-halo
  approximation accuracy — use the halo population assembly model instead
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: diffmah
  role: Provides the differentiable 3-parameter halo mass assembly model whose fits
    are being evaluated; installable via conda-forge or pip from the GitHub repo
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: JAX
  role: Autodiff library used to minimize log-mass residuals of the diffmah model
    during fitting, and underlying the fit quality evaluation workflow
  repo: https://jax.readthedocs.io/en/latest/
- name: numpy
  role: Array handling for per-halo mass accretion histories and residual computations
- name: Python >=3.9
  role: Runtime environment required by diffmah
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:8d9bf17937b48506c6ca7640ad77f8b76b309e473eb5f0d09d49c15e2749a20d
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
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# log-mass-residual-accuracy-evaluation

## Summary

Evaluates the quality of diffmah 3-parameter fits to individual halo mass accretion histories by measuring log-mass residuals (in dex) between the fitted model and the merger-tree MAH. The paper's acceptance benchmark is typical accuracy of ~0.1 dex for cosmic time t > 1 Gyr.

## When to use

Use when you have per-halo diffmah best-fit parameters and the original merger-tree mass histories and need to quantify fit quality via log-mass residuals against the 0.1 dex accuracy benchmark.

## When NOT to use

- Evaluating fit accuracy at t <= 1 Gyr, where the paper's 0.1 dex accuracy guarantee does not apply
- Halos below the present-day mass threshold of 10^11 solar masses, which are outside the paper's validation sample
- Populations-level statistics (mean MAH, assembly diversity) rather than per-halo approximation accuracy — use the halo population assembly model instead

## Inputs

- Merger-tree mass accretion histories (log halo mass vs. cosmic time) for host halos and subhalos with present-day mass > 10^11 Msun
- Best-fit 3-parameter diffmah parameters per halo (early-time power-law index, transition time, transition smoothness)

## Outputs

- Per-halo log-mass residual distributions in dex as a function of cosmic time
- Fit diagnostics table (e.g. diffmah_fits_tng.csv) with best-fit parameters and accuracy metrics

## How to apply

For each halo, take its merger-tree mass accretion history (log halo mass vs. cosmic time) and the best-fit 3-parameter diffmah model (early-time power-law index, transition time, transition smoothness) obtained by minimizing log-mass residuals with a JAX-based gradient optimizer. Compute residuals as the difference in log10 halo mass between model prediction and simulated history at each snapshot time. Restrict the accuracy evaluation to t > 1 Gyr, since the paper's 0.1 dex guarantee applies only beyond that epoch, and include both host halos and subhalos with present-day mass greater than 10^11 solar masses, including those from the TNG hydrodynamical simulation. Report the typical (distribution-wide) residual magnitude rather than only per-halo values, and retain ~0.1 dex for t > 1 Gyr as the acceptance criterion. Assemble best-fit parameters and per-halo fit diagnostics into a table such as diffmah_fits_tng.csv.

## Related tools

- **diffmah** (Provides the differentiable 3-parameter halo mass assembly model whose fits are being evaluated; installable via conda-forge or pip from the GitHub repo) — https://github.com/ArgonneCPAC/diffmah/
- **JAX** (Autodiff library used to minimize log-mass residuals of the diffmah model during fitting, and underlying the fit quality evaluation workflow) — https://jax.readthedocs.io/en/latest/
- **numpy** (Array handling for per-halo mass accretion histories and residual computations)
- **Python >=3.9** (Runtime environment required by diffmah)

## Examples

```
conda install -c conda-forge diffmah; jupyter notebook diffmah_fitter_demo.ipynb
```

## Evaluation signals

- Typical log-mass residuals are ~0.1 dex for t > 1 Gyr across all evaluated halos
- Evaluation set includes both host halos and subhalos with present-day mass > 10^11 Msun, from both gravity-only and TNG hydrodynamical simulations
- Residual errors show negligible residual correlation with the density field (assembly bias is accurately captured)
- Output diagnostics table contains best-fit parameters (early-time power-law index, transition time, transition smoothness) plus per-halo accuracy metrics

## Limitations

- The 0.1 dex accuracy benchmark holds only for t > 1 Gyr; early-time residuals can exceed this
- Validation is limited to halos with present-day mass greater than 10^11 solar masses; lower-mass halos are untested
- No changelog was found for the diffmah repository, so version-specific fitting behavior should be pinned explicitly

## Evidence

- [abstract] abstract: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [methods] methods-install: "The latest release of diffmah is available on pip and conda-forge:

       conda install -c conda-forge diffmah"
- [readme] readme-demo: [evidence span withheld — anchor status: external_doc]
- [abstract] results-assembly-bias: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model, and that residual errors in our approximations to halo"
