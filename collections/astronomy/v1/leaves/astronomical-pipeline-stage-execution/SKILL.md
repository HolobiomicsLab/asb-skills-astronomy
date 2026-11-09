---
name: astronomical-pipeline-stage-execution
description: Use when a dedispersed time series (one series per DM trial, e.g. 5120
  DMs from AMBER's Dedispersion stage) must be downsampled by time integration or
  another AMBER pipeline stage must be run on many-core hardware before SNR computation.
when_to_use_negative:
- Input is raw voltage or filterbank data that has not yet been dedispersed — run
  the Dedispersion stage first, since Integration consumes dedispersed time series.
- You need candidate detection output (beam, DM, time, pulse width, SNR) — that requires
  the downstream SNR stage and thresholding, not the Integration stage alone.
- No OpenCL environment is available — the README states a working OpenCL environment
  is a necessary dependency for the pipeline.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: AMBER
  role: Pipeline within which the Integration stage is executed as an OpenCL kernel;
    configuration files are generated and tested via AMBER_setup
  repo: https://github.com/TRASAL/AMBER
- name: Integration
  role: Many-core integration algorithm library that averages dedispersed samples
    over the configured integration steps
  repo: https://github.com/TRASAL/Integration
- name: OpenCL
  role: Runtime dependency providing the many-core accelerated kernels for the Integration
    stage
  repo: https://github.com/isazi/OpenCL
- name: AMBER_setup
  role: Install, tune, and test scripts (amber.sh install / tune / test) used to build
    and auto-tune the stage on the target hardware
  repo: https://github.com/TRASAL/AMBER_setup
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2020.100549
    title: 'AMBER: A real-time pipeline for the detection of single pulse astronomical
      transients'
  source_content_hash: sha256:6ea97507132770e3a33cab7204b5492cc2f70c259eab6559510f4d5eb548dfea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TRASAL/AMBER
  tools:
  - AMBER
  - AMBER_setup
  - Integration
  - OpenCL
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# astronomical-pipeline-stage-execution

## Summary

Executes a single stage of a many-core radio-astronomy transient pipeline (as in AMBER's four-stage design) by running its dedicated OpenCL-accelerated library — e.g. the Integration stage averaging dedispersed time series over a configured number of integration steps (6 in the ARTS survey) — and writing the stage output file for downstream stages.

## When to use

Use when a dedispersed time series (one series per DM trial, e.g. 5120 DMs from AMBER's Dedispersion stage) must be downsampled by time integration or another AMBER pipeline stage must be run on many-core hardware before SNR computation.

## When NOT to use

- Input is raw voltage or filterbank data that has not yet been dedispersed — run the Dedispersion stage first, since Integration consumes dedispersed time series.
- You need candidate detection output (beam, DM, time, pulse width, SNR) — that requires the downstream SNR stage and thresholding, not the Integration stage alone.
- No OpenCL environment is available — the README states a working OpenCL environment is a necessary dependency for the pipeline.

## Inputs

- Dedispersed time series from the AMBER Dedispersion stage, one series per DM trial (5120 DMs in the ARTS survey configuration)
- Stage configuration specifying the number of integration steps (6 for ARTS) and auto-tuned OpenCL kernel parameters (work-group size, local memory usage, serial processing per thread)

## Outputs

- Downsampled/integrated time series file (integrated_series.bin) with the time dimension reduced by the integration factor and the DM dimension preserved, ready for the SNR stage

## How to apply

Load the dedispersed time series emitted by the Dedispersion stage for all dispersion measure trials (5120 DMs in the ARTS survey configuration). Run the target stage's library — for Integration, the TRASAL/Integration many-core algorithm executed as an OpenCL kernel inside the AMBER pipeline — averaging samples over the configured number of integration steps (6 integration steps for ARTS), keeping the DM dimension intact while reducing the time dimension by the integration factor. Build the stage first by setting INSTALL_ROOT and running 'make install', satisfying its dependencies (isazi/utils, isazi/OpenCL, TRASAL/AstroData on the master branch). Write the result to the stage output file (e.g. integrated_series.bin) for the next pipeline stage (SNR). Because AMBER's kernels are auto-tuned, tune the stage for the execution hardware via the AMBER_setup script before production runs.

## Related tools

- **AMBER** (Pipeline within which the Integration stage is executed as an OpenCL kernel; configuration files are generated and tested via AMBER_setup) — https://github.com/TRASAL/AMBER
- **Integration** (Many-core integration algorithm library that averages dedispersed samples over the configured integration steps) — https://github.com/TRASAL/Integration
- **OpenCL** (Runtime dependency providing the many-core accelerated kernels for the Integration stage) — https://github.com/isazi/OpenCL
- **AMBER_setup** (Install, tune, and test scripts (amber.sh install / tune / test) used to build and auto-tune the stage on the target hardware) — https://github.com/TRASAL/AMBER_setup

## Examples

```
amber.sh tune examples/scenario.sh ${INSTALL_ROOT}/confs && amber.sh test examples/scenario.sh ${INSTALL_ROOT}/confs
```

## Evaluation signals

- integrated_series.bin exists and is readable after the run.
- The time dimension of the output is reduced by the integration factor (6 for ARTS) relative to the input dedispersed series.
- The DM dimension is preserved through integration (5120 DMs in and 5120 DMs out).
- Configuration files used were generated/validated by 'amber.sh tune' and 'amber.sh test' for the execution scenario.
- No quantitative SNR threshold is expected at this stage — structural consistency, not detection metrics, is the acceptance criterion.

## Limitations

- No quantitative metric threshold is reported for the Integration stage in the paper, so only structural consistency checks are available.
- All local changes to AMBER source code are lost when updating via 'amber.sh update'.
- The tuning step deletes previously generated configuration files in the target directory.
- OpenCL is a hard dependency; the stage cannot run without a working OpenCL environment, and kernels must be auto-tuned per hardware for best performance.

## Evidence

- [abstract] AMBER's ARTS survey search space is composed of 5120 DMs and 6 integration steps.: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline composed of four computing stages processing input in real-time: "Currently, the AMBER’s pipeline is composed of four computing stages"
- [abstract] OpenCL kernels require low-level parameters that must be auto-tuned: "The OpenCL kernels require many low-level parameters, such as the OpenCL work-group size, the usage of local memory, or the amount of serial processing per thread"
- [readme] Integration is a many-core algorithm library with an install procedure: [evidence span withheld — anchor status: external_doc]
- [readme] Integration stage depends on utils, OpenCL, and AstroData: [evidence span withheld — anchor status: external_doc]
