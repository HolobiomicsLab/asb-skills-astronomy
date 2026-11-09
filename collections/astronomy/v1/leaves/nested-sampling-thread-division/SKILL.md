---
name: nested-sampling-thread-division
description: Use when you have a single nested sampling run (e.g. PolyChord >=1.14
  output written with 'write_dead'=True, containing dead points and the iso-likelihood
  contours at which they were born) and need standard errors on posterior inferences
  such as logZ and parameter means.
when_to_use_negative:
- The input is not nested sampling samples (e.g. MCMC chains) — thread division depends
  on the nested sampling structure of dead points and born iso-likelihood contours.
- The run output lacks born-contour information (e.g. PolyChord <1.14 without 'write_dead'=True),
  so the run cannot be split into constituent single live point runs ('threads').
- You already have many independent nested sampling runs whose spread directly gives
  error estimates — the paper targets the single-run case, where no previous method
  was accurate.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: nestcheck
  role: 'Core package: loads runs (data_processing), splits runs into threads using
    born contours, computes estimators (ns_run_utils), and estimates standard errors
    via thread resampling (error_analysis)'
  repo: https://github.com/ejhigson/nestcheck
- name: numpy
  role: Fast array computation of posterior/evidence estimators and of estimator values
    on resampled thread collections
- name: PolyChord
  role: Source sampler whose >=1.14 'write_dead'=True output provides the dead points
    and born iso-likelihood contours needed for thread division
  repo: https://github.com/PolyChord/
- name: dyPolyChord
  role: Dynamic nested sampling package with the same output format as PolyChord,
    usable as an alternative input source
  repo: https://github.com/ejhigson/dyPolyChord
- name: nose
  role: Runs the nestcheck test suite to validate the implementation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1214/17-ba1075
    title: Sampling Errors in Nested Sampling Parameter Estimation
  source_content_hash: sha256:6f82d3d959ea1ae838814a7882f0fca204505a3310c9e5eb4bb7b41ff3f0abb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ejhigson/nestcheck
  tools:
  - PolyChord
  - dyPolyChord
  - nestcheck
  - nose
  - numpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# nested-sampling-thread-division

## Summary

Estimates parameter estimation errors from a single nested sampling run by splitting the run into its constituent single live point runs ('threads') and resampling thread collections, as implemented in the nestcheck Python package (Higson et al. 2018).

## When to use

Use when you have a single nested sampling run (e.g. PolyChord >=1.14 output written with 'write_dead'=True, containing dead points and the iso-likelihood contours at which they were born) and need standard errors on posterior inferences such as logZ and parameter means.

## When NOT to use

- The input is not nested sampling samples (e.g. MCMC chains) — thread division depends on the nested sampling structure of dead points and born iso-likelihood contours.
- The run output lacks born-contour information (e.g. PolyChord <1.14 without 'write_dead'=True), so the run cannot be split into constituent single live point runs ('threads').
- You already have many independent nested sampling runs whose spread directly gives error estimates — the paper targets the single-run case, where no previous method was accurate.

## Inputs

- PolyChord >=1.14 nested sampling run output written with 'write_dead'=True (dead points plus the iso-likelihood contours at which they were born)
- nested sampling samples from supported samplers: PolyChord, MultiNest, dyPolyChord, dynesty, perfectns

## Outputs

- Posterior and evidence estimator values (e.g. logZ, parameter means) computed on the full run
- Standard error estimates on those estimators, obtained by computing estimator values on resampled thread collections
- error_analysis_results.csv containing estimator values and their uncertainty estimates

## How to apply

Load the nested sampling run (dead points plus the iso-likelihood contours at which they were born) from the PolyChord >=1.14 'write_dead'=True output using nestcheck.data_processing. Split the run into its constituent single live point runs ('threads') using the born contour information, as required by the thread division algorithm — this is the key step since no other method accurately measures the parameter estimation errors of a single run. Compute the posterior and evidence estimators (e.g. logZ, parameter means) on the full run with numpy arrays via nestcheck.ns_run_utils. Generate resampled thread collections and compute estimator values on each to obtain standard error estimates on the posterior inferences via nestcheck.error_analysis, which uses fast numpy functions and parallelisation with concurrent.futures. Write the estimator values and their uncertainty estimates to error_analysis_results.csv with pandas, then validate by running the package test suite with nosetests from the root nestcheck directory.

## Related tools

- **nestcheck** (Core package: loads runs (data_processing), splits runs into threads using born contours, computes estimators (ns_run_utils), and estimates standard errors via thread resampling (error_analysis)) — https://github.com/ejhigson/nestcheck
- **numpy** (Fast array computation of posterior/evidence estimators and of estimator values on resampled thread collections)
- **PolyChord** (Source sampler whose >=1.14 'write_dead'=True output provides the dead points and born iso-likelihood contours needed for thread division) — https://github.com/PolyChord/
- **dyPolyChord** (Dynamic nested sampling package with the same output format as PolyChord, usable as an alternative input source) — https://github.com/ejhigson/dyPolyChord
- **nose** (Runs the nestcheck test suite to validate the implementation)

## Examples

```
from nestcheck import data_processing, error_analysis; run = data_processing.load_ns_run('polychord_out'); ests, errs = error_analysis.run_std_estimators(run, estimator_list)
```

## Evaluation signals

- The package test suite passes when run with nosetests from the root nestcheck directory (optionally with coverage: nosetests --with-coverage --cover-erase --cover-package=nestcheck).
- Estimator values computed on the full run (nestcheck.ns_run_utils) are consistent with those recomputed from the recombined threads, confirming the thread split preserved the run.
- Standard errors from nestcheck.error_analysis match the spread of estimator values across the generated resampled thread collections.
- error_analysis_results.csv contains both estimator values (e.g. logZ, parameter means) and their uncertainty estimates in paired columns.
- Input run was produced by a compatible sampler/version combination (e.g. nestcheck v0.1.6 with PolyChord v1.14, as used for the paper's results).

## Limitations

- Requires PolyChord >=1.14 output with 'write_dead'=True so that born iso-likelihood contours are available to split the run into threads.
- The code is research code and is not actively maintained (per the diagnostic repository README).
- Results in the paper were produced with Python 3.6, nestcheck v0.1.6 and PolyChord v1.14; other version combinations may need revalidation.

## Evidence

- [abstract] No current method can accurately measure the parameter estimation errors of a single nested sampling run; the authors propose a method using a new algorithm for dividing nested sampling runs.: "We find no current method can accurately measure the parameter estimation errors of a single nested sampling run, and propose a method for doing so"
- [methods] Thread splitting via born contour information.: "which is needed to split nested sampling runs into their constituent single live point runs ("threads");"
- [methods] nestcheck's purpose and scope.: "``nestcheck`` provides Python utilities for analysing samples produced by nested sampling, and estimating uncertainties on nested sampling calculations"
- [readme] Version compatibility used for the paper's results.: [evidence span withheld — anchor status: external_doc]
