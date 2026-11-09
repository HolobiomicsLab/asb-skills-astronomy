---
name: hardware-configuration-metadata-extraction
description: Use when a paper reports benchmark or performance runs on a named many-core/GPU
  platform (e.g., NERSC Cori Xeon Phi) and you must turn prose like 'all runs used
  4 hardware threads per core' into a validated machine-readable record.
when_to_use_negative:
- The document reports performance on commodity hardware without a named many-core
  platform, thread configuration, or architecture-specific optimization statement
- The optimization status cannot be traced to an explicit authorial statement — do
  not guess 'not optimized' from timing plots
- The input is already a structured performance table or JSON record; no prose extraction
  is needed
tools:
- name: Castro
  role: Simulation code whose benchmark runs (wdmerger self-gravity problem on Cori
    Xeon Phi) supply the hardware-configuration and optimization-status fields being
    extracted
- name: AMReX
  role: Underlying AMR framework whose architecture-portability goals (many-core and
    GPU backends) motivate recording per-architecture run metadata
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:5575e0012c8e1b297a7517a6f505e3c4b1290363a74ae555dd121ab2368d8ab0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Castro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# hardware-configuration-metadata-extraction

## Summary

Extracts structured hardware-configuration metadata (platform, core count, threads per core, machine name, optimization status) from the performance-evaluation sections of HPC simulation papers, as illustrated by the Castro wdmerger runs on NERSC Cori's Intel Xeon Phi (68 cores, 4 hardware threads per core, 272 threads per chip).

## When to use

Use when a paper reports benchmark or performance runs on a named many-core/GPU platform (e.g., NERSC Cori Xeon Phi) and you must turn prose like 'all runs used 4 hardware threads per core' into a validated machine-readable record.

## When NOT to use

- The document reports performance on commodity hardware without a named many-core platform, thread configuration, or architecture-specific optimization statement
- The optimization status cannot be traced to an explicit authorial statement — do not guess 'not optimized' from timing plots
- The input is already a structured performance table or JSON record; no prose extraction is needed

## Inputs

- Article section text describing many-core performance runs (e.g., Cori Intel Xeon Phi results for Castro's wdmerger self-gravity problem)
- Stated hardware parameters: core_count=68, threads_per_core=4, total_threads_per_chip=272, machine='Cori'
- Authorial statements about optimization status for the architecture

## Outputs

- castro_xeon_phi_record.json — a structured record with platform, machine, core_count, threads_per_core, total_threads_per_chip, code, problem, and optimization_status fields

## How to apply

Locate the many-core architecture performance discussion in the article text (for this paper, the section reporting Castro runs of the wdmerger self-gravity problem on the Intel Xeon Phi platform of NERSC Cori). Extract the hardware fields exactly as stated: platform='Intel Xeon Phi', core_count=68, threads_per_core=4, total_threads_per_chip=272, machine='Cori'. Extract the run-context fields: code='Castro', problem='wdmerger' self-gravity test, and set optimization_status='not yet optimized' only from an explicit authorial statement that the code has not yet been optimized for that architecture — never infer it from timings alone. Serialize all fields into castro_xeon_phi_record.json, then validate: the JSON parses, core_count equals 68 and threads_per_core equals 4 (value_in_range checks), total_threads_per_chip is consistent with core_count × threads_per_core (68 × 4 = 272), and optimization_status contains 'not yet' / 'not optimized'. The rationale is that the paper explicitly frames architecture portability ('performant on current and future many-core and GPU-based architectures') as a design goal, so hardware-configuration fields are first-class metadata that must be preserved alongside the performance numbers.

## Related tools

- **Castro** (Simulation code whose benchmark runs (wdmerger self-gravity problem on Cori Xeon Phi) supply the hardware-configuration and optimization-status fields being extracted)
- **AMReX** (Underlying AMR framework whose architecture-portability goals (many-core and GPU backends) motivate recording per-architecture run metadata) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import json; rec={'platform':'Intel Xeon Phi','machine':'Cori','core_count':68,'threads_per_core':4,'total_threads_per_chip':272,'code':'Castro','problem':'wdmerger','optimization_status':'not yet optimized'}; json.dump(rec,open('castro_xeon_phi_record.json','w'))"
```

## Evaluation signals

- castro_xeon_phi_record.json exists and parses as valid JSON
- core_count field equals 68 (value_in_range check)
- threads_per_core field equals 4 (value_in_range check)
- total_threads_per_chip (272) equals core_count × threads_per_core (68 × 4), an internal consistency invariant
- optimization_status string contains the substring 'not yet' or 'not optimized', matching the paper's explicit statement

## Limitations

- The extracted configuration (68 cores, 4 threads/core, 272 threads/chip) is specific to the NERSC Cori Intel Xeon Phi nodes and must not be reused for other machines
- The paper explicitly states Castro has not yet been optimized for the Intel Xeon Phi architecture, so recorded performance numbers are not representative of the code's optimized potential
- Only explicit authorial statements about optimization status qualify; inferring status from raw timings risks over-claiming

## Evidence

- [abstract] Architecture-portability framing motivating hardware metadata extraction: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
- [readme] Parallelization modes AMReX supports and which must be recorded per run: [evidence span withheld — anchor status: external_doc]
- [readme] Heterogeneous backend context for per-architecture run records: [evidence span withheld — anchor status: external_doc]
