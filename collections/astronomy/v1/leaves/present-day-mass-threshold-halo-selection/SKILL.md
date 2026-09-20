---
name: present-day-mass-threshold-halo-selection
description: Use when validating or characterizing halo mass assembly history fits
  (e.g., diffmah residuals in dex) and you must restrict the sample to halos above
  a present-day mass threshold such as 10^11 Msun.
when_to_use_negative:
- When the target population includes low-mass halos below 10^11 Msun — the 0.1 dex
  accuracy claim does not extend to them, so any residual comparison over the full
  catalog would be misinterpreted
- When the goal is population-average assembly modeling rather than validating individual-halo
  fits; the population-level diffmah model has different accuracy characteristics
- When the analysis concerns times earlier than 1 Gyr, where the paper's accuracy
  guarantee does not hold regardless of halo mass
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3353
tools:
- name: numpy
  role: Loads fitted diffmah parameters and simulated MAHs, applies the >10^11 Msun
    present-day mass mask, and aggregates dex residual statistics across the selected
    halos
- name: diffmah
  role: Supplies the 3-parameter power-law model whose 0.1 dex accuracy claim defines
    the selection threshold and the t > 1 Gyr validity window
  repo: https://github.com/ArgonneCPAC/diffmah
- name: jax
  role: Autodiff backend underlying diffmah model evaluation used to compute fitted
    curves on the t > 1 Gyr time grid
  repo: https://jax.readthedocs.io/en/latest/
- name: Python >=3.9
  role: Runtime requirement for diffmah and the validation script
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:adadd4fb7d6f4a11bb9978b0f58bbf288a3915f96305a89e38dbe9a70b6edd75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - Python >=3.9
  - diffmah
  - jax
  - numpy
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3353
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# present-day-mass-threshold-halo-selection

## Summary

Filters simulated halo samples to those with present-day (z=0) mass greater than 10^11 solar masses before validating diffmah assembly-history fits, matching the halo population over which the 3-parameter model's 0.1 dex accuracy claim is defined.

## When to use

Use when validating or characterizing halo mass assembly history fits (e.g., diffmah residuals in dex) and you must restrict the sample to halos above a present-day mass threshold such as 10^11 Msun.

## When NOT to use

- When the target population includes low-mass halos below 10^11 Msun — the 0.1 dex accuracy claim does not extend to them, so any residual comparison over the full catalog would be misinterpreted
- When the goal is population-average assembly modeling rather than validating individual-halo fits; the population-level diffmah model has different accuracy characteristics
- When the analysis concerns times earlier than 1 Gyr, where the paper's accuracy guarantee does not hold regardless of halo mass

## Inputs

- Simulated halo mass assembly histories (MAHs) from cosmological merger trees, including subhalos and host halos, in gravity-only and TNG hydrodynamical simulations
- Fitted diffmah 3-parameter model parameters per halo (from task_001)
- Cosmic-time grid over which MAHs are evaluated (evaluated for t > 1 Gyr)

## Outputs

- Filtered halo sample of present-day mass > 10^11 Msun used for accuracy validation
- Aggregated per-halo residual statistics (median and typical absolute dex residuals) written to residual_accuracy_report.csv

## How to apply

Load the simulated halo mass assembly histories and fitted diffmah parameters (e.g., from task_001 outputs) and read the present-day mass of each halo, applying the same selection to subhalos and host halos alike. Retain only halos whose mass at the final cosmic time exceeds 10^11 Msun, in both gravity-only and TNG hydrodynamical simulations, since the paper's typical accuracy claim of 0.1 dex is explicitly scoped to this population. For each selected halo, restrict evaluation of pointwise dex residuals to cosmic times t > 1 Gyr, consistent with the paper's stated validity range. Aggregate per-halo statistics (e.g., median and typical absolute residual) with numpy across the selected sample, and flag the result if the median absolute residual exceeds 0.1 dex. Write the aggregated statistics (e.g., residual_accuracy_report.csv) so the selection population is documented alongside the accuracy numbers.

## Related tools

- **numpy** (Loads fitted diffmah parameters and simulated MAHs, applies the >10^11 Msun present-day mass mask, and aggregates dex residual statistics across the selected halos)
- **diffmah** (Supplies the 3-parameter power-law model whose 0.1 dex accuracy claim defines the selection threshold and the t > 1 Gyr validity window) — https://github.com/ArgonneCPAC/diffmah
- **jax** (Autodiff backend underlying diffmah model evaluation used to compute fitted curves on the t > 1 Gyr time grid) — https://jax.readthedocs.io/en/latest/
- **Python >=3.9** (Runtime requirement for diffmah and the validation script)

## Examples

```
import numpy as np; mask = m0 > 1e11; res = np.log10(mah_fit[mask, t > 1.0] / mah_sim[mask, t > 1.0]); print(np.median(np.abs(res)))
```

## Evaluation signals

- Selected sample includes only halos with present-day mass > 10^11 Msun; verify no below-threshold halos leak into the residual statistics
- Median absolute residual over the selected sample for t > 1 Gyr should not exceed 0.1 dex; flag the report if it does
- Selection applied uniformly to subhalos and host halos, and separately confirmed in both gravity-only and TNG hydrodynamical simulations
- Aggregated statistics written to residual_accuracy_report.csv include the selection population definition (threshold and t > 1 Gyr window)
- Residuals computed in dex (log10 ratio of fitted to simulated mass), consistent with how the paper reports accuracy

## Limitations

- The 0.1 dex accuracy guarantee applies only for t > 1 Gyr; residuals at earlier cosmic times may exceed this level and should not be pooled into the validation statistic
- The threshold is defined at present day (z=0); selecting on mass at earlier epochs would change the population and invalidate comparison to the paper's claim
- No changelog is available for the diffmah repository, so version-specific behavioral changes in fitting cannot be ruled out from documentation alone

## Evidence

- [abstract] Scoped accuracy claim defining the mass threshold and time window: "our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] Selection population covers both halo types and both simulation types: [evidence span withheld — anchor status: not_found]
- [methods] Dependency and Python version requirement for running the selection and validation workflow: "diffmah requires python>=3.9, and has the following dependencies"
- [readme] README guidance on how to run the fitting workflow that produces the inputs to filter: [evidence span withheld — anchor status: external_doc]
