---
name: independent-sample-efficiency-quantification
description: Use when comparing MCMC/HMC sampler efficiency between two implementations
  run on the same fitting problem, to decide which sampler (e.g. stan vs emcee) delivers
  more independent posterior draws per step.
when_to_use_negative:
- The goal is best-fit parameter estimates or model residuals rather than a sampler
  efficiency comparison — a single vis-r run with hard --sz u,v averaging suffices.
- The two sampler runs use different step counts or different visibility inputs, so
  per-step independent-sample counts are not comparable.
- The model requires features only present in one implementation (e.g. --astrom independent
  astrometry per visibility dataset, emcee-only), which would make the two runs non-equivalent.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_2269
tools:
- name: vis-r
  role: Provides both the stan and python/emcee fitting executables whose sampler
    efficiency is being compared; run with and without the --stan flag on identical
    inputs
  repo: https://github.com/drgmk/vis-r
- name: stan
  role: HMC-based sampler version of vis-r; uses the Pathfinder algorithm to estimate
    parameter scales before sampling and yields many more independent samples per
    step
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: Ensemble-sampler python version of vis-r against which stan's per-step independent-sample
    yield is benchmarked
  repo: https://github.com/drgmk/vis-r
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:d08904d2b2b1625edf4e4e1a464bbd36be76ea56d594f995ed0e3e67e1294822
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - emcee
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# independent-sample-efficiency-quantification

## Summary

Quantifies the posterior-sampling efficiency of the stan and python/emcee versions of vis-r by extracting the number of independent posterior samples each produces for a matched number of warmup/sampling steps on identical visibility inputs.

## When to use

Use when comparing MCMC/HMC sampler efficiency between two implementations run on the same fitting problem, to decide which sampler (e.g. stan vs emcee) delivers more independent posterior draws per step.

## When NOT to use

- The goal is best-fit parameter estimates or model residuals rather than a sampler efficiency comparison — a single vis-r run with hard --sz u,v averaging suffices.
- The two sampler runs use different step counts or different visibility inputs, so per-step independent-sample counts are not comparable.
- The model requires features only present in one implementation (e.g. --astrom independent astrometry per visibility dataset, emcee-only), which would make the two runs non-equivalent.

## Inputs

- Exported visibilities from an ms file in text or numpy format (e.g. data/HD109573.12m*npy)
- Initial geometry parameters (-g 0.012 -0.035 26.6 76.5)
- Initial radial-profile parameters (-p 0.013 1.07 0.06 0.01)
- Matched warmup/sampling step counts for both samplers

## Outputs

- Per-step independent posterior sample counts for the emcee and stan versions
- Comparison table (e.g. sampler_comparison.csv) of sampler efficiency

## How to apply

Load the same exported visibility files (e.g. data/HD109573.12m*npy) into both sampler paths. Run the python/emcee version with vis-r using identical geometry (-g) and profile (-p) parameters and default warmup/sampling steps; then rerun the identical command with the --stan flag, noting that the stan path first estimates parameter scales with the Pathfinder algorithm before sampling. With the same default step count for both, extract the number of independent posterior samples each sampler produced. Tabulate per-step independent-sample counts for the two versions (e.g. into sampler_comparison.csv) and compare — the expectation is that stan yields many more independent samples for a given number of steps than emcee, because HMC-style gradient-based moves are less autocorrelated than emcee's ensemble walks.

## Related tools

- **vis-r** (Provides both the stan and python/emcee fitting executables whose sampler efficiency is being compared; run with and without the --stan flag on identical inputs) — https://github.com/drgmk/vis-r
- **stan** (HMC-based sampler version of vis-r; uses the Pathfinder algorithm to estimate parameter scales before sampling and yields many more independent samples per step) — https://github.com/drgmk/vis-r
- **emcee** (Ensemble-sampler python version of vis-r against which stan's per-step independent-sample yield is benchmarked) — https://github.com/drgmk/vis-r

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01; vis-r -v data/HD109573.12m*npy --stan -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The comparison table reproduces the reported result that stan yields many more independent samples for a given number of steps than emcee.
- Both sampler runs used the identical command-line inputs (-v, -g, -p values) and the same default warmup/sampling step count.
- The stan run shows evidence of the Pathfinder pre-stage estimating parameter scales before sampling.
- Independent-sample counts are tabulated per sampler for the matched step count (e.g. in sampler_comparison.csv) rather than only raw draws.

## Limitations

- Both versions assume a single disk plane, and multiple radial components must all be of the same type (e.g. Gaussian, power), constraining the models whose samplers can be compared.
- Only the emcee version supports independent offset parameters (--astrom) for each set of visibilities, so some models cannot be run identically in both samplers.
- The two implementations differ in minor ways beyond the sampler itself; other observed differences may not be attributable to sampling efficiency alone.

## Evidence

- [readme] stan yields many more independent samples: "Both default to the same number of 
warmup/sampling steps, but `stan` will yield many more independent samples
for a given number of steps."
- [readme] Pathfinder scale estimation: "The `stan` version first estimates the parameter scales with the Pathfinder algorithm,
 and only the `emcee` version has independent astrometry fitting for 
 each visibility dataset `--astrom`"
- [intro] minor differences between versions: "There are `stan` and `python/emcee` 
versions that are generally the same, but with minor differences."
- [intro] independent offsets in emcee: "Each set of visibilities can have independent offset
parameters in the `emcee` version."
