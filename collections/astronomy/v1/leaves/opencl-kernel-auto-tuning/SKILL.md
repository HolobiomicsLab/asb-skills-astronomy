---
name: opencl-kernel-auto-tuning
description: Use when an OpenCL or other many-core kernel exposes low-level parameters
  (work-group size, local memory, serial per-thread work) that must be optimized per
  execution platform against a fixed observational workload scenario.
when_to_use_negative:
- The target code does not expose tunable low-level parameters (e.g. already-fixed
  OpenCL work-group sizes), so there is no configuration space to enumerate.
- The goal is portability by prediction rather than measurement — exhaustive timing
  on each platform is required by this skill, which is impractical without access
  to the target hardware.
- You are debugging a specific kernel failure rather than optimizing performance;
  compile with DEBUG=1 via amber.sh instead.
tools:
- name: AMBER_setup
  role: Install/update/compile script whose 'tune' mode drives per-module auto-tuning
    and configuration-file generation, and whose 'test' mode validates the generated
    configuration files
  repo: https://github.com/TRASAL/AMBER_setup
- name: AMBER
  role: FRB pipeline whose modules (Dedispersion, Integration, SNR, RFIm) each contain
    the auto-tuning utility and whose kernels run in the AMBER auto-tuner to be timed
  repo: https://github.com/TRASAL/AMBER
- name: Dedispersion
  role: Many-core incoherent dedispersion algorithm in OpenCL — a representative kernel
    auto-tuned by this skill
  repo: https://github.com/TRASAL/Dedispersion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2020.100549
    title: 'AMBER: A real-time pipeline for the detection of single pulse astronomical
      transients'
  source_content_hash: sha256:172caf9698dba9d686fd4318410170e936e342d79ab93c420583683b76efa4df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TRASAL/AMBER_setup
  tools:
  - AMBER
  - AMBER_setup
  - Dedispersion
  - OpenCL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# opencl-kernel-auto-tuning

## Summary

Exhaustive enumeration-and-timing auto-tuning of OpenCL kernels (work-group size, local memory usage, serial per-thread work) to find the fastest configuration for each kernel on a given GPU platform, as implemented in the AMBER FRB pipeline modules.

## When to use

Use when an OpenCL or other many-core kernel exposes low-level parameters (work-group size, local memory, serial per-thread work) that must be optimized per execution platform against a fixed observational workload scenario.

## When NOT to use

- The target code does not expose tunable low-level parameters (e.g. already-fixed OpenCL work-group sizes), so there is no configuration space to enumerate.
- The goal is portability by prediction rather than measurement — exhaustive timing on each platform is required by this skill, which is impractical without access to the target hardware.
- You are debugging a specific kernel failure rather than optimizing performance; compile with DEBUG=1 via amber.sh instead.

## Inputs

- Scenario description file (e.g. examples/scenario.sh) capturing observational scenario parameters such as number of DMs and integration steps
- OpenCL kernel source for the AMBER module to be tuned (e.g. Dedispersion, Integration, SNR, RFIm)
- Target GPU/many-core platform with a working OpenCL environment

## Outputs

- tuning_report.csv with one timing row per enumerated configuration
- best_configuration.json recording the selected work-group size, local memory usage, and serial per-thread work
- AMBER configuration files for the pipeline modules (written to the directory passed to the tuner)

## How to apply

Select the computational kernel to tune (e.g. the dedispersion kernel) and write a scenario description file capturing the observational workload (e.g. the ARTS scenario of 5120 DMs and 6 integration steps). Enumerate all valid low-level OpenCL configurations for the kernel — work-group size, usage of local memory, and amount of serial processing per thread — and time each configuration on the chosen GPU platform by running the kernel in the AMBER auto-tuner. Rank configurations by measured runtime and select the minimum-runtime one for that kernel-platform pair. Persist per-configuration timings to tuning_report.csv and the selected configuration to best_configuration.json. Because OpenCL kernels require many low-level parameters such as work-group size, local memory usage, and serial processing per thread, exhaustive tuning is what gives AMBER performance portability across GPUs from different vendors.

## Related tools

- **AMBER_setup** (Install/update/compile script whose 'tune' mode drives per-module auto-tuning and configuration-file generation, and whose 'test' mode validates the generated configuration files) — https://github.com/TRASAL/AMBER_setup
- **AMBER** (FRB pipeline whose modules (Dedispersion, Integration, SNR, RFIm) each contain the auto-tuning utility and whose kernels run in the AMBER auto-tuner to be timed) — https://github.com/TRASAL/AMBER
- **Dedispersion** (Many-core incoherent dedispersion algorithm in OpenCL — a representative kernel auto-tuned by this skill) — https://github.com/TRASAL/Dedispersion

## Examples

```
amber.sh tune examples/scenario.sh ${INSTALL_ROOT}/confs
```

## Evaluation signals

- tuning_report.csv contains exactly one timing row per enumerated configuration
- best_configuration.json corresponds to the configuration with the minimum measured runtime in tuning_report.csv
- Generated configuration files pass the amber.sh 'test' mode validation against the same scenario description
- The selected work-group size, local memory usage, and serial per-thread work in best_configuration.json are all valid configurations for the target kernel and platform
- Rerunning the tuner on the same kernel, scenario, and platform reproduces a consistent best configuration (runtime ranking is deterministic up to timing noise)

## Limitations

- Running the tuner deletes previously generated configuration files in the target directory, so existing configurations are overwritten.
- Tuning results are specific to the chosen scenario description and platform; a new scenario or GPU requires re-tuning.
- Exhaustive enumeration and timing of all configurations is required, which is costly for kernels with large configuration spaces.
- Local source changes to AMBER are lost when updating via amber.sh, which can disturb a tuned installation.

## Evidence

- [readme] Enumerate and time all valid configurations for each kernel.: [evidence span withheld — anchor status: external_doc]
- [abstract] Rank configurations by measured runtime and select the fastest one for that kernel and platform.: "all of AMBER’s computational kernels can be tuned to achieve the best possible performance on the hardware the pipeline is executed on"
- [full_text] Timing granularity of tuner.: "The OpenCL kernels require many low-level parameters, such as the OpenCL work-group size, the usage of local memory, or the amount of serial processing per thread"
- [readme] Scenario description as input to tuning.: [evidence span withheld — anchor status: external_doc]
- [readme] Tuner overwrites configuration directory.: [evidence span withheld — anchor status: external_doc]
