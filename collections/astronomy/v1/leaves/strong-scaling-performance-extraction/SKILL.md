---
name: strong-scaling-performance-extraction
description: Use when a paper reports strong-scaling results (e.g., Figure 3 of the
  Castro paper showing Sedov and wdmerger on OLCF Titan) and you need to extract per-problem
  core-count behavior, threading mix, and compilers into a table.
when_to_use_negative:
- The figure reports weak-scaling, GPU/many-core throughput, or single-node timings
  rather than MPI strong scaling across core counts
- You have raw wall-clock timing dumps from your own runs — recompute scaling efficiencies
  directly instead of transcribing published figure claims
- The target problems use subsonic, low-Mach methods (e.g., Maestro's convective flows)
  whose performance characteristics differ from Castro's compressible explosion problems
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Castro
  role: 'Source of the strong-scaling results: Sedov (pure hydro, real EOS) and wdmerger
    (hydro+Poisson with factor-4 AMR) runs on OLCF Titan'
- name: AMReX
  role: Underlying block-structured AMR framework supporting the flat MPI, OpenMP,
    and hybrid MPI/OpenMP parallelization modes recorded in the threading_mix field
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:0a734e3c56fb0d0ffc036b29ee1cb5cd18acf95aac6a7b1fa9b16983b14c3a98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Castro
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# strong-scaling-performance-extraction

## Summary

Transcribes published strong-scaling behavior of simulation codes from performance figures into a structured per-problem table, capturing maximum core counts, MPI/OpenMP threading mix, and compiler versions. Grounded in Castro's Figure 3 Sedov and wdmerger scaling runs on OLCF Titan.

## When to use

Use when a paper reports strong-scaling results (e.g., Figure 3 of the Castro paper showing Sedov and wdmerger on OLCF Titan) and you need to extract per-problem core-count behavior, threading mix, and compilers into a table.

## When NOT to use

- The figure reports weak-scaling, GPU/many-core throughput, or single-node timings rather than MPI strong scaling across core counts
- You have raw wall-clock timing dumps from your own runs — recompute scaling efficiencies directly instead of transcribing published figure claims
- The target problems use subsonic, low-Mach methods (e.g., Maestro's convective flows) whose performance characteristics differ from Castro's compressible explosion problems

## Inputs

- Performance/scaling figure from the article (Castro Figure 3, Sedov and wdmerger problems)
- Reported hybrid MPI/OpenMP execution findings per problem
- Compiler versions used for the scaling runs (PGI 17.7, Cray 8.5.7)

## Outputs

- castro_strong_scaling.csv: a two-row table with fields for problem name, scaling behavior/maximum core count (including the 'O(10^5)' note), threading_mix, and compiler

## How to apply

Read the performance figure (here Figure 3 of the Castro paper) and transcribe the strong-scaling behavior per test problem: the pure-hydro Sedov run with a real equation of state, which scales excellently to O(10^5) cores on OLCF Titan, and the wdmerger hydro+Poisson run, which scales strongly but degrades at high core counts under factor-4 AMR refinement. Record the hybrid-execution finding — that fewer MPI ranks with more OpenMP threads per rank gives the best performance for multilevel AMR problems — as a threading_mix entry per problem. Capture the build environment by adding a 'compiler' field with the versions used for the runs (PGI 17.7 and Cray 8.5.7). Assemble one row per problem into castro_strong_scaling.csv with fields for problem name, scaling behavior/maximum core count (preserving the order-of-magnitude 'O(10^5)' note), threading mix, and compiler, then validate row count, presence of the compiler field, and the 'O(10^5)' substring.

## Related tools

- **Castro** (Source of the strong-scaling results: Sedov (pure hydro, real EOS) and wdmerger (hydro+Poisson with factor-4 AMR) runs on OLCF Titan)
- **AMReX** (Underlying block-structured AMR framework supporting the flat MPI, OpenMP, and hybrid MPI/OpenMP parallelization modes recorded in the threading_mix field) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import pandas as pd; df=pd.DataFrame(rows); df.to_csv('castro_strong_scaling.csv', index=False); assert len(df)==2 and 'O(10^5)' in open('castro_strong_scaling.csv').read()"
```

## Evaluation signals

- Output file castro_strong_scaling.csv exists and row_count equals 2 problem rows (Sedov and wdmerger)
- File content contains the substring 'O(10^5)' for the Sedov maximum core count
- The 'compiler' field is present and populated in every row (PGI 17.7 and Cray 8.5.7)
- Each row carries a threading_mix entry reflecting the fewer-MPI-ranks/more-OpenMP-threads finding for multilevel AMR
- The wdmerger row notes degraded strong scaling at high core counts under factor-4 AMR refinement

## Limitations

- The 'O(10^5)' maximum core count is an order-of-magnitude figure transcribed from a plot, not an exact rank count
- wdmerger performance degrades at high core counts under factor-4 AMR refinement, so scaling conclusions do not transfer uniformly across problems
- Results are specific to OLCF Titan with PGI 17.7 and Cray 8.5.7 compilers; they may not generalize to other architectures or compiler stacks

## Evidence

- [results] sedov_scaling_finding: [evidence span withheld — anchor status: not_found]
- [readme] amrex_parallelization_modes: [evidence span withheld — anchor status: external_doc]
- [abstract] performance_goal: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
