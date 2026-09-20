---
name: mpi-parallel-destriping-execution
description: Use when CMB polarimeter timestreams (e.g. HDF5 files with TIME, PHI,
  THETA, PSI, FLAG, TEMP, Q, U columns) need correlated-noise removal and the destriping
  equation's left-side matrix is too large to invert directly for modest-to-large
  datasets.
when_to_use_negative:
- Data contain systematic effects such as thermally induced gain variations (e.g.
  near sunrise/sunset) or spin-synchronous effects — destriping can be ineffective
  or even harmful and these must be masked or handled separately before map-making
- The scanning strategy lacks frequent crossing points, so the destriping equation
  is poorly constrained
- The noise is white or the 1/f knee frequency is above the scan rate with no removable
  low-frequency correlated component
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: dst
  role: Python parallel destriper that solves the destriping equation for baselines;
    executed under MPI with a config file
  repo: http://github.com/zonca/dst
- name: MPI
  role: Launches the destriper in parallel (mpirun -np N) and supports parallel reading/writing
    of data and maps
- name: HDF5
  role: Input data storage format for the polarimeter timeline columns consumed by
    dst_io.read_data
- name: Python
  role: Implementation language of dst, with cython extensions built via setup.py
    build_ext -i
  repo: http://github.com/zonca/dst
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:45cbf332d551487f33802974b09c8ece9394c451e9c44406d9f4321e31d68648
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - MPI
  - Python
  - dst
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# mpi-parallel-destriping-execution

## Summary

Runs the dst Python parallel destriper under MPI (mpirun) on HDF5 polarimeter timelines to remove low-frequency correlated noise via iterative GMRES/CG baseline estimation, avoiding explicit inversion of the heavy left-side destriping matrix.

## When to use

Use when CMB polarimeter timestreams (e.g. HDF5 files with TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns) need correlated-noise removal and the destriping equation's left-side matrix is too large to invert directly for modest-to-large datasets.

## When NOT to use

- Data contain systematic effects such as thermally induced gain variations (e.g. near sunrise/sunset) or spin-synchronous effects — destriping can be ineffective or even harmful and these must be masked or handled separately before map-making
- The scanning strategy lacks frequent crossing points, so the destriping equation is poorly constrained
- The noise is white or the 1/f knee frequency is above the scan rate with no removable low-frequency correlated component

## Inputs

- Single HDF5 file with columns dtype=[('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2'), ('TEMP', '>f4'), ('Q', '>f4'), ('U', '>f4')])
- dst configuration file (e.g. ch6_256.cfg) specifying baseline length and output settings
- Built dst source tree with cython extensions compiled in place

## Outputs

- Estimated 1-minute baselines (offsets) for the timestream, computed via iterative GMRES/CG solution of the destriping equation
- Destriped timelines/map-making-ready data with low-frequency correlated noise removed
- Performance record: GMRES iteration count, residual reduction factor (~1e12), and wall-clock runtime

## How to apply

Build the cython extensions in place with 'python setup.py build_ext -i', then place the HDF5 dataset (e.g. ch6_fill_3d.h5) inside the h5/ folder and configure baselines in the .cfg file (e.g. 1-minute baselines, 2000 samples at 33.4 Hz). Launch the destriper in parallel with 'mpirun -np <N> python dst.py <config>.cfg' on MPI-enabled nodes; data, baselines and maps are distributed uniformly among processors, binned locally, then reduced to a global HEALPix map via Trilinos. The solver (GMRES via Belos, or Conjugate Gradient) applies the left-hand operator algorithmically — never forming or inverting the large matrix — to estimate the baselines. Record the GMRES iteration count to convergence (reported ~80 iterations for a 12-month simulated dataset), the final residual reduction factor (~1e12), and wall-clock runtime (~6 minutes with 16 processes) as performance metrics.

## Related tools

- **dst** (Python parallel destriper that solves the destriping equation for baselines; executed under MPI with a config file) — http://github.com/zonca/dst
- **MPI** (Launches the destriper in parallel (mpirun -np N) and supports parallel reading/writing of data and maps)
- **HDF5** (Input data storage format for the polarimeter timeline columns consumed by dst_io.read_data)
- **Python** (Implementation language of dst, with cython extensions built via setup.py build_ext -i) — http://github.com/zonca/dst

## Examples

```
mpirun -np 16 python dst.py ch6_256.cfg
```

## Evaluation signals

- GMRES solver converges in about 80 iterations on the 12-month simulated dataset (exact-match check against the reported target)
- Final residual reduction factor falls between 1e11 and 1e13 (reported target ~1e12)
- Wall-clock runtime of the destriping phase is within a broad bound of the reported ~6 minutes
- Data files load successfully from a single HDF5 file with the exact TIME/PHI/THETA/PSI/FLAG/TEMP/Q/U dtype
- Output maps bin to unique global HEALPix pixels after local binning and Trilinos reduction, with no duplicated pixels across ranks

## Limitations

- Input currently must be a single HDF5 file in the fixed dtype; other formats require modifying dst_io.read_data
- Destriping requires a scanning strategy assuring frequent crossing points to be effective
- Systematic effects like gain variations or spin-synchronous effects must be dealt with before the map-making stage; destriping alone cannot handle them
- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility

## Evidence

- [intro] iterative solvers like Conjugate Gradient (CG) or GMRES for the destriping equation: [evidence span withheld — anchor status: not_found]
- [methods] data binned locally on each process, then summed to the global map via Trilinos: [evidence span withheld — anchor status: not_found]
