---
name: large-scale-density-field-correlation-analysis
description: Use when you have fitted halo accretion histories (e.g. diffmah parameters),
  halo positions, and a large-scale density field, and you need to quantify the correlation
  between halo assembly class or model residuals and environment.
when_to_use_negative:
- Input is raw halo merger-tree data without fitted diffmah accretion-history parameters
  — fit the histories first (this skill consumes task_001 outputs)
- The goal is fitting or validating the diffmah 3-parameter model itself rather than
  testing whether its residuals correlate with environment
- The simulation data lack a large-scale density field or halo positions, so environment
  assignment for each halo is impossible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: diffmah
  role: Provides the 3-parameter accretion-history fits whose residuals are correlated
    against the density field, and the basis for fast- vs. slow-accreting classification
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: numpy
  role: Used to load halo positions and the TNG large-scale density field and to implement
    the correlation estimators
  repo: https://numpy.org/
- name: jax
  role: Autodiff backend underlying the diffmah model that produces the assembly-history
    approximations analyzed here
  repo: https://jax.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:edd82a100d0c810128bae32b7df270ce0a2d5b7d84dc438046e7d5656dcf6aed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - Python
  - diffmah
  - jax
  - numpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# large-scale-density-field-correlation-analysis

## Summary

A numpy-based correlation analysis that tests whether halo assembly histories (and residuals of diffmah approximations to them) are correlated with the large-scale density field in a simulation such as TNG, i.e. whether halo assembly bias is captured by an empirical model of halo assembly.

## When to use

Use when you have fitted halo accretion histories (e.g. diffmah parameters), halo positions, and a large-scale density field, and you need to quantify the correlation between halo assembly class or model residuals and environment.

## When NOT to use

- Input is raw halo merger-tree data without fitted diffmah accretion-history parameters — fit the histories first (this skill consumes task_001 outputs)
- The goal is fitting or validating the diffmah 3-parameter model itself rather than testing whether its residuals correlate with environment
- The simulation data lack a large-scale density field or halo positions, so environment assignment for each halo is impossible

## Inputs

- diffmah 3-parameter accretion-history fits to halo merger trees (e.g. from task_001, ~0.1 dex accuracy for t > 1 Gyr, present-day halo mass > 10^11 M_sun)
- Halo positions from the TNG simulation
- Large-scale density field from the TNG simulation
- Halo assembly classification (fast- vs. slow-accreting) derived from the fitted histories

## Outputs

- Correlation measurements between halo assembly class and the large-scale density field (assembly bias signal)
- Correlation measurements between diffmah approximation residuals and the density field (residual assembly bias)
- assembly_bias_correlations.csv containing the measured correlations

## How to apply

Load the 3-parameter diffmah accretion-history fits (which approximate halo growth to ~0.1 dex accuracy for t > 1 Gyr for halos of present-day mass greater than 10^11 M_sun) alongside halo positions and the large-scale density field from the TNG simulation using numpy. Assign each halo its large-scale density environment and classify halos into fast- vs. slow-accreting populations based on their fitted assembly histories. With numpy-based correlation estimators, compute (1) the correlation between halo assembly class and the large-scale density field, and (2) the correlation between diffmah approximation residuals and the density field. Validate that the residual correlations between diffmah approximation errors and the density field are negligible, which is the paper's evidence that the model accurately captures halo assembly bias, and write the measurements to a CSV such as assembly_bias_correlations.csv.

## Related tools

- **diffmah** (Provides the 3-parameter accretion-history fits whose residuals are correlated against the density field, and the basis for fast- vs. slow-accreting classification) — https://github.com/ArgonneCPAC/diffmah/
- **numpy** (Used to load halo positions and the TNG large-scale density field and to implement the correlation estimators) — https://numpy.org/
- **jax** (Autodiff backend underlying the diffmah model that produces the assembly-history approximations analyzed here) — https://jax.readthedocs.io/en/latest/

## Examples

```
import numpy as np; halos = np.load('tng_halos.npy'); dens = np.load('density_field.npy'); env = assign_environment(halos, dens); corr = np.corrcoef(env, fast_slow_class)[0,1]
```

## Evaluation signals

- Residual correlations between diffmah approximation errors and the large-scale density field are negligible, reproducing the paper's finding that assembly bias is accurately captured by the model
- The primary assembly-class vs. density-field correlation (halo assembly bias) is detected and non-negligible, consistent with the known effect the model is expected to capture
- Results are written to assembly_bias_correlations.csv with the measured correlation values
- Halo sample restricted to the validated regime: present-day mass > 10^11 M_sun and t > 1 Gyr, where diffmah fits hold ~0.1 dex accuracy
- Fast- vs. slow-accreting classification is derived consistently from the fitted assembly histories before correlation estimation

## Limitations

- Conclusions hold only where diffmah fits are accurate: typical 0.1 dex accuracy is demonstrated for t > 1 Gyr and present-day halo masses above 10^11 M_sun; outside this regime residual correlations may not be meaningful
- The validation is specific to the simulations tested in the paper (gravity-only and TNG hydrodynamical simulations); other simulations would require re-verification
- Requires the 3-parameter diffmah fits as a prerequisite; cannot be applied to raw, unfitted halo assembly histories

## Evidence

- [abstract] The model accurately captures halo assembly bias, with negligible residual correlation between approximation errors and the density field.: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model, and that residual errors in our approximations to halo"
- [abstract] Diffmah accuracy regime for the fits consumed by this skill.: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] TNG simulation is the dataset used for halo positions and the density field.: "as well as in the TNG hydrodynamical simulation"
- [methods] numpy is a declared dependency used for the array-based correlation work.: "- `numpy <https://numpy.org/>`__"
- [readme] Project data location for diffmah-fitted halo histories.: [evidence span withheld — anchor status: external_doc]
