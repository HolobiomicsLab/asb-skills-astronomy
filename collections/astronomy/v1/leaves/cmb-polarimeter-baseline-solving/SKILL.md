---
name: cmb-polarimeter-baseline-solving
description: Use when polarimeter timestream data (e.g. HDF5 files with TIME, PHI,
  THETA, PSI, FLAG, Q, U columns) contains correlated 1/f noise to be removed via
  Maximum Likelihood destriping and the baseline amplitudes must be solved for.
when_to_use_negative:
- Systematic effects such as gain variations or spin-synchronous effects dominate
  the timestream — destriping can be ineffective or even harmful and these must be
  handled before map-making (e.g. sunrise/sunset sections with thermally induced gain
  variations were masked out)
- The scanning strategy lacks frequent crossing points, since destriping requires
  such a strategy to be effective
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: dst
  role: Python parallel destriper that solves the destriping equation for baselines
    with GMRES via PyTrilinos Belos
  repo: http://github.com/zonca/dst
- name: MPI
  role: Runs dst in parallel (mpirun) with data, baselines and maps distributed uniformly
    among processors
- name: HDF5
  role: Input/output storage format for the timeline and baseline arrays, supporting
    parallel reading and writing through MPI
- name: HEALPix
  role: Pixelization scheme (Nside 256) on which the binning operator B is built and
    destriped Q/U maps are written
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:37aad0d4591637860b87bf8babb2119d51bb495092b1413cc12bd50230f211a6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HEALPix
  - MPI
  - Python
  - dst
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cmb-polarimeter-baseline-solving

## Summary

Estimates the baseline array of the destriping equation for CMB polarimeter timelines by iteratively applying the left-hand operator with GMRES, avoiding inversion of the dense [n_b, n_b] matrix that is computationally infeasible even for modest datasets.

## When to use

Use when polarimeter timestream data (e.g. HDF5 files with TIME, PHI, THETA, PSI, FLAG, Q, U columns) contains correlated 1/f noise to be removed via Maximum Likelihood destriping and the baseline amplitudes must be solved for.

## When NOT to use

- Systematic effects such as gain variations or spin-synchronous effects dominate the timestream — destriping can be ineffective or even harmful and these must be handled before map-making (e.g. sunrise/sunset sections with thermally induced gain variations were masked out)
- The scanning strategy lacks frequent crossing points, since destriping requires such a strategy to be effective

## Inputs

- Single HDF5 file with columns dtype=[('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2'), ('TEMP', '>f4'), ('Q', '>f4'), ('U', '>f4')], e.g. ch6_fill_3d.h5
- Configuration file ch6_256.cfg specifying HEALPix Nside 256 and baseline length
- Baseline partitioning: 2000-sample (~60 s at 33.4 Hz) baselines

## Outputs

- Solved baseline amplitude array a, written to HDF5
- Destriped HEALPix Q and U maps in FITS format
- Solver performance record: GMRES iteration count (~80), residual reduction factor (~1e12), and wall-clock runtime (~6 min for 12-month data)

## How to apply

Load the timeline from a single HDF5 file (e.g. ch6_fill_3d.h5 with TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns) and drop samples flagged per the FLAG column. Build the pointing matrix P and binning operator B on a HEALPix Nside 256 grid (as set in ch6_256.cfg), and partition the timeline into baselines of 2000 samples (~60 seconds at 33.4 Hz). Assemble the destriping operator R^T (I - P B^T P^T) R implicitly, applying it via matrix-vector products on the timeline rather than forming any dense matrix, then solve for baseline amplitudes a with GMRES (Belos from Trilinos via PyTrilinos in the dst code) until the residual is reduced by ~1e12 (reported: 80 iterations, ~6 minutes for a 12-month dataset run in parallel with MPI). Finally subtract the solved baselines and bin corrected Q and U timestreams into destriped HEALPix maps. The rationale is that iterative solvers only require algorithmic application of the left-hand operator, never large matrix operations, enabling scaling to large parallel runs such as the ~half-billion-sample LATTE dataset.

## Related tools

- **dst** (Python parallel destriper that solves the destriping equation for baselines with GMRES via PyTrilinos Belos) — http://github.com/zonca/dst
- **MPI** (Runs dst in parallel (mpirun) with data, baselines and maps distributed uniformly among processors)
- **HDF5** (Input/output storage format for the timeline and baseline arrays, supporting parallel reading and writing through MPI)
- **HEALPix** (Pixelization scheme (Nside 256) on which the binning operator B is built and destriped Q/U maps are written)

## Examples

```
mpirun -np 16 python dst.py ch6_256.cfg
```

## Evaluation signals

- GMRES solver runs to convergence with a large residual reduction factor between 1e11 and 1e13 (reported ~1e12) at ~80 iterations
- Destriped Q and U map files exist as HEALPix FITS with Q and U fields
- Destriped residuals show reduced correlated-noise striping relative to binned maps
- Wall-clock runtime of the destriping phase is within a broad bound of the reported ~6 minutes for the 12-month run
- No dense [n_b, n_b] matrix was ever formed — the left-hand operator is applied only via matrix-vector products

## Limitations

- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility
- Destriping must be preceded by handling of systematic effects (gain variations, spin-synchronous effects); no method for that is specified
- Scalability to larger datasets is claimed but not demonstrated (the LATTE dataset is only upcoming/expected)

## Evidence

- [results] Iterative solvers allow to estimate the baselines with the required accuracy by just applying algorithmically the steps needed to apply the left-hand operator, without involving any large matrix operation.: [evidence span withheld — anchor status: not_found]
- [intro] Solve the destriping equation for baselines using iterative solvers such as Conjugate Gradient or GMRES: [evidence span withheld — anchor status: not_found]
- [methods] The data are binned locally on each process, then the local maps are distributed and summed to the global map which has unique pixels: [evidence span withheld — anchor status: not_found]
