---
name: hpc-performance-benchmarking
description: Decision support for quantifying HPC solver performance of a parallel
  destriping pipeline — e.g. when asking whether the destriping matrix is too heavy
  to invert directly and how baselines can instead be estimated iteratively for large
  parallel runs.
when_to_use_negative:
- The dataset is small enough that direct matrix inversion of the destriping operator
  is feasible and an explicit solve is preferred
- Systematic effects such as gain variations or spin-synchronous effects dominate
  — the paper warns these must be dealt with before map-making and destriping alone
  is ineffective or even harmful
- The goal is destriping results validation (map/power-spectrum quality) rather than
  computational performance measurement
tools:
- name: dst
  role: Parallel destriper being benchmarked; solves the destriping equation for baselines
    via iterative solvers
  repo: http://github.com/zonca/dst
- name: MPI
  role: Parallel execution of the destriping run via mpirun, with data, baselines,
    and maps distributed uniformly among processors
- name: HDF5
  role: Input data storage format for the simulated timelines, read in parallel
- name: Python
  role: Implementation language of dst and its Cython extensions built with 'python
    setup.py build_ext -i'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:e56b090e9041dc490c2d969296be162d07f9a988e2d168ba9936b98ca18948f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - MPI
  - Python
  - dst
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# hpc-performance-benchmarking

## Summary

Benchmarks the dst Python parallel destriper's iterative-solver performance (GMRES iteration count, residual reduction factor, wall-clock runtime) on a 12-month simulated polarimeter dataset, demonstrating that the left-hand destriping operator never needs to be explicitly inverted.

## When to use

Decision support for quantifying HPC solver performance of a parallel destriping pipeline — e.g. when asking whether the destriping matrix is too heavy to invert directly and how baselines can instead be estimated iteratively for large parallel runs.

## When NOT to use

- The dataset is small enough that direct matrix inversion of the destriping operator is feasible and an explicit solve is preferred
- Systematic effects such as gain variations or spin-synchronous effects dominate — the paper warns these must be dealt with before map-making and destriping alone is ineffective or even harmful
- The goal is destriping results validation (map/power-spectrum quality) rather than computational performance measurement

## Inputs

- 12-month simulated polarimeter dataset in a single HDF5 file with columns TIME (>f8), PHI (>f8), THETA (>f8), PSI (>f8), FLAG (>i2), TEMP (>f4), Q (>f4), U (>f4)
- dst configuration file (e.g. ch6_256.cfg) specifying 1-minute baselines (2000 samples at 33.4 Hz)

## Outputs

- JSON record with three performance metrics: GMRES iteration count, residual reduction factor, and wall-clock runtime in seconds/minutes
- Destriping log reporting the number of Belos GMRES iterations to convergence

## How to apply

Load the 12-month simulated dataset as a single HDF5 file with TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns and configure dst with 1-minute baselines (2000 samples at 33.4 Hz). Launch the run in parallel on an HPC-class node with 'mpirun -np 16 python dst.py ch6_256.cfg', the same pattern the paper demonstrates serially ('python dst.py ch6_256.cfg') and in parallel via MPI. Log the number of GMRES iterations executed by the Belos solver until convergence, the final residual reduction factor, and the wall-clock time of the destriping phase. The rationale is that iterative solvers (Conjugate Gradient or GMRES) estimate baselines by only applying the left-hand operator algorithmically — binning local maps and reducing via Trilinos — never forming or inverting a large matrix. Record the three metrics (iterations, residual reduction, runtime) as a JSON record so future runs can be compared against the reported targets.

## Related tools

- **dst** (Parallel destriper being benchmarked; solves the destriping equation for baselines via iterative solvers) — http://github.com/zonca/dst
- **MPI** (Parallel execution of the destriping run via mpirun, with data, baselines, and maps distributed uniformly among processors)
- **HDF5** (Input data storage format for the simulated timelines, read in parallel)
- **Python** (Implementation language of dst and its Cython extensions built with 'python setup.py build_ext -i')

## Examples

```
mpirun -np 16 python dst.py ch6_256.cfg
```

## Evaluation signals

- The GMRES iteration count recorded for the 12-month run equals 80
- The final residual reduction factor falls between 1e11 and 1e13 (reported target ~1e12)
- Wall-clock destriping runtime is within a broad bound of the reported ~6 minutes on an HPC-class node (equivalent to the Gordon supercomputer)
- The JSON record containing iteration count, residual reduction factor, and runtime exists and parses correctly
- Run completes without matrix inversion — baselines are estimated by iteratively applying the left-hand operator only

## Limitations

- The reported runtime is tied to a specific HPC-class node (Gordon supercomputer equivalent); other hardware will differ
- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility
- Scalability to larger datasets is claimed but not demonstrated, since the LATTE dataset (~half billion samples) was only upcoming/expected

## Evidence

- [full_text] Iterative solvers allow to estimate the baselines with the required accuracy by just applying algorithmically the steps needed to apply the left-hand operator: [evidence span withheld — anchor status: not_found]
- [intro] Solve destriping equation with iterative solvers: [evidence span withheld — anchor status: not_found]
- [methods] Local binning and global map reduction via Trilinos: [evidence span withheld — anchor status: not_found]
