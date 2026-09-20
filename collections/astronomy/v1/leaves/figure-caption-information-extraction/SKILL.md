---
name: figure-caption-information-extraction
description: Use when a research question asks to reproduce or verify results reported
  in a specific figure (e.g., GPU speedups in Figure 5) and the values must be read
  from the figure or caption and converted to structured rows.
when_to_use_negative:
- The quantity of interest is reported only in the article body text or a table, not
  in a figure/caption — direct text extraction is more appropriate.
- The figure is a schematic or conceptual illustration with no quantitative values
  to transcribe.
- You need to re-run the simulations (Maestro/Castro networks on Summitdev) rather
  than transcribe reported values from the publication.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: AMReX
  role: Underlying block-structured AMR framework whose GPU-accelerated (CUDA-ported)
    reaction networks are the subject of the extracted figure; context for interpreting
    the speedup values.
  repo: https://github.com/AMReX-Codes/amrex
- name: Castro
  role: Compressible-flow stellar explosion code whose reaction-network GPU speedups
    (13-, 9-, and 3-isotope networks) are the data being extracted from Figure 5.
- name: Maestro
  role: Subsonic convection code from the same suite; the extracted speedup findings
    concern the network porting work described alongside Maestro and Castro in the
    paper.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:f805e3f3671c992ea68fe7c86c93c1cc363c409b59592278e1f14db50df9e7dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Castro
  - Maestro
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Figure Caption Information Extraction

## Summary

Locates a specific figure in a paper (e.g., Figure 5 of the Castro/Maestro/AMReX astrophysical explosions paper), reads quantitative values from the figure and its caption, and serializes them into a structured table with explicit limitation annotations.

## When to use

Use when a research question asks to reproduce or verify results reported in a specific figure (e.g., GPU speedups in Figure 5) and the values must be read from the figure or caption and converted to structured rows.

## When NOT to use

- The quantity of interest is reported only in the article body text or a table, not in a figure/caption — direct text extraction is more appropriate.
- The figure is a schematic or conceptual illustration with no quantitative values to transcribe.
- You need to re-run the simulations (Maestro/Castro networks on Summitdev) rather than transcribe reported values from the publication.

## Inputs

- PDF or full text of the target article (e.g., 10.1088/1742-6596/1031/1/012024) containing the figure and caption
- The figure number/title to extract (e.g., Figure 5)
- The list of series/conditions to tabulate (reaction networks and grid sizes 16^3–128^3)

## Outputs

- figure5_gpu_speedup.csv with exactly 3 rows, one per reaction network, with a grid_sizes column of speedups and a limitations column

## How to apply

First locate the named figure (Figure 5) and read its caption to identify what is plotted and the experimental platform — here, GPU-versus-single-Power8-core speedup on Summitdev's Tesla P100. For each series in the figure (the 13-isotope alpha chain, 9-isotope A=23 Urca, and 3-isotope carbon reaction networks), extract the metric at every reported condition (grid sizes 16^3, 32^3, 64^3, and 128^3). Alongside the quantitative values, record the qualitative caveats the paper attributes to each series — per-thread local stack memory exhaustion for larger networks and thread divergence from differing zone conditions — in a dedicated limitations column. Serialize to a flat CSV (figure5_gpu_speedup.csv) with one row per network and a grid_sizes column holding the per-grid-size speedups.

## Related tools

- **AMReX** (Underlying block-structured AMR framework whose GPU-accelerated (CUDA-ported) reaction networks are the subject of the extracted figure; context for interpreting the speedup values.) — https://github.com/AMReX-Codes/amrex
- **Castro** (Compressible-flow stellar explosion code whose reaction-network GPU speedups (13-, 9-, and 3-isotope networks) are the data being extracted from Figure 5.)
- **Maestro** (Subsonic convection code from the same suite; the extracted speedup findings concern the network porting work described alongside Maestro and Castro in the paper.)

## Examples

```
python scripts/extract_figure5.py --article 10.1088/1742-6596/1031/1/012024 --figure 5 --output figure5_gpu_speedup.csv
```

## Evaluation signals

- figure5_gpu_speedup.csv exists and contains exactly 3 rows, one per reaction network (3-isotope carbon, 9-isotope Urca, 13-isotope alpha chain).
- The file contains a 'grid_sizes' field holding speedups for all four grid sizes (16^3, 32^3, 64^3, 128^3).
- The limitations column includes the substrings 'stack memory' and 'thread divergence'.
- Extracted speedup directions match the paper's claim of good speedup for all three networks on Tesla P100 vs. a single Power8 core.
- Each extracted value is traceable to the figure or its caption (e.g., can be checked against the caption text).

## Limitations

- The paper reports per-thread local stack memory exhaustion as a scaling issue for larger reaction networks, which constrains interpretation of the largest-network speedups.
- Thread divergence arises from differing zone conditions across the grid, so speedups may not scale uniformly across grid sizes.
- Values are transcribed from the figure/caption as reported; the workflow validates transcription and serialization, not independent re-execution of the benchmarks.

## Evidence

- [results] All three reaction networks showed good speedup on Tesla P100 vs. single Power8 core: [evidence span withheld — anchor status: not_found]
- [methods] Workflow instructs locating Figure 5 and its caption: [evidence span withheld — anchor status: not_found]
- [methods] Recorded scaling limitations: [evidence span withheld — anchor status: not_found]
- [abstract] Codes built on AMReX targeting GPU architectures: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
