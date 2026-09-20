---
name: work-group-size-optimization
description: Use when an OpenCL kernel must be tuned to a specific GPU platform to
  select the fastest work-group size, local-memory strategy, and per-thread serial
  workload before production execution.
when_to_use_negative:
- The kernel is not written in OpenCL or does not expose work-group size, local-memory,
  and serial-work parameters — there are no tunable low-level configurations to enumerate.
- Only a single pre-chosen configuration is acceptable (e.g. no benchmarking time
  available); enumeration-and-timing of all configurations is then wasted effort.
- The execution platform differs from the platform used for tuning — timings and the
  selected configuration are platform-specific.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3398
tools:
- name: AMBER
  role: Pipeline whose per-module auto-tuner enumerates, times, and ranks valid OpenCL
    configurations; tuned via amber.sh tune
  repo: https://github.com/TRASAL/AMBER
- name: AMBER_setup
  role: 'Setup/management script driving tuning: amber.sh tune <scenario file> <config
    output path>'
  repo: https://github.com/TRASAL/AMBER_setup
- name: OpenCL
  role: Kernel runtime whose low-level parameters (work-group size, local memory,
    serial per-thread processing) are the tuning space
  repo: https://github.com/isazi/OpenCL
- name: Dedispersion
  role: Example AMBER module with an OpenCL dedispersion kernel that can be auto-tuned
    on many-core accelerators
  repo: https://github.com/TRASAL/Dedispersion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2020.100549
    title: 'AMBER: A real-time pipeline for the detection of single pulse astronomical
      transients'
  source_content_hash: sha256:d851a96357686776184ddce27469ae02c3d9a392a4d696665a74206f0600dde1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TRASAL/AMBER
  tools:
  - AMBER
  - AMBER_setup
  - Dedispersion
  - OpenCL
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3398
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# work-group-size-optimization

## Summary

Exhaustive auto-tuning of OpenCL kernel low-level parameters — work-group size, local memory usage, and serial per-thread processing — by enumerating and timing all valid configurations for a given kernel and GPU platform, as done per module in the AMBER FRB pipeline.

## When to use

Use when an OpenCL kernel must be tuned to a specific GPU platform to select the fastest work-group size, local-memory strategy, and per-thread serial workload before production execution.

## When NOT to use

- The kernel is not written in OpenCL or does not expose work-group size, local-memory, and serial-work parameters — there are no tunable low-level configurations to enumerate.
- Only a single pre-chosen configuration is acceptable (e.g. no benchmarking time available); enumeration-and-timing of all configurations is then wasted effort.
- The execution platform differs from the platform used for tuning — timings and the selected configuration are platform-specific.

## Inputs

- Description of the observational scenario (e.g. examples/scenario.sh; ARTS scenario with 5120 DMs and 6 integration steps)
- Target GPU/OpenCL platform with a working OpenCL environment
- AMBER computational kernel to tune (dedispersion, integration, SNR, RFIm)
- Path where configuration files are saved (e.g. ${INSTALL_ROOT}/confs)

## Outputs

- tuning_report.csv with one timing row per enumerated configuration
- best_configuration.json with the selected work-group size, local memory usage, and serial per-thread work
- AMBER configuration files generated in the tuning output directory

## How to apply

Select the computational kernel to tune (e.g. AMBER's dedispersion kernel) and describe the observational scenario it will run under (e.g. the ARTS scenario of 5120 DMs and 6 integration steps). Enumerate every valid low-level OpenCL configuration: work-group size, usage of local memory, and amount of serial processing per thread. Time each enumerated configuration on the target GPU by running the kernel in the module's auto-tuner, then rank configurations by measured runtime and select the fastest. Write the per-configuration timings to tuning_report.csv and the winning configuration (work-group size, local memory usage, serial per-thread work) to best_configuration.json; AMBER's amber.sh 'tune' mode drives this and requires a scenario description file plus an output configuration directory.

## Related tools

- **AMBER** (Pipeline whose per-module auto-tuner enumerates, times, and ranks valid OpenCL configurations; tuned via amber.sh tune) — https://github.com/TRASAL/AMBER
- **AMBER_setup** (Setup/management script driving tuning: amber.sh tune <scenario file> <config output path>) — https://github.com/TRASAL/AMBER_setup
- **OpenCL** (Kernel runtime whose low-level parameters (work-group size, local memory, serial per-thread processing) are the tuning space) — https://github.com/isazi/OpenCL
- **Dedispersion** (Example AMBER module with an OpenCL dedispersion kernel that can be auto-tuned on many-core accelerators) — https://github.com/TRASAL/Dedispersion

## Examples

```
amber.sh tune examples/scenario.sh ${INSTALL_ROOT}/confs
```

## Evaluation signals

- tuning_report.csv contains exactly one timing row per enumerated configuration
- The configuration written to best_configuration.json is the one with the minimum measured runtime in tuning_report.csv
- best_configuration.json specifies all three dimensions: work-group size, local memory usage, and serial per-thread work
- amber.sh test on the generated configuration files reports they are correct
- The tuned kernel meets the real-time budget for the described scenario (e.g. ARTS: 12 of 480 input beams per node in real time)

## Limitations

- The tuning script deletes previously generated configuration files in the same output directory.
- Timings and the selected configuration are specific to the chosen GPU platform and scenario; re-tuning is needed for other hardware or observational scenarios.
- Exhaustive enumeration times all valid configurations, so tuning cost grows with the size of the configuration space.

## Evidence

- [abstract] The OpenCL kernels require many low-level parameters, such as the OpenCL work-group size, the usage of local memory, or the amount of serial processing per thread: "The OpenCL kernels require many low-level parameters, such as the OpenCL work-group size, the usage of local memory, or the amount of serial processing per thread"
- [abstract] all of AMBER's computational kernels can be tuned: "all of AMBER’s computational kernels can be tuned to achieve the best possible performance on the hardware the pipeline is executed on"
- [readme] README tune mode: [evidence span withheld — anchor status: external_doc]
- [readme] README config generation caveat: [evidence span withheld — anchor status: external_doc]
