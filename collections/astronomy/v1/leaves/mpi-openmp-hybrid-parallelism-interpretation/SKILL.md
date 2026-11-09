---
name: mpi-openmp-hybrid-parallelism-interpretation
description: Use when interpreting or transcribing strong-scaling performance results
  (core counts, threading mixes, compilers) for hybrid MPI/OpenMP runs of AMReX-based
  codes like Castro on leadership-class machines such as OLCF Titan.
when_to_use_negative:
- The input is raw timing data from runs you executed yourself rather than reported
  scaling behavior to transcribe from a publication
- The scaling study concerns GPU-accelerated execution (CUDA/HIP/SYCL) rather than
  the MPI/OpenMP hybrid CPU parallelism reported here
- The question is about algorithmic capabilities of the codes (e.g., subsonic vs.
  compressible flow modeling) rather than parallel performance
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: Castro
  role: The AMReX-based compressible astrophysics code whose reported strong-scaling
    results (Sedov, wdmerger on OLCF Titan) are interpreted and transcribed by this
    skill
- name: AMReX
  role: Block-structured AMR framework underlying Castro that supports the parallelization
    modes (flat MPI, OpenMP, hybrid MPI/OpenMP) whose scaling behavior is being interpreted
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:cae950aa2d57c14c8cff22aa9aeae5df490327b90199a54389674a18790849f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Castro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# MPI/OpenMP Hybrid Parallelism Interpretation

## Summary

Interprets published strong-scaling results for MPI/OpenMP hybrid execution of AMReX-based codes (Castro), extracting scaling behavior, maximum core counts, threading mixes, and compiler stacks into a structured record. It captures the reported finding that fewer MPI ranks with more OpenMP threads per rank performs best for multilevel AMR problems.

## When to use

Use when interpreting or transcribing strong-scaling performance results (core counts, threading mixes, compilers) for hybrid MPI/OpenMP runs of AMReX-based codes like Castro on leadership-class machines such as OLCF Titan.

## When NOT to use

- The input is raw timing data from runs you executed yourself rather than reported scaling behavior to transcribe from a publication
- The scaling study concerns GPU-accelerated execution (CUDA/HIP/SYCL) rather than the MPI/OpenMP hybrid CPU parallelism reported here
- The question is about algorithmic capabilities of the codes (e.g., subsonic vs. compressible flow modeling) rather than parallel performance

## Inputs

- Published strong-scaling figure or table (e.g., Figure 3 of the Castro paper) covering the Sedov (real EOS) and wdmerger (hydro+Poisson) problems
- Problem names and configuration notes (e.g., factor-4 AMR refinement for wdmerger)
- Reported threading configurations (MPI ranks vs. OpenMP threads per rank)
- Compiler versions used for the scaling runs (PGI 17.7, Cray 8.5.7)

## Outputs

- castro_strong_scaling.csv — a two-row table with fields: problem name, scaling behavior / maximum core count (including the 'O(10^5)' note), threading mix, and compiler
- Per-problem threading_mix entries capturing the MPI-rank/OpenMP-thread balance finding

## How to apply

Read the reported scaling figure (Figure 3 of the Castro paper) and transcribe the strong-scaling behavior of each test problem separately: the pure-hydro Sedov run with a real equation of state, which scales excellently up to O(10^5) cores on OLCF Titan, and the wdmerger hydro+Poisson run, which scales well but degrades at high core counts under factor-4 AMR refinement. For each problem, record the threading_mix as a distinct entry — the paper reports that running with fewer MPI ranks and more OpenMP threads per rank gives the best performance for multilevel AMR problems. Record the compilers used for the scaling runs (PGI 17.7 and Cray 8.5.7) in a 'compiler' field, since hybrid performance conclusions are compiler-stack dependent. Assemble the values into a two-row table with fields for problem name, scaling behavior/maximum core count, threading mix, and compiler, and write it to castro_strong_scaling.csv. This skill is interpretive: it extracts and structures reported results rather than re-running the benchmarks.

## Related tools

- **Castro** (The AMReX-based compressible astrophysics code whose reported strong-scaling results (Sedov, wdmerger on OLCF Titan) are interpreted and transcribed by this skill)
- **AMReX** (Block-structured AMR framework underlying Castro that supports the parallelization modes (flat MPI, OpenMP, hybrid MPI/OpenMP) whose scaling behavior is being interpreted) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('castro_strong_scaling.csv'); assert len(df)==2 and 'compiler' in df.columns and 'O(10^5)' in open('castro_strong_scaling.csv').read()"
```

## Evaluation signals

- The output file castro_strong_scaling.csv exists and row_count equals 2 problem rows (Sedov and wdmerger)
- The 'compiler' field is present in every row and contains the reported versions (PGI 17.7, Cray 8.5.7)
- The file content contains the substring 'O(10^5)' for the Sedov maximum core count
- Each row carries a threading_mix entry reflecting the fewer-MPI-ranks/more-OpenMP-threads finding for multilevel AMR problems
- Every recorded value is traceable to the reported Figure 3 scaling behavior rather than extrapolated

## Limitations

- The wdmerger hydro+Poisson run shows degraded performance at high core counts under factor-4 AMR refinement, so the 'excellent scaling' conclusion cannot be generalized across problem types
- The MPI/OpenMP threading conclusion (fewer ranks, more threads) is stated specifically for multilevel AMR problems and may not transfer to flat-grid or pure-hydro runs
- The results are tied to the OLCF Titan hardware and the specific compiler stack (PGI 17.7, Cray 8.5.7); performance conclusions may not hold on other architectures or compilers

## Evidence

- [readme] README parallelization modes: [evidence span withheld — anchor status: external_doc]
- [results] Castro scales to O(10^5) cores: [evidence span withheld — anchor status: not_found]
- [results] threading mix finding: [evidence span withheld — anchor status: not_found]
- [results] wdmerger high-core-count degradation: [evidence span withheld — anchor status: not_found]
- [abstract] many-core performance motivation: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
