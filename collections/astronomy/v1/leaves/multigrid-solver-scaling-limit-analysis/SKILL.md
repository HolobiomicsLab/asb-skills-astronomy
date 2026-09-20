---
name: multigrid-solver-scaling-limit-analysis
description: Use when strong-scaling curves from AMReX-based simulations (Maestro/Castro)
  flatten or show non-monotonic upturns, and the goal is to attribute the behavior
  to the multigrid nodal solver or to hybrid MPI/OpenMP NUMA placement changes.
when_to_use_negative:
- The simulation is highly compressible and should run with Castro rather than Maestro,
  which is tuned for subsonic convective flows
- The scaling data are weak-scaling or per-node throughput curves with no MPI/OpenMP
  thread-configuration change to attribute artifacts to
- The goal is GPU- or many-core portability assessment of AMReX rather than MPI/NUMA
  strong-scaling analysis of the nodal multigrid solver
tools:
- name: Maestro
  role: Simulation code whose 3-d XRB strong-scaling behavior on Edison and Titan
    is analyzed; tuned for subsonic convective flows
- name: AMReX
  role: Underlying block-structured AMR framework providing the nodal-data and hybrid
    MPI/OpenMP parallelization whose multigrid nodal solver limits scaling
  repo: github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:63b02ca9d02c382d8ec1e96c553b0bed2ee3ede3287a86da6ebb75f3bfe0df27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AMReX
  - Maestro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multigrid-solver-scaling-limit-analysis

## Summary

Diagnoses the scaling ceiling of AMReX-based codes (e.g., Maestro on the 3-d X-ray burst problem) by isolating the multigrid nodal-projection solver as the dominant bottleneck at O(10^4) processors and attributing curve artifacts to MPI/OpenMP thread-configuration switches on NUMA nodes.

## When to use

Use when strong-scaling curves from AMReX-based simulations (Maestro/Castro) flatten or show non-monotonic upturns, and the goal is to attribute the behavior to the multigrid nodal solver or to hybrid MPI/OpenMP NUMA placement changes.

## When NOT to use

- The simulation is highly compressible and should run with Castro rather than Maestro, which is tuned for subsonic convective flows
- The scaling data are weak-scaling or per-node throughput curves with no MPI/OpenMP thread-configuration change to attribute artifacts to
- The goal is GPU- or many-core portability assessment of AMReX rather than MPI/NUMA strong-scaling analysis of the nodal multigrid solver

## Inputs

- Strong-scaling performance data or published figures (e.g., Figure 2) for Maestro on the 3-d X-ray burst (XRB) problem
- Machine configuration metadata: MPI-task-per-NUMA-node and OpenMP-thread-per-task counts (e.g., 1 MPI / 8 threads vs 1 MPI / 16 threads)
- Article sections reporting the scaling study (e.g., Section 4) and the stated limiting solver behavior

## Outputs

- CSV table (e.g., maestro_strong_scaling.csv) with columns code, problem, machine, scaling_extent, limiting_factor, thread_configuration — one row per machine
- Attribution statement mapping curve artifacts (Titan upturn) to thread-configuration switches and identifying the multigrid nodal solver as the scaling limitation

## How to apply

Locate the strong-scaling figure and its section (e.g., Figure 2 and Section 4 of the article reporting Maestro's 3-d XRB study) and extract, per machine (NERSC Edison, OLCF Titan), the extent of excellent scaling — here O(10^4) processors. Identify configuration switches that coincide with curve features: on Titan, the upturn occurs when moving from 1 MPI task / 8 OpenMP threads per NUMA node to 1 MPI task / 16 threads per node, so attribute the upturn to thread configuration rather than solver physics. Then record the algorithmic limiting factor — the multigrid nodal-projection solver's scaling behavior — since nodal (as opposed to cell-centered) data and elliptic solves degrade fastest as per-rank work shrinks. Tabulate one row per machine with code, problem, machine, scaling_extent, limiting_factor, and thread_configuration, and validate that the processor count is of order 10^4. Rationale: AMReX supports hybrid MPI/OpenMP and nodal data explicitly, so both NUMA-aware MPI/OMP ratios and the nodal multigrid solver are the natural suspects for end-of-curve artifacts.

## Related tools

- **Maestro** (Simulation code whose 3-d XRB strong-scaling behavior on Edison and Titan is analyzed; tuned for subsonic convective flows)
- **AMReX** (Underlying block-structured AMR framework providing the nodal-data and hybrid MPI/OpenMP parallelization whose multigrid nodal solver limits scaling) — github.com/AMReX-Codes/amrex

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('maestro_strong_scaling.csv'); assert len(df)==2 and df['scaling_extent'].notna().all() and df['limiting_factor'].notna().all()"
```

## Evaluation signals

- Output CSV exists with exactly 2 machine rows (Edison, Titan)
- Both rows contain non-empty scaling_extent and limiting_factor fields
- The reported scaling extent is of order 10^4 processors on both machines
- The Titan row's thread_configuration captures the switch from 1 MPI / 8 OpenMP threads to 1 MPI / 16 threads per NUMA node
- The limiting_factor field names the multigrid nodal-projection solver, matching the article's stated bottleneck

## Limitations

- The upturn attribution depends on knowing the exact NUMA/thread configuration change; without machine-configuration metadata the artifact cannot be assigned
- The finding is specific to the 3-d XRB problem, Edison and Titan, and the era's architecture (many-core CPU); future GPU-based architectures are treated separately in the paper
- Only the multigrid nodal-projection solver is identified as limiting; other kernels' scaling behavior is not decomposed by this analysis

## Evidence

- [readme] AMReX supports hybrid MPI/OpenMP and nodal multigrid data: [evidence span withheld — anchor status: external_doc]
- [readme] AMReX parallelization modes underlie the NUMA thread-configuration analysis: [evidence span withheld — anchor status: external_doc]
- [abstract] Maestro is the subsonic-flow code whose scaling is analyzed: "Maestro is tuned to efficiently model subsonic convective flows while Castro models the highly compressible flows associated with stellar explosions."
- [abstract] The paper's motivation is performance on many-core architectures: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
