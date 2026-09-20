---
name: posterior-comparison-across-samplers
description: Use when an MCMC fit (e.g. vis-r on ALMA visibility data such as hr4796.selfcal.npy)
  must be validated against an alternative sampler, or when an emcee posterior has
  insufficient independent samples and HMC convergence must be demonstrated.
when_to_use_negative:
- The analysis goal is image-plane fitting rather than visibility-space radial-profile
  modelling; vis-r is designed for visibilities exported from an ms file
- Only one sampler is available in the environment (e.g. --astrom multi-dataset astrometry
  is emcee-only, and stan requires the cmdstanpy toolchain), so no cross-sampler comparison
  can be constructed
- The visibility data are heavily flagged or unaveraged with no --sz averaging, making
  any sampler fit unreasonably slow before comparison is meaningful
tools:
- name: vis-r
  role: Provides the radial-profile visibility likelihood and both the python/emcee
    and stan executables being compared
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: Reference affine-invariant MCMC sampler used as the baseline posterior
  repo: https://github.com/drgmk/vis-r
- name: stan
  role: Gradient-based HMC/NUTS sampler, preceded by Pathfinder scale estimation,
    run with the --stan flag
  repo: https://github.com/drgmk/vis-r
- name: arviz
  role: Computes R-hat, effective sample size, and trace-stability diagnostics for
    both sampler runs
- name: conda
  role: Environment creation for the vis-r stack including emcee, cmdstanpy, and arviz
    (e.g. scipy=1.11.4 because frank wants scipy<1.12)
  repo: https://github.com/drgmk/vis-r
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:ff1ba790dec845edf45c686d6f39387a36cf1645cc5c0a10a800c94b573df409
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - Python
  - arviz
  - conda
  - emcee
  - stan
  - vis-r
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# posterior-comparison-across-samplers

## Summary

Refit the same vis-r radial-profile model on the same input visibilities with both the python/emcee sampler and the stan HMC/NUTS sampler (--stan flag), then compare the two posteriors and convergence diagnostics to validate that gradient-based Hamiltonian Monte-Carlo is a viable, better-sampled replacement for emcee at equal step budget.

## When to use

Use when an MCMC fit (e.g. vis-r on ALMA visibility data such as hr4796.selfcal.npy) must be validated against an alternative sampler, or when an emcee posterior has insufficient independent samples and HMC convergence must be demonstrated.

## When NOT to use

- The analysis goal is image-plane fitting rather than visibility-space radial-profile modelling; vis-r is designed for visibilities exported from an ms file
- Only one sampler is available in the environment (e.g. --astrom multi-dataset astrometry is emcee-only, and stan requires the cmdstanpy toolchain), so no cross-sampler comparison can be constructed
- The visibility data are heavily flagged or unaveraged with no --sz averaging, making any sampler fit unreasonably slow before comparison is meaningful

## Inputs

- Self-calibrated visibilities exported from an ms file to a numpy file (e.g. hr4796.selfcal.npy)
- Disc geometry parameters supplied via -g (e.g. 0.012 -0.035 26.6 76.5)
- Radial profile parameters supplied via -p (e.g. 0.013 1.07 0.06 0.01)
- Identical warmup/sampling step budget for both the emcee and stan runs

## Outputs

- Two posterior sample sets (emcee and stan/HMC) over the same radial-profile parameters
- arviz convergence diagnostics: R-hat, effective sample size, posterior traces
- Acceptance/rejection verdict for the HMC run based on convergence and posterior consistency with emcee

## How to apply

Load the self-calibrated visibilities (numpy file, e.g. hr4796.selfcal.npy) and run the reference emcee-based vis-r fit, supplying disc geometry parameters via -g and radial profile parameters via -p at the default warmup/sampling steps. Rerun the identical fit adding the --stan flag: the stan version first estimates parameter scales with the Pathfinder algorithm, then samples with the gradient-based HMC/NUTS sampler using the same number of warmup/sampling steps. Compute convergence diagnostics (R-hat, effective sample size, posterior trace stability) for both runs with arviz. Accept the HMC run only if it converges (R-hat near 1, substantially higher effective sample size than emcee at equal step count) and its per-parameter posteriors are consistent with the emcee estimates within the fitted credible intervals; expect stan to yield many more independent samples for a given number of steps.

## Related tools

- **vis-r** (Provides the radial-profile visibility likelihood and both the python/emcee and stan executables being compared) — https://github.com/drgmk/vis-r
- **emcee** (Reference affine-invariant MCMC sampler used as the baseline posterior) — https://github.com/drgmk/vis-r
- **stan** (Gradient-based HMC/NUTS sampler, preceded by Pathfinder scale estimation, run with the --stan flag) — https://github.com/drgmk/vis-r
- **arviz** (Computes R-hat, effective sample size, and trace-stability diagnostics for both sampler runs)
- **conda** (Environment creation for the vis-r stack including emcee, cmdstanpy, and arviz (e.g. scipy=1.11.4 because frank wants scipy<1.12)) — https://github.com/drgmk/vis-r

## Examples

```
vis-r -v data/hr4796.selfcal.npy --stan -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- R-hat for every profile parameter is near 1 in the HMC run (convergence criterion)
- Effective sample size of the stan run is substantially higher than emcee at the same warmup/sampling step count
- Per-parameter HMC posteriors agree with the emcee estimates within the fitted credible intervals
- Posterior traces are visually stable for both runs (no wandering chains)
- The fitted model residuals (best-fit *vismod.npy subtracted, dirty images made) look consistent for both samplers

## Limitations

- The stan and emcee versions of vis-r are 'generally the same, but with minor differences': only emcee implements per-dataset independent astrometry fitting (--astrom) and per-dataset offset parameters, so some models cannot be sampled identically in both versions
- The stan version must first estimate parameter scales with Pathfinder before sampling, which is an extra stage absent from emcee
- Both versions assume a single disk plane and multiple radial components must all be of the same type (e.g. Gaussian, power), restricting the parameter space that can be compared across samplers

## Evidence

- [abstract] HMC usable for ALMA debris-disc fitting: "is sufficient for most ALMA observations of debris discs, and can be used with gradient-based fitting methods such as Hamiltonian Monte-Carlo."
- [intro] stan and emcee versions coexist: "There are `stan` and `python/emcee` 
versions that are generally the same, but with minor differences."
- [methods] Pathfinder pre-stage in stan version: "The `stan` version first estimates the parameter scales with the Pathfinder algorithm"
- [results] HR 4796 visibility input file: "uv_file = '../data/hr4796.selfcal.npy'"
- [readme] Stan yields more independent samples per step: "Both default to the same number of warmup/sampling steps, but `stan` will yield many more independent samples"
- [readme] --stan flag invocation pattern: "or for the `stan` version, just add the `--stan` flag:"
