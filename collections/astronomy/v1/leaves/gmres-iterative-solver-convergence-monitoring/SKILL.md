---
name: gmres-iterative-solver-convergence-monitoring
description: Use when the left-side matrix of the destriping equation is too large
  to invert directly (e.g. 12-month datasets of ~half a billion samples) and baselines
  must be solved iteratively with GMRES while monitoring convergence.
when_to_use_negative:
- The dataset is small enough that the left-side matrix of the destriping equation
  can be inverted directly with standard linear algebra
- Systematic effects such as thermally induced gain variations near sunrise/sunset
  are present — destriping can be ineffective or even harmful, and these must be masked
  or handled before map-making
- The scanning strategy lacks frequent crossing points, which destriping requires
  to be effective
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: dst
  role: Python parallel destriper whose destriping equation is solved by GMRES; baselines
    are estimated without forming any large matrix
  repo: http://github.com/zonca/dst
- name: PyTrilinos
  role: Provides the Belos GMRES solver and MPI communication for the parallel iterative
    solution
- name: MPI
  role: Launches the destriping run in parallel (mpirun) so data, baselines and maps
    are distributed uniformly among processors
- name: HDF5
  role: Stores the input timestream read by dst_io.read_data before iterative solving
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:fc926b96692e1330edbd0b1cada1ef0a921ab497b302199c0ed952c40d23d652
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - MPI
  - PyTrilinos
  - Python
  - dst
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# GMRES iterative solver convergence monitoring

## Summary

Estimate destriping baselines for large parallel CMB polarimeter datasets by applying GMRES iteratively to the destriping equation via the dst destriper, never forming the left-side matrix explicitly, and track iteration count, residual reduction, and runtime as convergence metrics.

## When to use

Use when the left-side matrix of the destriping equation is too large to invert directly (e.g. 12-month datasets of ~half a billion samples) and baselines must be solved iteratively with GMRES while monitoring convergence.

## When NOT to use

- The dataset is small enough that the left-side matrix of the destriping equation can be inverted directly with standard linear algebra
- Systematic effects such as thermally induced gain variations near sunrise/sunset are present — destriping can be ineffective or even harmful, and these must be masked or handled before map-making
- The scanning strategy lacks frequent crossing points, which destriping requires to be effective

## Inputs

- Simulated 12-month polarimeter dataset in a single HDF5 file with columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U
- dst configuration file (e.g. ch6_256.cfg) specifying baseline length of 1 minute (2000 samples at 33.4 Hz)

## Outputs

- Estimated 1-minute baselines for the timestream (equivalent to inverting the destriping equation)
- JSON performance record with GMRES iteration count (target 80), residual reduction factor (target ~1e12), and runtime in seconds/minutes

## How to apply

Configure dst with 1-minute baselines (2000 samples at 33.4 Hz) and launch the destriper in parallel (e.g. 'mpirun -np 16 python dst.py ch6_256.cfg') on the simulated 12-month HDF5 dataset. Do not build the left-hand operator as a dense or sparse matrix; instead apply its algorithmic steps (local binning of data on the sky map, then reduction of local maps to the global map via Trilinos) inside each GMRES iteration, using the Belos solver from PyTrilinos. Log the number of GMRES iterations until convergence (reference target: 80) and compute the final residual reduction factor (reference target: ~1e12). Record the wall-clock time of the destriping phase (reference target: about 6 minutes for the 12-month case) so solver performance can be compared across dataset sizes and processor counts.

## Related tools

- **dst** (Python parallel destriper whose destriping equation is solved by GMRES; baselines are estimated without forming any large matrix) — http://github.com/zonca/dst
- **PyTrilinos** (Provides the Belos GMRES solver and MPI communication for the parallel iterative solution)
- **MPI** (Launches the destriping run in parallel (mpirun) so data, baselines and maps are distributed uniformly among processors)
- **HDF5** (Stores the input timestream read by dst_io.read_data before iterative solving)

## Examples

```
mpirun -np 16 python dst.py ch6_256.cfg
```

## Evaluation signals

- GMRES iteration count logged by the Belos solver equals 80 for the 12-month reference case
- Final residual reduction factor falls between 1e11 and 1e13 (reported target ~1e12)
- Wall-clock runtime of the destriping phase is within a broad bound of the reported ~6 minutes on an HPC-class node
- Destriping was run without ever constructing the left-side matrix, i.e. only operator-application steps were executed per iteration
- Output baselines feed a map-maker whose pixel scatter decreases relative to the pre-destriping map

## Limitations

- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility
- Scalability to larger datasets is claimed but not demonstrated (the LATTE dataset of ~half billion samples is only upcoming/expected)
- Runtime targets are tied to an HPC-class node equivalent to the Gordon supercomputer; smaller machines will differ
- Systematic effects such as gain variations or spin-synchronous effects must be handled separately before map-making; destriping alone does not address them

## Evidence

- [full_text] Iterative solvers allow to estimate the baselines with the required accuracy by just applying algorithmically the steps needed to apply the left-hand operator, without involving any large matrix operation.: [evidence span withheld — anchor status: not_found]
- [intro] Solve the destriping equation for baselines using iterative solvers such as Conjugate Gradient or GMRES: [evidence span withheld — anchor status: not_found]
- [methods] Distribute data, baselines and maps uniformly among processors; bin locally then reduce to global map via Trilinos: [evidence span withheld — anchor status: not_found]
