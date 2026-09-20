---
name: nuisance-parameter-prior-handling
description: Use when configuring a candl likelihood (e.g. SPT-3G 2018 TT/TE/EE or
  Planck via clipy) and you need to disable, keep, or selectively override its internal
  nuisance parameter priors.
when_to_use_negative:
- When the analysis intends to impose user-defined external priors and the likelihood's
  internal priors would silently double-count — clear them instead
- When working with a candl version prior to the default-behaviour change and assuming
  internal priors are already ignored on newer versions — the active prior set depends
  on version
tools:
- name: candl
  role: Differentiable CMB likelihood framework whose configuration keys (clear_internal_priors,
    clear_specific_priors) control nuisance parameter prior handling
  repo: github.com/Lbalkenhol/candl
- name: clipy
  role: Pure Python JAX-compatible Planck likelihood implementation natively compatible
    with candl, whose nuisance priors are subject to the same handling decisions
  repo: https://github.com/benabed/clipy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:bab76b7d2259e0b611ff19a68e6863e46023bd66ead629c96739be355dd2c5d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  - clipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Nuisance parameter prior handling

## Summary

Controls how internal and specific nuisance parameter priors in a candl CMB likelihood are activated, deactivated, or overridden via likelihood configuration keys, so that Fisher forecasts and MCMC analyses reflect the intended prior set rather than defaults.

## When to use

Use when configuring a candl likelihood (e.g. SPT-3G 2018 TT/TE/EE or Planck via clipy) and you need to disable, keep, or selectively override its internal nuisance parameter priors.

## When NOT to use

- When the analysis intends to impose user-defined external priors and the likelihood's internal priors would silently double-count — clear them instead
- When working with a candl version prior to the default-behaviour change and assuming internal priors are already ignored on newer versions — the active prior set depends on version

## Inputs

- candl likelihood object initialised from a .yaml index file (e.g. candl_data.SPT3G_2018_TTTEEE)
- Likelihood configuration dictionary/YAML with prior-control keys (clear_internal_priors, clear_specific_priors)
- List of nuisance parameters whose priors should be selectively removed

## Outputs

- Likelihood instance with the intended prior configuration active
- Differentiable parameters-to-log-likelihood function whose derivatives include (or exclude) prior contributions as intended

## How to apply

Set the prior policy in the likelihood configuration before evaluating the likelihood: 'clear_internal_priors: True' switches off candl's built-in internal priors, while leaving it off retains them. For finer control over individual nuisance priors, use 'clear_specific_priors' to remove only selected priors. Note the breaking change introduced around v1.5: candl internal priors are now ignored by default, so an analysis relying on internal priors must explicitly re-enable them per the migration guidance in the docs. Always verify which prior set is active before computing derivatives (e.g. Fisher matrices via candl.tools.get_fisher_matrix), since prior terms enter the Hessian and directly shift parameter uncertainties.

## Related tools

- **candl** (Differentiable CMB likelihood framework whose configuration keys (clear_internal_priors, clear_specific_priors) control nuisance parameter prior handling) — github.com/Lbalkenhol/candl
- **clipy** (Pure Python JAX-compatible Planck likelihood implementation natively compatible with candl, whose nuisance priors are subject to the same handling decisions) — https://github.com/benabed/clipy

## Examples

```
candl_like = candl.Like(candl_data.SPT3G_2018_TTTEEE); candl_like.clear_internal_priors = True; like = candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs)
```

## Evaluation signals

- Print/inspect the likelihood's active prior set after configuration and confirm it matches the intended policy (internal priors kept or cleared)
- Log-likelihood values at the fiducial parameters must shift consistently with adding/removing prior terms; a change when none is expected indicates the config key was not applied
- Fisher matrix uncertainties (square-root of diagonal of par_cov from candl.tools.get_fisher_matrix) must reflect prior contributions; compare against versions with and without internal priors
- Regression check across candl versions: analyses relying on internal priors must produce unchanged results after the v1.5 default change only if priors are explicitly re-enabled

## Limitations

- Breaking change in default behaviour: candl internal priors are now ignored by default, with migration guidance deferred to external docs rather than included inline
- Some JAX architectures do not support 64-bit computation, causing up to 1e-4 differences in log-likelihood values, which can interact with prior term comparisons

## Evidence

- [methods] clear_internal_priors switches off candl internal priors: "clear_internal_priors: True # Switch off to use candl internal priors"
- [methods] Fine-grained control of specific priors: "Also see ``clear_specific_priors`` for more fine-grained control."
- [discussion] Default behaviour change regarding internal priors: "__NOTE:__ Change in default behaviour: candl internal priors now ignored. Check docs for how to keep them"
