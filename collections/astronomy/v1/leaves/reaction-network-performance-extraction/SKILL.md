---
name: reaction-network-performance-extraction
description: Use when a paper or figure reports hardware speedups (e.g., Tesla P100
  GPU vs. a single Power8 core on Summitdev) for multiple reaction networks across
  several grid sizes, and the goal is to reproduce/serialize those numbers with their
  limitations.
when_to_use_negative:
- The source reports absolute wall-clock timings rather than GPU-vs-CPU speedup ratios
  across grid sizes — the skill's table schema (grid_sizes column of speedups) does
  not fit.
- You need to actually run the CUDA-ported reaction networks in Castro/Maestro to
  measure performance yourself, rather than extract reported values from a figure/caption.
- The figure covers a different paper's hardware setup (not Tesla P100 vs. single
  Power8 core on Summitdev) with different networks or grid sizes, requiring a re-designed
  table schema.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: Maestro
  role: Simulation code whose subsonic convective-flow reaction networks are among
    the benchmarks whose speedups Figure 5 reports
- name: Castro
  role: Simulation code for the highly compressible explosion phase whose CUDA-ported
    reaction networks were benchmarked for GPU speedup
- name: AMReX
  role: Block-structured AMR library providing the CUDA/GPU acceleration backend on
    which the reaction-network ports and their Summitdev benchmarks run
  repo: github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:3a9311dabe93b5e31f42ae8811e7bce829bd9d77985e0caf974222a609cb6757
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AMReX
  - Castro
  - Maestro
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# reaction-network-performance-extraction

## Summary

Extracts and tabulates reported GPU-versus-CPU speedups for the CUDA-ported astrophysical reaction networks (3-isotope carbon, 9-isotope A=23 Urca, 13-isotope alpha chain) from Figure 5 of the Castro/Maestro/AMReX paper, recording per-grid-size speedups and scaling limitations into a structured CSV.

## When to use

Use when a paper or figure reports hardware speedups (e.g., Tesla P100 GPU vs. a single Power8 core on Summitdev) for multiple reaction networks across several grid sizes, and the goal is to reproduce/serialize those numbers with their limitations.

## When NOT to use

- The source reports absolute wall-clock timings rather than GPU-vs-CPU speedup ratios across grid sizes — the skill's table schema (grid_sizes column of speedups) does not fit.
- You need to actually run the CUDA-ported reaction networks in Castro/Maestro to measure performance yourself, rather than extract reported values from a figure/caption.
- The figure covers a different paper's hardware setup (not Tesla P100 vs. single Power8 core on Summitdev) with different networks or grid sizes, requiring a re-designed table schema.

## Inputs

- Figure 5 and its caption from the paper 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions' (DOI 10.1088/1742-6596/1031/1/012024)
- Reported speedup values for 3-isotope carbon, 9-isotope A=23 Urca, and 13-isotope alpha chain networks at grid sizes 16^3, 32^3, 64^3, 128^3
- Documented scaling limitations (stack memory exhaustion, thread divergence) from the figure caption/text

## Outputs

- figure5_gpu_speedup.csv with exactly 3 rows (one per network), a 'grid_sizes' column of per-grid-size speedups, and a 'limitations' column

## How to apply

Locate the figure reporting GPU-versus-single-Power8-core speedup (Figure 5 in the paper) and its caption, which covers runs on Summitdev (Tesla P100). For each of the three networks — 13-isotope alpha chain, 9-isotope A=23 Urca, 3-isotope carbon — read the speedup at each grid size (16^3, 32^3, 64^3, 128^3). Alongside each network, record the reported scaling limitations: per-thread local stack memory exhaustion for larger networks and thread divergence arising from differing zone conditions. Serialize one row per network to figure5_gpu_speedup.csv, with a 'grid_sizes' column holding the speedups per grid size and a 'limitations' column holding the caveats, so downstream agents can compare speedup scaling against the documented failure modes.

## Related tools

- **Maestro** (Simulation code whose subsonic convective-flow reaction networks are among the benchmarks whose speedups Figure 5 reports)
- **Castro** (Simulation code for the highly compressible explosion phase whose CUDA-ported reaction networks were benchmarked for GPU speedup)
- **AMReX** (Block-structured AMR library providing the CUDA/GPU acceleration backend on which the reaction-network ports and their Summitdev benchmarks run) — github.com/AMReX-Codes/amrex

## Examples

```
python -c "import pandas as pd; pd.DataFrame(rows, columns=['network','grid_sizes','limitations']).to_csv('figure5_gpu_speedup.csv', index=False)"
```

## Evaluation signals

- figure5_gpu_speedup.csv exists and contains exactly 3 data rows, one per network (13-isotope alpha chain, 9-isotope A=23 Urca, 3-isotope carbon).
- The CSV includes a 'grid_sizes' field whose entries cover all four tested grid sizes: 16^3, 32^3, 64^3, and 128^3.
- The limitations column includes the substring 'stack memory' (per-thread local stack memory exhaustion) and the substring 'thread divergence' (differing zone conditions).
- Speedup values in grid_sizes are plausible GPU-vs-single-CPU-core ratios for a Tesla P100 vs. one Power8 core (i.e., > 1 and monotonically improving or stable with grid size), not absolute times.

## Limitations

- Per-thread local stack memory exhaustion limits how large a reaction network can be ported per-thread before GPU kernels fail — larger networks (e.g., the 13-isotope alpha chain) are the most affected.
- Thread divergence from differing zone conditions reduces effective speedup, since CUDA warps must serialize through divergent reaction-integration branches.
- The reported speedups are for Summitdev's Tesla P100 vs. a single Power8 CPU core only; they do not generalize to other architectures or to full multi-core/GPU-node configurations.

## Evidence

- [abstract] Approach to GPU performance of the codes: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
- [intro] Coupled physics motivating reaction networks: "Accurate modeling of these events requires the coupling of hydrodynamics, gravity, thermonuclear reactions, and in some cases, radiation and magnetic fields."
- [readme] AMReX GPU backends used for the CUDA port: [evidence span withheld — anchor status: external_doc]
- [readme] AMReX scope on block-structured meshes: [evidence span withheld — anchor status: external_doc]
