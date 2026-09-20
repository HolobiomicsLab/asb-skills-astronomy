---
name: astrophysical-burning-network-benchmarking
description: Use when a paper reports accelerator speedups for ported reaction-network
  kernels (e.g., Tesla P100 vs. a single Power8 core) and the values must be extracted
  per network and per grid size into a comparison table.
when_to_use_negative:
- The goal is measuring speedups yourself on new hardware or with new code — this
  skill only transcribes and structures values already reported in the paper's Figure
  5
- The figure/caption does not decompose speedups by both reaction network and grid
  size, so the required 'grid_sizes' column cannot be populated
- The analysis concerns Maestro/Castro hydrodynamics or AMReX AMR features generally
  rather than the thermonuclear reaction-network GPU benchmark
tools:
- name: Castro
  role: Simulation code whose highly compressible-flow/explosion modeling context
    the CUDA-ported reaction networks are benchmarked within
- name: Maestro
  role: Companion code in the AMReX astrophysics suite whose GPU/many-core porting
    strategy the Figure 5 networks exemplify
- name: AMReX
  role: Block-structured AMR framework underlying both codes, providing the GPU acceleration
    backends (CUDA) used by the ported networks
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:b3917b422433f9d0992149a4602d3e33cf8839c111f92ffd25bf6755215cffd3
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
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# astrophysical-burning-network-benchmarking

## Summary

Reproduces and tabulates the GPU-versus-single-CPU-core speedups reported for the CUDA-ported astrophysical reaction networks (3-isotope carbon, 9-isotope A=23 Urca, 13-isotope alpha chain) across grid sizes 16^3 to 128^3 on Summitdev, as shown in Figure 5 of the Castro/Maestro/AMReX paper.

## When to use

Use when a paper reports accelerator speedups for ported reaction-network kernels (e.g., Tesla P100 vs. a single Power8 core) and the values must be extracted per network and per grid size into a comparison table.

## When NOT to use

- The goal is measuring speedups yourself on new hardware or with new code — this skill only transcribes and structures values already reported in the paper's Figure 5
- The figure/caption does not decompose speedups by both reaction network and grid size, so the required 'grid_sizes' column cannot be populated
- The analysis concerns Maestro/Castro hydrodynamics or AMReX AMR features generally rather than the thermonuclear reaction-network GPU benchmark

## Inputs

- Figure 5 (and its caption) from the Castro/Maestro/AMReX suite paper (DOI 10.1088/1742-6596/1031/1/012024)
- Benchmark configuration: three reaction networks (13-isotope alpha chain, 9-isotope A=23 Urca, 3-isotope carbon) on Summitdev (Tesla P100 GPU vs. single Power8 CPU core) at grid sizes 16^3, 32^3, 64^3, 128^3

## Outputs

- figure5_gpu_speedup.csv with exactly 3 rows (one per network), a 'grid_sizes' column of per-grid-size speedups, and a 'limitations' column

## How to apply

Locate the benchmark figure (Figure 5) and its caption, which reports GPU-versus-single-Power8-core speedup for the reaction networks run on Summitdev with a Tesla P100 GPU. For each of the three networks — the 13-isotope alpha chain, the 9-isotope A=23 Urca network, and the 3-isotope carbon network — read the speedup at each tested grid size (16^3, 32^3, 64^3, 128^3). Record the reported scaling limitations alongside the numbers: per-thread local stack memory exhaustion for the larger networks and thread divergence arising from differing zone conditions. Serialize the result as figure5_gpu_speedup.csv with one row per network, a 'grid_sizes' column holding speedups per grid size, and a 'limitations' column.

## Related tools

- **Castro** (Simulation code whose highly compressible-flow/explosion modeling context the CUDA-ported reaction networks are benchmarked within)
- **Maestro** (Companion code in the AMReX astrophysics suite whose GPU/many-core porting strategy the Figure 5 networks exemplify)
- **AMReX** (Block-structured AMR framework underlying both codes, providing the GPU acceleration backends (CUDA) used by the ported networks) — https://github.com/AMReX-Codes/amrex

## Evaluation signals

- figure5_gpu_speedup.csv exists and contains exactly 3 rows, one per reaction network (3-isotope carbon, 9-isotope Urca, 13-isotope alpha chain)
- Each row contains the 'grid_sizes' field populated with speedups for all four tested grid sizes (16^3, 32^3, 64^3, 128^3)
- The 'limitations' column includes the substring 'stack memory' (per-thread local stack memory exhaustion for larger networks)
- The 'limitations' column includes the substring 'thread divergence' (arising from differing zone conditions)
- Hardware/context labels in the table are consistent with the paper: Tesla P100 GPU vs. single Power8 CPU core on Summitdev

## Limitations

- Per-thread local stack memory exhaustion limits the larger reaction networks (e.g., the 13-isotope alpha chain) on the GPU
- Thread divergence arises when zones in the same launch have differing thermodynamic/composition conditions, degrading GPU scaling
- Results are specific to Summitdev hardware (Tesla P100 GPU vs. single Power8 CPU core); the paper describes the porting approach as targeting 'current and future many-core and GPU-based architectures', so absolute speedups may not transfer
- BoxLib, the predecessor framework, is deprecated and superseded by AMReX, so reproducing with legacy BoxLib builds is not advised

## Evidence

- [abstract] Porting approach for GPU/many-core architectures: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
- [intro] Thermonuclear reactions as part of the modeled physics: "Accurate modeling of these events requires the coupling of hydrodynamics, gravity, thermonuclear reactions, and in some cases, radiation and magnetic fields."
- [readme] AMReX GPU acceleration backends used by the ported networks: [evidence span withheld — anchor status: external_doc]
- [other] BoxLib deprecation affects reproduction on legacy builds: [evidence span withheld — anchor status: external_doc]
