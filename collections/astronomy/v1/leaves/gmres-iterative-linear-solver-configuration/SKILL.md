---
name: gmres-iterative-linear-solver-configuration
description: Use when running the dst destriper on polarimeter timelines (HDF5 with
  TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns) and the baseline-offset linear
  system must be solved iteratively rather than by direct inversion.
when_to_use_negative:
- The timestream still contains uncorrected systematic effects (e.g. thermally induced
  gain variations near sunrise/sunset, spin-synchronous effects) that destriping cannot
  remove and that 'need to be dealt with before the map-making stage'
- The scanning strategy lacks frequent crossing points, so the destriping equation
  is ill-conditioned and iterative baseline solution will be ineffective
- The 1/f knee frequency is higher than the scan rate, outside the regime where removing
  predetermined-length offsets efficiently eliminates correlated noise
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: dst
  role: Parallel Python destriper whose baseline linear system is solved with GMRES
    via Trilinos; invoked as 'python dst.py ch6_256.cfg'
  repo: http://github.com/zonca/dst
- name: PyTrilinos
  role: Python bindings to Trilinos providing the Belos GMRES iterative solver and
    distributed reduction of locally binned maps to the global map
- name: HEALPix
  role: Pixelization scheme used for the binned map distribution against which the
    baseline solution is iterated
- name: MPI
  role: Enables parallel execution of the GMRES solve ('mpirun -np 3 python dst.py
    ch6_256.cfg') with data, baselines and maps distributed uniformly among processors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:9b3f1dae2de6be12caab7c5a0312571d8ff8c24498463935d67e69b3a9bb4038
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HEALPix
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
yes_rate: 0.6
n_claims: 10
---

# GMRES iterative linear solver configuration

## Summary

Configuration and use of the GMRES (Generalized Minimal RESidual) iterative solver, via PyTrilinos/Belos, to solve the dst destriper's linear system for baseline offsets in CMB polarimeter timestreams (B-Machine 37.5 GHz / ch6_256 configuration).

## When to use

Use when running the dst destriper on polarimeter timelines (HDF5 with TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns) and the baseline-offset linear system must be solved iteratively rather than by direct inversion.

## When NOT to use

- The timestream still contains uncorrected systematic effects (e.g. thermally induced gain variations near sunrise/sunset, spin-synchronous effects) that destriping cannot remove and that 'need to be dealt with before the map-making stage'
- The scanning strategy lacks frequent crossing points, so the destriping equation is ill-conditioned and iterative baseline solution will be ineffective
- The 1/f knee frequency is higher than the scan rate, outside the regime where removing predetermined-length offsets efficiently eliminates correlated noise

## Inputs

- Polarimeter timestream in a single HDF5 file with columns [('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')], e.g. the ch6_fill_3d.h5 3-day test dataset from figshare placed in the h5/ folder
- dst configuration file (e.g. ch6_256.cfg) specifying baseline length (60 s / 2000 samples at 33.4 Hz) and solver/mapmaking settings

## Outputs

- Solved baseline-offset array (writable to baselines.h5 or .csv), expected ~0 to 1e-16 K for signal-only input
- Destriped HEALPix map (Q/U) distributed and summed to unique pixels across processors via Trilinos

## How to apply

Build the dst package's cython extensions in place with 'python setup.py build_ext -i' after installing h5py, PyTrilinos, cython and openmpi, since the destriper relies on Trilinos for distributed communication. The destriping equation — which relates the baseline offsets (e.g. 1-minute baselines, 2000 samples at 33.4 Hz, slightly shorter than the 70-second B-Machine spin period) to the map scatter — is too large for direct inversion, so it is solved with an iterative Krylov method; the paper names Conjugate Gradient (CG) and GMRES as the candidate solvers, with GMRES via Trilinos Belos used in the dst workflow. Configure the solver through the ch6_256.cfg file (see its comments for the configuration format) and run serially with 'python dst.py ch6_256.cfg' or in parallel with 'mpirun -np 3 python dst.py ch6_256.cfg', where data, baselines and maps are distributed among processes and local binnings are reduced to a global HEALPix map through Trilinos. Validate convergence by extracting the solved baseline array and checking that, for a signal-only input timestream, the maximum absolute baseline is null to machine precision (~1e-16 K, below 1e-15 K), consistent with the model in which 'n is the white plus correlated noise timeline' and signal-only data contain no correlated-noise component for the baselines to represent.

## Related tools

- **dst** (Parallel Python destriper whose baseline linear system is solved with GMRES via Trilinos; invoked as 'python dst.py ch6_256.cfg') — http://github.com/zonca/dst
- **PyTrilinos** (Python bindings to Trilinos providing the Belos GMRES iterative solver and distributed reduction of locally binned maps to the global map)
- **HEALPix** (Pixelization scheme used for the binned map distribution against which the baseline solution is iterated)
- **MPI** (Enables parallel execution of the GMRES solve ('mpirun -np 3 python dst.py ch6_256.cfg') with data, baselines and maps distributed uniformly among processors)

## Examples

```
python setup.py build_ext -i && mpirun -np 3 python dst.py ch6_256.cfg
```

## Evaluation signals

- For a signal-only input timestream, the maximum absolute solved baseline amplitude is < 1e-15 K (machine-precision null, ~1e-16 K as reported)
- The GMRES solve (Trilinos Belos) converges without direct inversion, and the destriped map's scatter among pixel measurements decreases relative to the pre-destriping binned map
- The output baseline file (baselines.h5) and HEALPix FITS map have the expected structure (baseline array per timeline segment; HEALPix Q/U pixelization)
- Serial ('python dst.py ch6_256.cfg') and parallel ('mpirun -np 3 python dst.py ch6_256.cfg') runs produce consistent baseline solutions

## Limitations

- Version numbers for Python, Trilinos, HDF5, or HEALPix are not given, limiting reproducibility of the solver environment
- Destriping is a high-pass filter that only affects the noise in the timestream; it can be ineffective or even harmful when systematic effects (gain variations, spin-synchronous signals) dominate
- The dst input reader must be modified (dst_io.read_data) to support any HDF5 column format other than the specific dtype listed in the README

## Evidence

- [intro] Solve the destriping equation for baselines using iterative solvers such as Conjugate Gradient or GMRES: [evidence span withheld — anchor status: not_found]
- [methods] The destriper reduces locally binned maps to a global unique-pixel map through Trilinos during the iterative solve: [evidence span withheld — anchor status: not_found]
