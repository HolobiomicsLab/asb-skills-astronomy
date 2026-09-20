---
name: dex-accuracy-quantification
description: Use when you have fitted model MAHs and simulated halo mass-assembly
  histories (from merger trees of gravity-only or TNG simulations) and need to validate
  approximation accuracy in dex units over a time range.
when_to_use_negative:
- Your goal is fitting the diffmah parameters themselves rather than quantifying the
  accuracy of already-fitted curves against simulated MAHs
- You need to characterize halo assembly bias (residual correlations with the large-scale
  density field) rather than per-halo accuracy in dex
- Your halos are below the 10^11 Msun present-day mass threshold, where the paper's
  0.1 dex accuracy claim does not apply
edam_operation: http://edamontology.org/operation_2945
tools:
- name: diffmah
  role: Provides the 3-parameter differentiable power-law model used to evaluate the
    fitted mass-assembly curves being compared against simulated histories
  repo: https://github.com/ArgonneCPAC/diffmah
- name: numpy
  role: Loads fitted parameters and simulated MAHs, evaluates model curves on the
    time grid, and computes pointwise dex residuals and median/typical absolute residual
    statistics
- name: jax
  role: Autodiff backend underlying the diffmah implementation used to evaluate the
    model histories
  repo: https://jax.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:50d6f57535d94cf2df43c22e5b4628a3967b7c6d8ec5c9c6ba4907989de59295
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - Python >=3.9
  - diffmah
  - jax
  - numpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dex-accuracy-quantification

## Summary

Quantifies how well fitted diffmah mass-assembly histories approximate simulated halo growth by computing pointwise residuals in dex and aggregating typical accuracy statistics over cosmic time and halo-mass cuts.

## When to use

Use when you have fitted model MAHs and simulated halo mass-assembly histories (from merger trees of gravity-only or TNG simulations) and need to validate approximation accuracy in dex units over a time range.

## When NOT to use

- Your goal is fitting the diffmah parameters themselves rather than quantifying the accuracy of already-fitted curves against simulated MAHs
- You need to characterize halo assembly bias (residual correlations with the large-scale density field) rather than per-halo accuracy in dex
- Your halos are below the 10^11 Msun present-day mass threshold, where the paper's 0.1 dex accuracy claim does not apply

## Inputs

- Fitted diffmah parameters per halo (from a prior MAH fitting step, loadable with numpy)
- Simulated halo mass assembly histories on a cosmic-time grid, from merger trees of gravity-only or TNG hydrodynamical simulations
- Present-day halo masses (M0) for applying the 10^11 Msun mass cut

## Outputs

- Aggregated residual statistics (median and typical absolute residual in dex) for the selected halo sample
- residual_accuracy_report.csv containing the aggregated accuracy statistics

## How to apply

Load the fitted diffmah parameters and the simulated halo mass assembly histories (e.g., produced by a prior fitting step) with numpy. Evaluate the 3-parameter diffmah power-law model curves for each halo on a cosmic-time grid restricted to t > 1 Gyr. Compute pointwise residuals in dex as log10(fitted mass / simulated mass), selecting only halos with present-day mass greater than 10^11 Msun (both subhalos and host halos qualify). Aggregate per-halo statistics (median and typical absolute residual in dex) across the sample with numpy and write them to a report such as residual_accuracy_report.csv. Validation criterion: the paper reports typical accuracy of 0.1 dex for t > 1 Gyr for all halos of present-day mass greater than 10^11 Msun; flag the result if the median absolute residual exceeds 0.1 dex.

## Related tools

- **diffmah** (Provides the 3-parameter differentiable power-law model used to evaluate the fitted mass-assembly curves being compared against simulated histories) — https://github.com/ArgonneCPAC/diffmah
- **numpy** (Loads fitted parameters and simulated MAHs, evaluates model curves on the time grid, and computes pointwise dex residuals and median/typical absolute residual statistics)
- **jax** (Autodiff backend underlying the diffmah implementation used to evaluate the model histories) — https://jax.readthedocs.io/en/latest/

## Examples

```
import numpy as np; dex = np.log10(mah_fit / mah_sim); med = np.median(np.abs(dex[t > 1.0])); assert med <= 0.1
```

## Evaluation signals

- Median absolute dex residual over t > 1 Gyr for halos with present-day mass > 10^11 Msun should be at or below 0.1 dex, matching the paper's reported typical accuracy
- Residuals are computed only for t > 1 Gyr — the time grid used in aggregation must exclude earlier times
- Sample must include both subhalos and host halos from gravity-only and TNG hydrodynamical simulations for the result to be comparable to the paper
- residual_accuracy_report.csv is written and contains per-sample aggregated statistics, not per-halo raw residuals
- A flag/warning is raised if the median absolute residual exceeds 0.1 dex

## Limitations

- The 0.1 dex accuracy claim applies only for t > 1 Gyr and halos of present-day mass greater than 10^11 Msun; accuracy outside these ranges is not asserted
- Accuracy depends on the prior fitting step: residuals are only as good as the diffmah 3-parameter fits to the simulated merger trees
- Residual dex errors should be checked for negligible correlation with the large-scale density field if assembly-bias fidelity matters

## Evidence

- [abstract] 0.1 dex accuracy claim: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr"
- [methods] diffmah model definition: "Diffmah is a differentiable model for the mass assembly history of individual and populations of dark matter halos."
- [readme] project data location: [evidence span withheld — anchor status: external_doc]
