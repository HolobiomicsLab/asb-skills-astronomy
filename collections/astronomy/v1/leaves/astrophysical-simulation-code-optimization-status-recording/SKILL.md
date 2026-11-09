---
name: astrophysical-simulation-code-optimization-status-recording
description: Use when reproducing or reporting a performance figure for an AMReX-based
  astrophysics code on a specific platform (e.g., Castro's wdmerger self-gravity test
  on NERSC Cori Intel Xeon Phi) and the hardware configuration plus stated optimization
  status must be recorded as a machine-readable artifact.
when_to_use_negative:
- The source text does not explicitly state an optimization status or hardware thread
  configuration for the architecture — do not infer 'optimized' or 'not optimized'
  from scaling curves alone.
- The goal is to benchmark or tune the code on the architecture rather than record
  the paper's reported run configuration and status.
- The performance result is for a different code/problem pairing (e.g., Maestro subsonic
  convection instead of the Castro wdmerger self-gravity problem) — a separate record
  should be created rather than overwriting this one.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Castro
  role: The simulation code whose run configuration (wdmerger self-gravity problem)
    and optimization status on the Intel Xeon Phi architecture are being recorded.
- name: AMReX
  role: The block-structured AMR library underlying Castro; its parallelization and
    GPU-backend options are the context for how the code is configured and run on
    many-core and GPU-based architectures.
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:d41d895a471ca07593d3256ca21bcddb3d287b3a5d1b7fb7984daa5bd7343864
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Castro
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# astrophysical-simulation-code-optimization-status-recording

## Summary

Captures a structured performance-record (JSON) documenting how an astrophysical simulation code (e.g., Castro) was run and configured on a specific many-core or GPU architecture, together with the authors' explicit optimization status for that architecture.

## When to use

Use when reproducing or reporting a performance figure for an AMReX-based astrophysics code on a specific platform (e.g., Castro's wdmerger self-gravity test on NERSC Cori Intel Xeon Phi) and the hardware configuration plus stated optimization status must be recorded as a machine-readable artifact.

## When NOT to use

- The source text does not explicitly state an optimization status or hardware thread configuration for the architecture — do not infer 'optimized' or 'not optimized' from scaling curves alone.
- The goal is to benchmark or tune the code on the architecture rather than record the paper's reported run configuration and status.
- The performance result is for a different code/problem pairing (e.g., Maestro subsonic convection instead of the Castro wdmerger self-gravity problem) — a separate record should be created rather than overwriting this one.

## Inputs

- Article section text reporting many-core architecture performance runs (e.g., Figure 4, right: Castro wdmerger on NERSC Cori Xeon Phi)
- Hardware configuration values stated in the text (platform, machine, core_count, threads_per_core, total_threads_per_chip)
- Authors' explicit optimization-status statement for the architecture

## Outputs

- castro_xeon_phi_record.json — a parsed-JSON performance record containing platform, machine, core_count, threads_per_core, total_threads_per_chip, code, problem, and optimization_status fields

## How to apply

Locate the many-core architecture performance discussion in the article text that reports the code's runs on the target platform (here, Castro on Intel Xeon Phi on NERSC Cori, using the same wdmerger self-gravity problem as the Titan test). Extract the hardware configuration fields — platform = 'Intel Xeon Phi', machine = 'Cori', core_count = 68, threads_per_core = 4, total_threads_per_chip = 272 — and the run context fields (code = 'Castro', problem = 'wdmerger'). Set optimization_status from the paper's explicit statement, here 'not yet optimized', since the authors state they have not yet focused on optimizing Castro for the Intel Xeon Phi architecture. Serialize all fields into castro_xeon_phi_record.json and validate that the file parses, the numeric fields pass value_in_range checks (core_count == 68, threads_per_core == 4), and the optimization_status string contains 'not yet' / 'not optimized'. The rationale is that hardware thread placement (4 hardware threads per core on a 68-core chip) and optimization status materially change how weak/strong-scaling results such as Figure 4 (right) should be interpreted, so both must be captured alongside the performance data.

## Related tools

- **Castro** (The simulation code whose run configuration (wdmerger self-gravity problem) and optimization status on the Intel Xeon Phi architecture are being recorded.)
- **AMReX** (The block-structured AMR library underlying Castro; its parallelization and GPU-backend options are the context for how the code is configured and run on many-core and GPU-based architectures.) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import json; json.dump({'platform':'Intel Xeon Phi','machine':'Cori','core_count':68,'threads_per_core':4,'total_threads_per_chip':272,'code':'Castro','problem':'wdmerger','optimization_status':'not yet optimized'}, open('castro_xeon_phi_record.json','w'))"
```

## Evaluation signals

- castro_xeon_phi_record.json exists and parses as valid JSON.
- core_count field equals 68 (value_in_range check).
- threads_per_core field equals 4 (value_in_range check).
- optimization_status field contains the substring 'not yet' or 'not optimized'.
- Internal consistency: core_count × threads_per_core (68 × 4) equals the stated total_threads_per_chip value of 272.

## Limitations

- The recorded optimization status ('not yet optimized') reflects the authors' statement at publication time; it is time-sensitive and does not describe later versions of Castro or AMReX.
- Reported scaling results for this configuration are baseline numbers — the authors explicitly state they have not yet focused on optimizing Castro for the Intel Xeon Phi architecture, so the record should not be cited as evidence of achieved architectural optimization.
- Hardware configuration values (68 cores, 4 threads per core, 272 threads per chip) are specific to the Cori Xeon Phi system and must be re-extracted if the runs are reproduced on a different machine.

## Evidence

- [abstract] Authors' stated aim for many-core/GPU performance: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
- [abstract] Castro's role in the suite (the code whose run configuration is recorded): "Maestro is tuned to efficiently model subsonic convective flows while Castro models the highly compressible flows associated with stellar explosions."
- [readme] AMReX parallelization options relevant to hardware thread placement: [evidence span withheld — anchor status: external_doc]
- [readme] AMReX GPU backend support for heterogeneous architectures: [evidence span withheld — anchor status: external_doc]
