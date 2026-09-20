---
name: many-core-platform-performance-extraction
description: Use when an article reports benchmark runs of a code (e.g., Castro's
  wdmerger self-gravity problem) on a many-core architecture such as Intel Xeon Phi
  on NERSC Cori, and the threading configuration and optimization status must be captured
  as structured fields.
when_to_use_negative:
- The article reports GPU-based (CUDA/HIP/SYCL) performance rather than CPU many-core
  threading configuration — the hardware fields extracted here (cores per chip, threads
  per core) do not apply.
- The benchmark configuration is only visible in a scaling plot and no explicit hardware/optimization
  statement exists in the text — extracting unverified core/threads counts would fabricate
  fields.
- The goal is to reproduce or benchmark the code locally (e.g., building AMReX) rather
  than to document a published run's configuration.
tools:
- name: Castro
  role: Subject code whose many-core run configuration and optimization status are
    extracted; used for the wdmerger self-gravity benchmark on Cori Xeon Phi
- name: AMReX
  role: Underlying AMR framework on which Castro and Maestro are built; provides the
    parallelization and architecture support context (MPI/OpenMP, GPU backends) for
    the extracted run configuration
  repo: https://github.com/AMReX-Codes/amrex
- name: Maestro
  role: Companion code in the AMReX astrophysics suite whose architecture-portability
    approach frames the many-core performance discussion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:681e8bed929a6e0de9cbc3adce59748ea5df25e9aa9ef736797fd97764aeb572
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Castro
  - Maestro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# many-core-platform-performance-extraction

## Summary

Extracts hardware configuration (platform, core count, threads per core, total threads per chip, machine name) and the code's stated optimization status for a specific many-core platform from an article's performance discussion, serializing them as a structured JSON record for reproducible performance analysis.

## When to use

Use when an article reports benchmark runs of a code (e.g., Castro's wdmerger self-gravity problem) on a many-core architecture such as Intel Xeon Phi on NERSC Cori, and the threading configuration and optimization status must be captured as structured fields.

## When NOT to use

- The article reports GPU-based (CUDA/HIP/SYCL) performance rather than CPU many-core threading configuration — the hardware fields extracted here (cores per chip, threads per core) do not apply.
- The benchmark configuration is only visible in a scaling plot and no explicit hardware/optimization statement exists in the text — extracting unverified core/threads counts would fabricate fields.
- The goal is to reproduce or benchmark the code locally (e.g., building AMReX) rather than to document a published run's configuration.

## Inputs

- Article performance-discussion section text describing many-core benchmark runs
- Run identifiers from related figures (e.g., Figure 4, right panel: Cori Xeon Phi results)
- Named test problem and machine (Castro wdmerger self-gravity problem; NERSC Cori)

## Outputs

- castro_xeon_phi_record.json with fields: platform, core_count, threads_per_core, total_threads_per_chip, machine, code, problem, optimization_status
- Validated JSON record confirming core_count == 68, threads_per_core == 4, and optimization_status containing 'not yet' / 'not optimized'

## How to apply

Locate the many-core architecture performance discussion in the article text (for this paper, the section reporting Castro runs on Intel Xeon Phi, tied to the same wdmerger self-gravity problem used on Titan). Extract the hardware fields — platform = 'Intel Xeon Phi', core_count = 68, threads_per_core = 4, total_threads_per_chip = 272, machine = 'Cori' — and the run context (code = 'Castro', problem = 'wdmerger'). Record the authors' explicit optimization statement as optimization_status (here, Castro has not yet been optimized for Xeon Phi) rather than inferring it from scaling curves. Serialize all fields to a JSON record (e.g., castro_xeon_phi_record.json) and validate: the file parses, core_count equals 68, threads_per_core equals 4, and optimization_status contains 'not yet' / 'not optimized'. Rationale: the paper's framing is making the AMReX-based codes (Castro, Maestro) performant on current and future many-core and GPU-based architectures, so per-architecture configuration and optimization-status fields are the reusable unit of analysis.

## Related tools

- **Castro** (Subject code whose many-core run configuration and optimization status are extracted; used for the wdmerger self-gravity benchmark on Cori Xeon Phi)
- **AMReX** (Underlying AMR framework on which Castro and Maestro are built; provides the parallelization and architecture support context (MPI/OpenMP, GPU backends) for the extracted run configuration) — https://github.com/AMReX-Codes/amrex
- **Maestro** (Companion code in the AMReX astrophysics suite whose architecture-portability approach frames the many-core performance discussion)

## Examples

```
python -c "import json; r=json.load(open('castro_xeon_phi_record.json')); assert r['core_count']==68 and r['threads_per_core']==4; assert 'not yet' in r['optimization_status']"
```

## Evaluation signals

- castro_xeon_phi_record.json exists and parses as valid JSON.
- core_count field equals 68 and threads_per_core field equals 4 (value_in_range checks).
- optimization_status field contains the substring 'not yet' or 'not optimized', matching the authors' explicit statement.
- total_threads_per_chip is consistent with core_count × threads_per_core (68 × 4 = 272).
- code and problem fields correctly identify Castro and the wdmerger self-gravity problem shared with the Titan runs.

## Limitations

- The authors explicitly state they have not yet focused on optimizing Castro for the Intel Xeon Phi architecture, so extracted performance numbers for that platform do not reflect an optimized configuration and should not be read as best-case scaling.
- All Cori Xeon Phi runs used 4 hardware threads per core (272 threads per chip); results for other thread-per-core settings are absent and cannot be extrapolated from this record.
- Extracted optimization_status is a statement of author intent at publication time (2018); later code versions may have changed architecture support.

## Evidence

- [abstract] many-core and GPU-based architectures framing: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
- [abstract] Castro run context: "Castro models the highly compressible flows associated with stellar explosions"
- [readme] AMReX parallelization backends relevant to many-core runs: [evidence span withheld — anchor status: external_doc]
- [readme] AMReX scalable-performance rationale: [evidence span withheld — anchor status: external_doc]
