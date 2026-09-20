---
name: amr-refinement-scaling-analysis
description: Use when a task asks to transcribe, reproduce, or validate reported strong-scaling
  behavior of block-structured AMR codes (Castro Sedov/wdmerger on OLCF Titan, Figure
  3) into a table with core counts, threading mix, and compiler fields.
when_to_use_negative:
- The task requires running new benchmark timings on a different machine or architecture
  (e.g. GPU backends like CUDA/HIP/SYCL) rather than transcribing the published CPU
  strong-scaling results from Figure 3 — the reported O(10^5)-core and threading findings
  are specific to OLCF Titan CPU runs
- The problem being analyzed is a pure-hydro case without multilevel AMR refinement,
  where the wdmerger degradation at high core counts and the hybrid-threading guidance
  do not apply
- The input is already a structured scaling table — this skill only converts figure/paper
  content into the CSV schema
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Castro
  role: The astrophysical hydrodynamics code whose Sedov and wdmerger test-problem
    strong-scaling results are transcribed and analyzed
- name: AMReX
  role: Block-structured AMR library underlying Castro; its supported parallelization
    modes (flat MPI, OpenMP, hybrid MPI/OpenMP) define the threading-mix axis recorded
    per problem
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:69380546829533cd49394943ca5a8e5a031cd2ae44537ecda159f18a3e659388
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
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# amr-refinement-scaling-analysis

## Summary

Transcribes and validates published CPU strong-scaling results for the Castro AMR code (Sedov with real EOS and wdmerger hydro+Poisson test problems) on OLCF Titan from Figure 3, capturing the O(10^5)-core Sedov result, per-problem MPI/OpenMP threading mix, and compiler versions in a structured two-row CSV table.

## When to use

Use when a task asks to transcribe, reproduce, or validate reported strong-scaling behavior of block-structured AMR codes (Castro Sedov/wdmerger on OLCF Titan, Figure 3) into a table with core counts, threading mix, and compiler fields.

## When NOT to use

- The task requires running new benchmark timings on a different machine or architecture (e.g. GPU backends like CUDA/HIP/SYCL) rather than transcribing the published CPU strong-scaling results from Figure 3 — the reported O(10^5)-core and threading findings are specific to OLCF Titan CPU runs
- The problem being analyzed is a pure-hydro case without multilevel AMR refinement, where the wdmerger degradation at high core counts and the hybrid-threading guidance do not apply
- The input is already a structured scaling table — this skill only converts figure/paper content into the CSV schema

## Inputs

- Figure 3 of the Castro paper (strong-scaling plots for Sedov and wdmerger on OLCF Titan)
- Reported MPI/OpenMP hybrid execution finding per test problem (threading mix for multilevel AMR)
- Compiler provenance for the scaling runs (PGI 17.7 and Cray 8.5.7)
- AMReX parallelization configuration (flat MPI, OpenMP, hybrid MPI/OpenMP)

## Outputs

- castro_strong_scaling.csv with exactly 2 problem rows and fields: problem name, scaling behavior/maximum core count (including the 'O(10^5)' note), threading_mix, and compiler

## How to apply

Read Figure 3 of the Castro paper and transcribe strong-scaling behavior for the two test problems separately: the pure-hydro Sedov run with a real equation of state, which shows excellent scaling up to O(10^5) cores, and the wdmerger hydro+Poisson run, which scales well but degrades at high core counts under a factor-4 AMR refinement. For each problem, record a threading_mix entry from the reported hybrid-execution finding — for multilevel AMR problems, running with fewer MPI ranks and more OpenMP threads per rank gives the best performance — because AMReX explicitly supports flat MPI, OpenMP, and hybrid MPI/OpenMP parallelization, making the thread/rank split a first-class axis of the analysis. Record the compilers used for the scaling runs (PGI 17.7 and Cray 8.5.7) in a dedicated 'compiler' field. Assemble the values into a two-row table with fields for problem name, scaling behavior/maximum core count (preserving the 'O(10^5)' note verbatim), threading mix, and compiler, then write it to castro_strong_scaling.csv.

## Related tools

- **Castro** (The astrophysical hydrodynamics code whose Sedov and wdmerger test-problem strong-scaling results are transcribed and analyzed)
- **AMReX** (Block-structured AMR library underlying Castro; its supported parallelization modes (flat MPI, OpenMP, hybrid MPI/OpenMP) define the threading-mix axis recorded per problem) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import csv; csv.writer(open('castro_strong_scaling.csv','w')).writerows([['problem','max_cores','threading_mix','compiler'],['Sedov','O(10^5)','excellent strong scaling','PGI 17.7'],['wdmerger','degrades at high cores under factor-4 AMR','fewer ranks + more OpenMP threads','Cray 8.5.7']])"
```

## Evaluation signals

- castro_strong_scaling.csv exists and row_count equals exactly 2 problem rows (Sedov and wdmerger)
- The file contains the verbatim substring 'O(10^5)' in the scaling/maximum-core-count field for the Sedov row
- A 'compiler' field is present and populated with the reported versions, PGI 17.7 and Cray 8.5.7
- Each row carries a threading_mix entry reflecting the reported finding that fewer MPI ranks with more OpenMP threads per rank gives best performance for multilevel AMR problems
- The wdmerger row records degraded performance at high core counts under a factor-4 AMR refinement, distinct from the Sedov row's excellent scaling

## Limitations

- The wdmerger strong-scaling result is problem-dependent: performance degrades at high core counts under factor-4 AMR refinement, so scaling claims cannot be generalized across test problems
- The hybrid MPI/OpenMP threading guidance was reported for CPU runs on OLCF Titan (PGI 17.7, Cray 8.5.7) and may not transfer to GPU-accelerated architectures the suite is now targeting
- The maximum core count is recorded qualitatively as O(10^5) rather than as an exact peak core number, limiting quantitative re-analysis

## Evidence

- [other] Castro exhibits excellent strong scaling for the pure hydro (Sedov with real EOS) problem, reaching O(10^5) cores on OLCF Titan.: [evidence span withheld — anchor status: not_found]
- [other] wdmerger degraded performance under factor-4 refinement: [evidence span withheld — anchor status: not_found]
- [other] hybrid threading finding for multilevel AMR: [evidence span withheld — anchor status: not_found]
- [readme] AMReX parallelization modes: [evidence span withheld — anchor status: external_doc]
- [readme] AMReX scalable AMR performance rationale: [evidence span withheld — anchor status: external_doc]
- [abstract] AMReX as the foundation for the codes being scaled: "Both are built on the block-structured adaptive mesh refinement library AMReX."
- [abstract] architecture-performant approach context: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
