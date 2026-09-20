---
name: radio-transient-pipeline-configuration
description: 'Use when deploying AMBER on new GPU hardware or a new survey scenario
  (e.g. ARTS: 5120 DMs, 6 integration steps) and the optimal OpenCL low-level parameters
  per kernel must be measured and written to configuration files.'
when_to_use_negative:
- Validated configuration files already exist for the same platform and scenario —
  re-tuning would delete and regenerate them unnecessarily
- You want to run the pipeline on data (time series / PSRDADA ringbuffer / LOFAR HDF5
  observations) rather than tune it — use the tuned AMBER pipeline itself
- The target machine has no OpenCL environment — OpenCL is a necessary dependency
  and kernels cannot be tuned or executed
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: AMBER_setup
  role: Provides the amber.sh script whose tune and test modes generate and validate
    AMBER configuration files for a given scenario
  repo: https://github.com/TRASAL/AMBER_setup
- name: AMBER
  role: The FRB pipeline whose per-kernel OpenCL auto-tuner enumerates, times, and
    ranks configurations
  repo: github.com/TRASAL/AMBER
- name: OpenCL
  role: Required runtime whose kernels expose the tunable low-level parameters (work-group
    size, local memory, serial per-thread processing)
  repo: https://github.com/isazi/OpenCL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2020.100549
    title: 'AMBER: A real-time pipeline for the detection of single pulse astronomical
      transients'
  source_content_hash: sha256:3b7b4007858c13ed8cf3d7117ca95bd4af065f5fdf457e82a4cab6b4da94de3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TRASAL/AMBER_setup
  tools:
  - AMBER
  - AMBER_setup
  - OpenCL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# radio-transient-pipeline-configuration

## Summary

Auto-tune AMBER's OpenCL kernels (dedispersion, integration, SNR, RFIm) for a specific GPU platform and observational scenario, generating per-module configuration files that select the fastest work-group size, local memory usage, and serial per-thread work.

## When to use

Use when deploying AMBER on new GPU hardware or a new survey scenario (e.g. ARTS: 5120 DMs, 6 integration steps) and the optimal OpenCL low-level parameters per kernel must be measured and written to configuration files.

## When NOT to use

- Validated configuration files already exist for the same platform and scenario — re-tuning would delete and regenerate them unnecessarily
- You want to run the pipeline on data (time series / PSRDADA ringbuffer / LOFAR HDF5 observations) rather than tune it — use the tuned AMBER pipeline itself
- The target machine has no OpenCL environment — OpenCL is a necessary dependency and kernels cannot be tuned or executed

## Inputs

- Scenario description file (e.g. examples/scenario.sh) parameterizing the observational scenario (DMs, integration steps)
- Compiled AMBER installation and its modules (Dedispersion, Integration, SNR, RFIm) with a working OpenCL environment
- Target GPU platform on which kernels will be timed

## Outputs

- Per-module configuration files in the chosen output directory (e.g. ${INSTALL_ROOT}/confs)
- tuning_report.csv containing one measured runtime per enumerated configuration
- best_configuration.json recording the selected work-group size, local memory usage, and serial per-thread work per kernel

## How to apply

First set SOURCE_ROOT and INSTALL_ROOT and ensure a working OpenCL environment, then install the pipeline via the AMBER_setup script. Write a scenario description file describing the observational scenario the tuning should target (e.g. the ARTS scenario of 5120 DMs and 6 integration steps). Run the tuner in tune mode, passing the scenario file and the directory where configuration files should be saved: for each module the auto-tuner enumerates all valid low-level OpenCL configurations (work-group size, usage of local memory, amount of serial processing per thread), times each configuration by running the kernel on the chosen GPU, and ranks them by measured runtime. The per-configuration timings are written to tuning_report.csv and the fastest configuration is written to best_configuration.json. Finally validate the generated files by running the tuner in test mode with the same scenario file and configuration directory; be aware that tuning deletes any previously generated configuration files in the target directory.

## Related tools

- **AMBER_setup** (Provides the amber.sh script whose tune and test modes generate and validate AMBER configuration files for a given scenario) — https://github.com/TRASAL/AMBER_setup
- **AMBER** (The FRB pipeline whose per-kernel OpenCL auto-tuner enumerates, times, and ranks configurations) — github.com/TRASAL/AMBER
- **OpenCL** (Required runtime whose kernels expose the tunable low-level parameters (work-group size, local memory, serial per-thread processing)) — https://github.com/isazi/OpenCL

## Examples

```
amber.sh tune examples/scenario.sh ${INSTALL_ROOT}/confs
```

## Evaluation signals

- tuning_report.csv contains exactly one timing row per enumerated configuration for each kernel
- The configuration written to best_configuration.json is the one with the minimum measured runtime in tuning_report.csv
- amber.sh test mode passes on the generated configuration files for the given scenario
- Every selected configuration specifies the three tunable dimensions: OpenCL work-group size, local memory usage, and amount of serial processing per thread

## Limitations

- Running the tuner deletes previously generated configuration files in the same output directory
- Tuning results are specific to one GPU platform and one scenario description; new hardware or a new survey scenario (different DMs/integration steps) requires re-tuning
- Local changes to AMBER source are lost on update via AMBER_setup, and the installer is only tested on Linux with GCC

## Evidence

- [abstract] OpenCL kernels require many low-level parameters: "The OpenCL kernels require many low-level parameters, such as the OpenCL work-group size, the usage of local memory, or the amount of serial processing per thread"
- [abstract] all kernels auto-tuned for performance portability: "all of AMBER’s computational kernels can be tuned to achieve the best possible performance on the hardware the pipeline is executed on"
- [readme] AMBER modules need tuning: [evidence span withheld — anchor status: external_doc]
- [readme] tune mode invocation: [evidence span withheld — anchor status: external_doc]
- [readme] tuning deletes previous configs: [evidence span withheld — anchor status: external_doc]
- [readme] test mode validates configs: [evidence span withheld — anchor status: external_doc]
