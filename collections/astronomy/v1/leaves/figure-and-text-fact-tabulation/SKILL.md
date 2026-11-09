---
name: figure-and-text-fact-tabulation
description: Use when an article reports quantitative performance or benchmark results
  in a figure plus supporting section text (e.g. strong-scaling curves), and the goal
  is to turn those stated facts into a structured, per-condition table.
when_to_use_negative:
- The input is raw simulation output or AMReX plotfiles that must be re-analyzed numerically
  rather than reported facts already stated in a figure and section text
- The article contains no labeled figures or sections with quantitative machine/condition
  claims to extract — e.g. a repository README such as the AMReX README, which has
  no scaling data
- The task requires generating new benchmark measurements rather than tabulating facts
  the article already reports
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Maestro
  role: The code whose reported strong-scaling results for the 3-d XRB problem on
    Edison and Titan are extracted and tabulated
- name: AMReX
  role: Block-structured AMR framework both Maestro and Castro are built on; context
    for interpreting the nodal multigrid solver as the scaling limitation
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:c7122f88ee9a72ff624c112f12b625c13778489d635699ce197058171bc48643
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Maestro
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# figure-and-text-fact-tabulation

## Summary

Extracts discrete quantitative facts (scaling extents, machine configurations, limiting factors) from named figures and sections of an article — e.g. Figure 2 and Section 4 of the Castro/Maestro AMReX paper — and tabulates them into a validated CSV schema with one row per measured condition.

## When to use

Use when an article reports quantitative performance or benchmark results in a figure plus supporting section text (e.g. strong-scaling curves), and the goal is to turn those stated facts into a structured, per-condition table.

## When NOT to use

- The input is raw simulation output or AMReX plotfiles that must be re-analyzed numerically rather than reported facts already stated in a figure and section text
- The article contains no labeled figures or sections with quantitative machine/condition claims to extract — e.g. a repository README such as the AMReX README, which has no scaling data
- The task requires generating new benchmark measurements rather than tabulating facts the article already reports

## Inputs

- Article document (Journal of Physics: Conference Series, DOI 10.1088/1742-6596/1031/1/012024) containing Figure 2 and Section 4
- Source card specifying the research question, finding, and target CSV schema (code, problem, machine, scaling_extent, limiting_factor, thread_configuration)

## Outputs

- maestro_strong_scaling.csv with exactly 2 rows (one per machine: Edison, Titan) and the 6 specified columns
- Extracted fact list mapping each machine to its scaling extent (O(10^4) processors), thread configuration (1 MPI/8 threads vs 1 MPI/16 threads per NUMA node), and limiting factor (multigrid nodal solver)

## How to apply

Load the article and first locate the specific figure and section named in the task (here, Figure 2 and Section 4, which report the Maestro strong-scaling study of the 3-d XRB problem). For each machine or condition named in the figure (Edison, Titan), extract the stated scaling extent — excellent scaling demonstrated up to O(10^4) processors on both machines — plus any special annotation such as the performance upturn on Titan caused by switching from 1 MPI task / 8 OpenMP threads per NUMA node to 1 MPI task / 16 threads per node. Separately record the stated limiting factor for further scaling, here the multigrid nodal-projection solver's scaling behavior. Tabulate the extracted facts into a CSV with columns code, problem, machine, scaling_extent, limiting_factor, thread_configuration, writing one row per machine rather than merging conditions. The rationale for one-row-per-condition is that thread configuration and machine are independent variables whose combination explains the observed upturn, so collapsing them would lose the explanatory structure the task asks you to preserve.

## Related tools

- **Maestro** (The code whose reported strong-scaling results for the 3-d XRB problem on Edison and Titan are extracted and tabulated)
- **AMReX** (Block-structured AMR framework both Maestro and Castro are built on; context for interpreting the nodal multigrid solver as the scaling limitation) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import pandas as pd; rows=[{'code':'Maestro','problem':'3-d XRB','machine':m,'scaling_extent':'O(10^4) processors','limiting_factor':'multigrid nodal-projection solver','thread_configuration':t} for m,t in [('Edison','1 MPI/8 threads per NUMA node'),('Titan','1 MPI/16 threads per NUMA node')]]; pd.DataFrame(rows).to_csv('maestro_strong_scaling.csv',index=False)"
```

## Evaluation signals

- maestro_strong_scaling.csv exists at the specified path
- The CSV contains exactly 2 rows, one per machine (Edison, Titan)
- Both rows have non-empty scaling_extent and limiting_factor fields
- The reported processor count in scaling_extent is of order 10^4
- The thread_configuration values distinguish the 1 MPI/8-thread and 1 MPI/16-thread NUMA setups that explain the Titan upturn

## Limitations

- The GitHub-sourced document (AMReX README) lacks DOI, title, authors, year, or journal metadata, so bibliographic facts must come from the article itself, not the repo
- Repository README content is truncated and untrusted; it is context for tooling, not a source of benchmark numbers
- The upturn in the Titan curves is an interpretation tied to the specific 1 MPI/8-thread → 1 MPI/16-thread NUMA switch; tabulating it without the thread_configuration column loses the causal explanation

## Evidence

- [abstract] Maestro scaling on 3-d XRB: "Maestro is tuned to efficiently model subsonic convective flows while Castro models the highly compressible flows associated with stellar explosions."
- [abstract] AMReX foundation: "Both are built on the block-structured adaptive mesh refinement library AMReX."
- [readme] Parallelization context from README: [evidence span withheld — anchor status: external_doc]
