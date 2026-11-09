---
name: posterior-estimator-computation-from-dead-points
description: Use when you have a nested sampling run stored as dead points plus the
  iso-likelihood contours at which they were born (PolyChord >=1.14 'write_dead'=True
  output), and you need posterior inferences such as logZ or parameter means with
  standard error estimates on a single run.
when_to_use_negative:
- Input is raw live-point-only output without born iso-likelihood contour information
  (e.g. PolyChord <1.14 with 'write_dead' disabled), since the thread division algorithm
  requires born contours to split the run
- Input is posterior samples from an MCMC sampler rather than nested sampling dead
  points — nestcheck is designed for analysing samples produced by nested sampling
- You need parameter estimation errors from many independent nested sampling runs
  averaged together instead of from a single run; this skill addresses the single-run
  case, where the paper finds no prior method could measure errors accurately
tools:
- name: nestcheck
  role: 'Core package: data_processing loads the run, ns_run_utils computes estimators
    on numpy arrays, error_analysis performs thread-division-based uncertainty estimation'
  repo: https://github.com/ejhigson/nestcheck
- name: PolyChord
  role: Produces the nested sampling runs (dead points with born contours, >=v1.14
    'write_dead'=True) that are the input to the analysis
  repo: https://github.com/PolyChord/
- name: numpy
  role: Fast array functions for computing estimator values on runs and resampled
    thread collections
- name: pandas
  role: Writes estimator values and uncertainty estimates to error_analysis_results.csv
- name: nose
  role: Runs the package test suite from the root nestcheck directory to validate
    the implementation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1214/17-ba1075
    title: Sampling Errors in Nested Sampling Parameter Estimation
  source_content_hash: sha256:c35e724afd550a09bd0c573aa6e230b064826652e1f3372ee80583bfbf12c1c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ejhigson/nestcheck
  tools:
  - PolyChord
  - nestcheck
  - nose
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# posterior-estimator-computation-from-dead-points

## Summary

Computes nested sampling evidence and posterior estimators (e.g. logZ, parameter means) from the dead points of a single run using nestcheck's numpy-based ns_run_utils, and estimates their standard errors via resampled thread collections — the method proposed in Higson et al. (2018) for measuring parameter estimation errors of a single run.

## When to use

Use when you have a nested sampling run stored as dead points plus the iso-likelihood contours at which they were born (PolyChord >=1.14 'write_dead'=True output), and you need posterior inferences such as logZ or parameter means with standard error estimates on a single run.

## When NOT to use

- Input is raw live-point-only output without born iso-likelihood contour information (e.g. PolyChord <1.14 with 'write_dead' disabled), since the thread division algorithm requires born contours to split the run
- Input is posterior samples from an MCMC sampler rather than nested sampling dead points — nestcheck is designed for analysing samples produced by nested sampling
- You need parameter estimation errors from many independent nested sampling runs averaged together instead of from a single run; this skill addresses the single-run case, where the paper finds no prior method could measure errors accurately

## Inputs

- PolyChord >=1.14 output with 'write_dead'=True: dead points plus the iso-likelihood contours at which they were born
- Nested sampling samples from MultiNest, PolyChord, dyPolyChord, dynesty or perfectns (nestcheck-compatible output formats)

## Outputs

- Evidence and posterior estimator values (logZ, parameter means) computed on the full run
- Standard error estimates on the posterior inferences from resampled thread collections
- error_analysis_results.csv containing estimator values and their uncertainty estimates

## How to apply

Load the run (dead points plus born iso-likelihood contours) from PolyChord >=1.14 'write_dead'=True output using nestcheck.data_processing, which is needed to split nested sampling runs into their constituent single live point runs ('threads') per the thread division algorithm of Higson et al. (2018). Compute evidence and posterior estimators (logZ, parameter means) on the full run with numpy arrays via nestcheck.ns_run_utils. Generate resampled thread collections and compute estimator values on each to obtain standard error estimates on the posterior inferences via nestcheck.error_analysis, which uses fast numpy functions and parallelisation with concurrent.futures. Write the estimator values and their uncertainty estimates to error_analysis_results.csv with pandas, then validate by running the nosetests suite from the root nestcheck directory.

## Related tools

- **nestcheck** (Core package: data_processing loads the run, ns_run_utils computes estimators on numpy arrays, error_analysis performs thread-division-based uncertainty estimation) — https://github.com/ejhigson/nestcheck
- **PolyChord** (Produces the nested sampling runs (dead points with born contours, >=v1.14 'write_dead'=True) that are the input to the analysis) — https://github.com/PolyChord/
- **numpy** (Fast array functions for computing estimator values on runs and resampled thread collections)
- **pandas** (Writes estimator values and uncertainty estimates to error_analysis_results.csv)
- **nose** (Runs the package test suite from the root nestcheck directory to validate the implementation)

## Examples

```
import nestcheck.data_processing; import nestcheck.ns_run_utils; import nestcheck.error_analysis  # load PolyChord dead-point run, compute logZ/parameter means, estimate standard errors
```

## Evaluation signals

- All tests pass when running the nestcheck suite with nosetests from the root nestcheck directory
- error_analysis_results.csv is written and contains estimator values paired with their uncertainty estimates
- The run can be split into single live point threads using the born iso-likelihood contour information without errors
- logZ and parameter means computed via nestcheck.ns_run_utils on the full run are finite and consistent with the run's dead-point distribution
- Standard errors from resampled thread collections are computed for each estimator (non-zero, finite values)

## Limitations

- The paper finds no existing method can accurately measure parameter estimation errors of a single nested sampling run; this thread-division approach is the newly proposed method, so results should be validated against tests rather than assumed correct
- PolyChord >=1.14 with 'write_dead'=True (or equivalent output format) is required; the diagnostic-repository results used Python 3.6, nestcheck v0.1.6 and PolyChord v1.14
- This is research code and is not actively maintained (per the diagnostic repository README)

## Evidence

- [abstract] nestcheck purpose: "``nestcheck`` is a Python package for analysing samples produced by nested sampling, and estimating uncertainty on posterior inferences."
- [abstract] no prior method for single-run errors: "We find no current method can accurately measure the parameter estimation errors of a single nested sampling run, and propose a method for doing so"
- [methods] thread splitting: "which is needed to split nested sampling runs into their constituent single live point runs ("threads");"
- [methods] nestcheck utilities: "``nestcheck`` provides Python utilities for analysing samples produced by nested sampling, and estimating uncertainties on nested sampling calculations"
- [methods] test suite: "You can run the test suite with `nose <http://nose.readthedocs.org/>`_. From the root ``nestcheck`` directory, run:"
- [readme] tested versions: [evidence span withheld — anchor status: external_doc]
