---
name: model-fit-accuracy-aggregation
description: Use when you have fitted diffmah parameters paired with simulated halo
  mass assembly histories and need to quantify, per-halo and across the sample, whether
  the 3-parameter model meets the 0.1 dex accuracy claim for t > 1 Gyr.
when_to_use_negative:
- Halo mass assembly histories are not yet fitted with the diffmah 3-parameter model
  — fit them first (e.g., via diffmah_fitter_demo.ipynb or history_fitting_script.py)
  before assessing accuracy
- The halo sample is dominated by low-mass halos below 10^11 solar masses, for which
  the paper's 0.1 dex accuracy claim is not made
- The analysis targets t < 1 Gyr, where the paper explicitly does not claim 0.1 dex
  accuracy
edam_operation: http://edamontology.org/operation_2945
tools:
- name: diffmah
  role: Provides the 3-parameter power-law model whose evaluated curves are compared
    against simulated halo growth histories to compute dex residuals
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: numpy
  role: Loads fitted parameters and simulated histories, evaluates model curves on
    the t > 1 Gyr grid, computes pointwise dex residuals, and aggregates median/typical
    absolute residual statistics
- name: jax
  role: Autodiff backend underlying the diffmah implementation used to evaluate the
    fitted model curves
  repo: https://jax.readthedocs.io/en/latest/
- name: Python >=3.9
  role: Minimum runtime version required by the diffmah environment used to run the
    residual aggregation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:22fa0763270d4f506863a4e838e9daffac1d290e8c15633549df57ad4679ce16
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# model-fit-accuracy-aggregation

## Summary

Aggregates pointwise residual statistics (in dex) between diffmah 3-parameter power-law fits and simulated halo mass assembly histories, to validate the paper's reported typical accuracy of 0.1 dex for t > 1 Gyr on halos above 10^11 solar masses in present-day mass.

## When to use

Use when you have fitted diffmah parameters paired with simulated halo mass assembly histories and need to quantify, per-halo and across the sample, whether the 3-parameter model meets the 0.1 dex accuracy claim for t > 1 Gyr.

## When NOT to use

- Halo mass assembly histories are not yet fitted with the diffmah 3-parameter model — fit them first (e.g., via diffmah_fitter_demo.ipynb or history_fitting_script.py) before assessing accuracy
- The halo sample is dominated by low-mass halos below 10^11 solar masses, for which the paper's 0.1 dex accuracy claim is not made
- The analysis targets t < 1 Gyr, where the paper explicitly does not claim 0.1 dex accuracy

## Inputs

- Fitted diffmah 3-parameter model parameters per halo (e.g., from diffmah fitting outputs, loaded with numpy)
- Simulated halo mass assembly histories from cosmological merger trees (gravity-only and/or TNG hydrodynamical simulations)
- Cosmic-time grid shared by the fits and the simulated histories
- Per-halo present-day mass values for applying the 10^11 solar mass cutoff

## Outputs

- Per-halo pointwise dex residuals (log10 of fitted-to-simulated mass ratio) for t > 1 Gyr
- Aggregated sample statistics (median and typical absolute residual in dex)
- residual_accuracy_report.csv with the aggregated residual statistics

## How to apply

Load the fitted diffmah parameters and the simulated mass assembly histories produced by the fitting task (e.g., outputs of the history_fitting workflow from ArgonneCPAC/diffmah) with numpy. Evaluate the 3-parameter diffmah curves on a cosmic-time grid restricted to t > 1 Gyr, since the paper's accuracy claim explicitly excludes earlier times. Compute pointwise residuals in dex (log10 of fitted mass divided by simulated mass) per halo, filtering the sample to halos of present-day mass greater than 10^11 solar masses — the paper's claim spans subhalos and host halos in both gravity-only and TNG hydrodynamical simulations. Aggregate per-halo accuracy statistics (median and typical absolute residual in dex) across the sample with numpy and write them to residual_accuracy_report.csv. Decision point: flag the result if the median absolute residual exceeds 0.1 dex, which would contradict the paper's reported typical accuracy.

## Related tools

- **diffmah** (Provides the 3-parameter power-law model whose evaluated curves are compared against simulated halo growth histories to compute dex residuals) — https://github.com/ArgonneCPAC/diffmah/
- **numpy** (Loads fitted parameters and simulated histories, evaluates model curves on the t > 1 Gyr grid, computes pointwise dex residuals, and aggregates median/typical absolute residual statistics)
- **jax** (Autodiff backend underlying the diffmah implementation used to evaluate the fitted model curves) — https://jax.readthedocs.io/en/latest/
- **Python >=3.9** (Minimum runtime version required by the diffmah environment used to run the residual aggregation)

## Examples

```
import numpy as np; resid = np.log10(mah_fit / mah_sim)[:, t > 1.0]; med = np.median(np.abs(resid[mask_m200 > 1e11])); print('median |dex| =', med)
```

## Evaluation signals

- Median absolute dex residual for halos with present-day mass > 10^11 Msun should not exceed 0.1 dex over t > 1 Gyr; flag if it does
- Residuals are computed as log10(fitted_mass / simulated_mass) pointwise on a cosmic-time grid restricted to t > 1 Gyr, not over the full history
- residual_accuracy_report.csv exists and contains aggregated per-sample statistics (median and typical absolute residual)
- Validation should hold for subhalos and host halos in both gravity-only and TNG hydrodynamical simulations, matching the paper's scope
- The halo selection filter (present-day mass > 10^11 solar masses) is applied before aggregation, not after

## Limitations

- The 0.1 dex accuracy holds only for t > 1 Gyr; earlier cosmic times are outside the validated range
- The accuracy claim applies only to halos of present-day mass greater than 10^11 solar masses; lower-mass halos are not covered
- The claim was validated specifically on halo merger trees from high-resolution cosmological simulations, including TNG; other simulations may not reproduce it

## Evidence

- [abstract] 0.1 dex accuracy claim for t > 1 Gyr and mass cutoff: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] residual errors in approximations to halo growth: "residual errors in our approximations to halo"
- [readme] diffmah fitting demo for individual halos: [evidence span withheld — anchor status: external_doc]
- [methods] numpy dependency for the diffmah environment: "- `numpy <https://numpy.org/>`__"
