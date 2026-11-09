---
name: gpu-speedup-data-tabulation
description: Use when a source card asks you to reproduce quantitative GPU speedup
  results from a paper figure (e.g., Figure 5) for multiple reaction networks and
  grid sizes, and the deliverable is a structured table with a limitations column.
when_to_use_negative:
- The input is raw simulation output or timing logs rather than an already-reported
  figure — re-deriving timings requires running the Maestro/Castro codes on Summitdev,
  not tabulating published values
- The target hardware is not the paper's Tesla P100 vs. single Power8 core setup,
  since the speedup values are hardware-specific and not transferable
- A single-network analysis is needed — this skill is structured around the three-network
  (13-, 9-, 3-isotope) comparison
tools:
- name: AMReX
  role: Underlying AMR framework whose GPU porting (CUDA backend) produced the speedup
    results being tabulated
  repo: https://github.com/AMReX-Codes/amrex
- name: Castro
  role: Compressible-flow simulation code in which the reaction networks were run
    for the GPU speedup measurements
- name: Maestro
  role: Companion subsonic-convection simulation code of the same suite whose GPU
    performance approach motivated the tabulation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:c2dc305e99fbcb25b218dd2d5c79c45516ef0da9c532c62676987b37bcc5eebf
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# gpu-speedup-data-tabulation

## Summary

Extracts reported GPU-versus-single-CPU-core speedup values for the 13-, 9-, and 3-isotope astrophysical reaction networks (Tesla P100 vs. Power8 core on Summitdev) from a paper figure and serializes them, with scaling-limitation annotations, into a machine-readable CSV.

## When to use

Use when a source card asks you to reproduce quantitative GPU speedup results from a paper figure (e.g., Figure 5) for multiple reaction networks and grid sizes, and the deliverable is a structured table with a limitations column.

## When NOT to use

- The input is raw simulation output or timing logs rather than an already-reported figure — re-deriving timings requires running the Maestro/Castro codes on Summitdev, not tabulating published values
- The target hardware is not the paper's Tesla P100 vs. single Power8 core setup, since the speedup values are hardware-specific and not transferable
- A single-network analysis is needed — this skill is structured around the three-network (13-, 9-, 3-isotope) comparison

## Inputs

- Published figure and caption reporting GPU speedup (Figure 5 of the Castro/Maestro/AMReX paper)
- Grid-size sweep specification (16^3, 32^3, 64^3, 128^3)
- Reaction-network labels (13-isotope alpha chain, 9-isotope A=23 Urca, 3-isotope carbon)
- Reported limitation descriptions (stack memory exhaustion, thread divergence)

## Outputs

- figure5_gpu_speedup.csv with one row per reaction network, a 'grid_sizes' column of speedup values per grid size, and a 'limitations' column

## How to apply

Locate the speedup figure and its caption (Figure 5 in this paper), then read off the GPU-versus-single-Power8-core speedup values for each of the three CUDA-ported networks — the 13-isotope alpha chain, the 9-isotope A=23 Urca network, and the 3-isotope carbon network — at each tested grid size (16^3, 32^3, 64^3, 128^3) on Summitdev's Tesla P100. Record the reported scaling caveats alongside the numbers: per-thread local stack memory exhaustion for larger networks and thread divergence arising from differing zone conditions. Build one row per network with a 'grid_sizes' column holding the per-grid-size speedups and a 'limitations' column, then serialize to figure5_gpu_speedup.csv. Validate that the file has exactly 3 rows, contains the 'grid_sizes' field, and that the limitations column contains the substrings 'stack memory' and 'thread divergence'.

## Related tools

- **AMReX** (Underlying AMR framework whose GPU porting (CUDA backend) produced the speedup results being tabulated) — https://github.com/AMReX-Codes/amrex
- **Castro** (Compressible-flow simulation code in which the reaction networks were run for the GPU speedup measurements)
- **Maestro** (Companion subsonic-convection simulation code of the same suite whose GPU performance approach motivated the tabulation)

## Examples

```
python -c "import pandas as pd; df = pd.DataFrame(rows); df.to_csv('figure5_gpu_speedup.csv', index=False)"
```

## Evaluation signals

- figure5_gpu_speedup.csv exists and parses successfully
- The CSV has exactly 3 rows, one per reaction network (13-isotope alpha chain, 9-isotope Urca, 3-isotope carbon)
- The schema includes a 'grid_sizes' field holding speedup values for 16^3, 32^3, 64^3, and 128^3
- The 'limitations' column contains the substrings 'stack memory' and 'thread divergence'
- Speedup values are attributed to the correct hardware pairing (Tesla P100 GPU vs. single Power8 CPU core on Summitdev)

## Limitations

- Per-thread local stack memory exhaustion limits reaction-network size on the GPU (larger networks are harder to run per-thread)
- Thread divergence arises because different zones are in different conditions, degrading GPU efficiency
- Reported speedups are specific to Summitdev's Tesla P100 vs. a single Power8 core and do not generalize to other architectures

## Evidence

- [results] Speedups reported on Summitdev: [evidence span withheld — anchor status: not_found]
- [abstract] GPU performance motivation from the abstract: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
- [readme] AMReX GPU backends from README: [evidence span withheld — anchor status: external_doc]
