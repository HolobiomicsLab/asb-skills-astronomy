---
name: hpc-strong-scaling-interpretation
description: 'Use when a benchmark figure or table reports GPU/node scaling for an
  AMReX astrophysics code (e.g., Figure 4 of the Castro/Maestro paper: Star Lord on
  Summitdev) and the goal is to characterize speedup behavior and GPU-vs-CPU performance
  ratios.'
when_to_use_negative:
- Input reports CPU-only cluster scaling with no GPU runs — there is no P100-vs-Power8
  ratio or gpu_count_max to extract
- The scaling data is for a different code/problem than the Castro/Sedov proxy benchmark
  described in this article — do not transfer Summitdev-specific numbers to other
  workloads
- The figure reports strong-scaling efficiency at fixed problem size with an explicit
  efficiency metric; in that case report the efficiency directly rather than the weak-scaling
  characterization used in this paper
tools:
- name: Castro
  role: Compressible hydrodynamics code whose proxy app (Star Lord, Castro hydrodynamics
    + stellar EOS) is the subject of the Summitdev GPU scaling benchmark
- name: AMReX
  role: Block-structured AMR framework that underlies Castro/Maestro and provides
    the GPU acceleration backends enabling the P100 scaling runs
  repo: github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:d3a85b2f0344454b2b8606784bcbff17a6fd7ffb05c6119370873609d6f74401
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AMReX
  - Castro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# hpc-strong-scaling-interpretation

## Summary

Interprets GPU/node scaling behavior of AMReX-based astrophysics codes (e.g., the Star Lord proxy app of Castro running the Sedov pure-hydro problem on Summitdev) by extracting the maximum GPU count, characterizing speedup behavior, and computing the single-GPU vs. CPU-core performance ratio.

## When to use

Use when a benchmark figure or table reports GPU/node scaling for an AMReX astrophysics code (e.g., Figure 4 of the Castro/Maestro paper: Star Lord on Summitdev) and the goal is to characterize speedup behavior and GPU-vs-CPU performance ratios.

## When NOT to use

- Input reports CPU-only cluster scaling with no GPU runs — there is no P100-vs-Power8 ratio or gpu_count_max to extract
- The scaling data is for a different code/problem than the Castro/Sedov proxy benchmark described in this article — do not transfer Summitdev-specific numbers to other workloads
- The figure reports strong-scaling efficiency at fixed problem size with an explicit efficiency metric; in that case report the efficiency directly rather than the weak-scaling characterization used in this paper

## Inputs

- GPU scaling figure or table from the article (e.g., Figure 4, left: Star Lord on Summitdev)
- Problem description identifying the benchmark workload (Sedov pure-hydro problem) and hardware platform (Summitdev: Tesla P100 GPUs, Power8 CPUs)
- Text passages stating maximum node/GPU counts and speedup characterization

## Outputs

- JSON record (star_lord_gpu_results.json) with fields gpu_count_max (integer), scaling_behavior (string), p100_vs_power8_ratio (float)
- Scaling characterization string, e.g. 'nearly linear speedup / good weak scaling'
- Single-GPU vs CPU performance ratio (float, approximately 2.5)

## How to apply

Locate the GPU performance discussion for the proxy app in the article (for Star Lord: Summitdev runs of the Sedov pure-hydro problem, Castro hydrodynamics + stellar EOS, Figure 4 left). Extract the maximum GPU count reported — 128 GPUs across 32 nodes on Summitdev — and the authors' scaling characterization, 'nearly linear speedup / good weak scaling'. Extract the single-GPU comparison: one Tesla P100 achieving approximately 2.5x the performance of 20 Power8 cores. Assemble these into a record with fields gpu_count_max (integer), scaling_behavior (string), and p100_vs_power8_ratio (float). Preserve the article's own scaling terminology (the article characterizes the result as good weak scaling) even when the analysis was motivated by strong-scaling questions.

## Related tools

- **Castro** (Compressible hydrodynamics code whose proxy app (Star Lord, Castro hydrodynamics + stellar EOS) is the subject of the Summitdev GPU scaling benchmark)
- **AMReX** (Block-structured AMR framework that underlies Castro/Maestro and provides the GPU acceleration backends enabling the P100 scaling runs) — github.com/AMReX-Codes/amrex

## Examples

```
python -c "import json; d=json.load(open('star_lord_gpu_results.json')); assert d['gpu_count_max']==128 and 2.0<=d['p100_vs_power8_ratio']<=3.0"
```

## Evaluation signals

- star_lord_gpu_results.json exists and parses as valid JSON
- Field gpu_count_max equals 128 (32 nodes on Summitdev)
- Field p100_vs_power8_ratio falls in the range 2.0–3.0 (expected value ~2.5)
- scaling_behavior string matches the article's characterization: 'nearly linear speedup / good weak scaling'
- Node-count consistency: gpu_count_max divided by 4 GPUs per node equals 32 nodes

## Limitations

- Scaling results are platform-specific to Summitdev (Tesla P100 GPUs, Power8 CPUs) and the Sedov pure-hydro problem; they should not be extrapolated to other architectures or workloads
- Speedup is reported only up to 128 GPUs (32 nodes); behavior beyond that point is not characterized in the article
- The scaling characterization is qualitative ('nearly linear'); no per-point efficiency or timing values are provided in the extracted entities

## Evidence

- [results] Star Lord scaling finding (Figure 4, left): [evidence span withheld — anchor status: not_found]
- [abstract] Architecture-performance motivation: "the approach we are taking to make these codes performant on current and future many-core and GPU-based architectures"
- [readme] GPU backends in AMReX: [evidence span withheld — anchor status: external_doc]
- [readme] Scalable heterogeneous performance: [evidence span withheld — anchor status: external_doc]
