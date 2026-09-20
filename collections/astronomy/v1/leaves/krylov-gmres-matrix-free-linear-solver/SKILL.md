---
name: krylov-gmres-matrix-free-linear-solver
description: Use when the destriping equation has a dense [n_b, n_b] left-side matrix
  too heavy to invert even for modest datasets, and baselines (e.g. 2000-sample/60
  s offsets for B-Machine data) must be recovered iteratively.
when_to_use_negative:
- 'Systematic effects such as gain variations or spin-synchronous effects dominate:
  the paper notes destriping ''can be ineffective or even harmful'' there, and these
  must be handled separately before map-making (e.g. masked sunrise/sunset sections).'
- The scanning strategy does not assure frequent crossing points — destriping requires
  them to be effective.
- Data are not in the expected single-HDF5 format; dst_io.read_data must be modified
  first, so the solver cannot be applied directly.
tools:
- name: dst
  role: python parallel destriper that implements the matrix-free operator application
    and calls the iterative GMRES solve
  repo: http://github.com/zonca/dst
- name: PyTrilinos
  role: provides Belos GMRES and the parallel communication used for distributing
    baselines and reducing local binned maps to the global map
- name: h5py
  role: reads the TIME/PHI/THETA/PSI/FLAG/Q/U columns from ch6_fill_3d.h5 and writes
    the baseline array to HDF5
- name: HEALPix
  role: Nside 256 pixelization on which P and the binning operator B are built
- name: MPI
  role: parallel run of dst.py (e.g. mpirun -np 3) for distributing data, baselines
    and maps among processors
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:fe09a7c37a746827d03c91653fd7e6d48bc4b9ce834b9ffe1a956ffe9a5f59ac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HEALPix
  - MPI
  - PyTrilinos
  - Python
  - dst
  - h5py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# krylov-gmres-matrix-free-linear-solver

## Summary

Solves the destriping equation for baseline amplitudes a by applying the left-hand operator of the form R^T (I - P B^T P^T) R through matrix-vector products on the timeline, rather than forming or inverting the dense [n_b, n_b] left-side matrix, using a Krylov iterative solver (GMRES, or CG) as implemented in the dst parallel destriper.

## When to use

Use when the destriping equation has a dense [n_b, n_b] left-side matrix too heavy to invert even for modest datasets, and baselines (e.g. 2000-sample/60 s offsets for B-Machine data) must be recovered iteratively.

## When NOT to use

- Systematic effects such as gain variations or spin-synchronous effects dominate: the paper notes destriping 'can be ineffective or even harmful' there, and these must be handled separately before map-making (e.g. masked sunrise/sunset sections).
- The scanning strategy does not assure frequent crossing points — destriping requires them to be effective.
- Data are not in the expected single-HDF5 format; dst_io.read_data must be modified first, so the solver cannot be applied directly.

## Inputs

- Single HDF5 timeline file (e.g. ch6_fill_3d.h5) with columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U (dtype [('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')])
- Configuration file (e.g. ch6_256.cfg) specifying HEALPix Nside 256 grid, baseline length and solver settings
- Baseline length parameter: 2000 samples (~60 s at 33.4 Hz sampling)

## Outputs

- Baseline amplitude array a written to HDF5
- Destriped Q and U HEALPix maps (Nside 256) written as HEALPix FITS files
- GMRES residual/convergence information (target residual reduction of order 10^12)

## How to apply

Load the polarimeter timeline (e.g. 3 days from ch6_fill_3d.h5 with TIME, PHI, THETA, PSI, FLAG, Q, U columns, dropping samples flagged in FLAG) and build the pointing matrix P and binning operator B on a HEALPix Nside 256 grid as configured in ch6_256.cfg. Partition the timeline into baselines of 2000 samples (about 60 seconds at 33.4 Hz sampling) and assemble the destriping operator R^T (I - P B^T P^T) R implicitly, applying it only via matrix-vector products on the timestream — never forming a dense [n_b, n_b] matrix. Solve for the baseline amplitudes a with GMRES (PyTrilinos/Belos in the paper's dst code, or SciPy's sparse GMRES), possibly distributed with MPI (e.g. mpirun -np 3), iterating until the residual is reduced by the order of 10^12. The rationale is that the baselines 'are estimated iteratively by binning the data on a sky map and minimizing the scatter among different measurements of each pixel', and Krylov methods such as Conjugate Gradient or GMRES need only operator applications, which keeps memory and compute feasible. Subtract the solved baselines from the timeline and bin the corrected Q and U timestreams into destriped HEALPix maps.

## Related tools

- **dst** (python parallel destriper that implements the matrix-free operator application and calls the iterative GMRES solve) — http://github.com/zonca/dst
- **PyTrilinos** (provides Belos GMRES and the parallel communication used for distributing baselines and reducing local binned maps to the global map)
- **h5py** (reads the TIME/PHI/THETA/PSI/FLAG/Q/U columns from ch6_fill_3d.h5 and writes the baseline array to HDF5)
- **HEALPix** (Nside 256 pixelization on which P and the binning operator B are built)
- **MPI** (parallel run of dst.py (e.g. mpirun -np 3) for distributing data, baselines and maps among processors)

## Examples

```
mpirun -np 3 python dst.py ch6_256.cfg
```

## Evaluation signals

- GMRES converges with the residual reduced by the order of 10^12, as reported for the paper's solver runs.
- No dense [n_b, n_b] matrix is ever formed: the operator is applied only through matrix-vector products on the timeline.
- A baseline array is produced and written to HDF5 for the 2000-sample baselines.
- Destriped Q and U map files exist as HEALPix FITS containing Q and U fields on the Nside 256 grid.
- Destriped residuals show reduced correlated-noise striping relative to the simply binned maps.

## Limitations

- Systematic effects (gain variations, spin-synchronous effects) must be handled separately before map-making; destriping alone can be ineffective or even harmful.
- Scalability to larger datasets (e.g. LATTE's ~half billion samples) is claimed but not demonstrated; LATTE data are only upcoming/expected.
- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility.
- Data must be in a single HDF5 file with the specific dtype listed in the README; other formats require modifying dst_io.read_data.

## Evidence

- [intro] iterative solvers: [evidence span withheld — anchor status: not_found]
- [intro] iterative baseline estimation: [evidence span withheld — anchor status: not_found]
