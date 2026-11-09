---
name: mpi-thread-configuration-interpretation
description: Use when strong-scaling timing data shows an upturn or discontinuity
  near the maximum processor count, and the run mixes hybrid MPI/OpenMP configurations
  across NUMA nodes — as in the Maestro 3-d XRB runs on Titan and Edison.
when_to_use_negative:
- The scaling study is weak-scaling or a problem size other than the 3-d XRB strong-scaling
  setup reported in Figure 2 — the upturn interpretation is specific to that configuration
  switch.
- The runs use a single, fixed MPI/OpenMP configuration, so there is no thread-configuration
  change to attribute an upturn to.
- The target code is Castro (highly compressible explosion flows) rather than Maestro
  (subsonic convective flows); no such scaling interpretation is reported for Castro
  in this context.
tools:
- name: Maestro
  role: Simulation code whose strong-scaling behavior on the 3-d XRB problem is measured
    and interpreted
- name: AMReX
  role: Block-structured AMR library underlying Maestro; supports the hybrid MPI/OpenMP
    parallelization modes whose configuration changes explain the Titan upturn
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:d83c611f90c05799ee98bbf9951dbf0d8abb32cfb8b8326c38e0cecb5a947ddf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Maestro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# mpi-thread-configuration-interpretation

## Summary

Interprets strong-scaling curves for the Maestro 3-d X-ray burst (XRB) problem on OLCF Titan and NERSC Edison, attributing the end-of-curve performance upturn on Titan to a switch in MPI/OpenMP configuration (1 MPI task / 8 OpenMP threads per NUMA node to 1 MPI task / 16 threads) rather than algorithmic improvement.

## When to use

Use when strong-scaling timing data shows an upturn or discontinuity near the maximum processor count, and the run mixes hybrid MPI/OpenMP configurations across NUMA nodes — as in the Maestro 3-d XRB runs on Titan and Edison.

## When NOT to use

- The scaling study is weak-scaling or a problem size other than the 3-d XRB strong-scaling setup reported in Figure 2 — the upturn interpretation is specific to that configuration switch.
- The runs use a single, fixed MPI/OpenMP configuration, so there is no thread-configuration change to attribute an upturn to.
- The target code is Castro (highly compressible explosion flows) rather than Maestro (subsonic convective flows); no such scaling interpretation is reported for Castro in this context.

## Inputs

- Strong-scaling timing data for Maestro on the 3-d X-ray burst (XRB) problem (Figure 2, Section 4)
- Machine and NUMA/MPI/OpenMP configuration notes per run (e.g. 1 MPI task / 8 threads vs 1 MPI task / 16 threads per node on Titan)
- Target machines: NERSC Edison and OLCF Titan

## Outputs

- maestro_strong_scaling.csv with exactly 2 rows (Edison, Titan) and columns code, problem, machine, scaling_extent, limiting_factor, thread_configuration

## How to apply

Locate the strong-scaling study (Figure 2 and Section 4 of the article) for the Maestro 3-d XRB problem on each machine. Extract the scaling extent per machine — Maestro demonstrated excellent scaling up to O(10^4) processors on both Edison and Titan. Where the curve upturns (Titan), check the accompanying MPI/OpenMP configuration notes: the upturn coincides with moving from 1 MPI task with 8 OpenMP threads per NUMA node to 1 MPI task with 16 threads per node, so the improvement is a thread-configuration effect, not better algorithmic scaling. Record the stated scaling limiter — the multigrid nodal-projection solver's scaling behavior — rather than attributing limits to the hydrodynamics. Tabulate one row per machine into maestro_strong_scaling.csv with columns code, problem, machine, scaling_extent, limiting_factor, thread_configuration.

## Related tools

- **Maestro** (Simulation code whose strong-scaling behavior on the 3-d XRB problem is measured and interpreted)
- **AMReX** (Block-structured AMR library underlying Maestro; supports the hybrid MPI/OpenMP parallelization modes whose configuration changes explain the Titan upturn) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import pandas as pd; pd.DataFrame(rows, columns=['code','problem','machine','scaling_extent','limiting_factor','thread_configuration']).to_csv('maestro_strong_scaling.csv', index=False)"
```

## Evaluation signals

- maestro_strong_scaling.csv exists and contains exactly 2 machine rows (Edison and Titan).
- scaling_extent and limiting_factor fields are non-empty in both rows.
- The reported scaling extent is of order 10^4 processors for both machines.
- The Titan upturn is attributed to the switch from 1 MPI / 8-thread to 1 MPI / 16-thread NUMA configuration, not to solver improvements.
- The limiting_factor in both rows names the multigrid nodal-projection solver's scaling behavior.

## Limitations

- The end-of-curve upturn on Titan is a thread-configuration artifact (8 vs 16 OpenMP threads per NUMA node) and must not be read as improved algorithmic strong scaling.
- The multigrid nodal-projection solver is the main scaling limitation, so conclusions beyond O(10^4) processors are not supported.
- Interpretation is tied to the specific Edison and Titan NUMA topologies and may not transfer to other architectures.

## Evidence

- [abstract] Maestro role in the suite: "Maestro is tuned to efficiently model subsonic convective flows while Castro models the highly compressible flows associated with stellar explosions."
- [readme] Parallelization modes underlying the configuration interpretation: [evidence span withheld — anchor status: external_doc]
- [readme] Nodal data support relevant to the nodal multigrid solver limiter: [evidence span withheld — anchor status: external_doc]
