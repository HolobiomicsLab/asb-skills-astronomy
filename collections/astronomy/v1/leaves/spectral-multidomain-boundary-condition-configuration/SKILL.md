---
name: spectral-multidomain-boundary-condition-configuration
description: Use when a PDE must be solved on a graph/network of coupled 1D segments
  (e.g., optical network dynamics), requiring vertex-matching boundary conditions
  and spectral time stepping in Dedalus.
when_to_use_negative:
- Problem is a single 1D/2D/3D domain with no internal boundaries — a plain Dedalus
  problem without multi-domain coupling is simpler
- You need eigenvalue/sparse solves (e.g., Rayleigh-Benard growth rates) rather than
  initial-value time integration
- The equation is linear and analytic solutions or spectral eigenvalue benchmarks
  (e.g., waves-on-a-string evals (n*pi/Lx)**2) suffice
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: Dedalus
  role: Constructs the coupled multi-segment 1D Chebyshev problem, applies continuity
    and Kirchhoff-flux boundary conditions at network vertices, and runs SBDF2 time
    integration
  repo: https://github.com/DedalusProject/dedalus
- name: Python
  role: Execution language for the example scripts; run under mpirun for MPI parallelization
- name: numpy
  role: Array handling for initial conditions and field data
- name: h5py
  role: Loads the HDF5 output written by Dedalus's simulation output handler
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Plots the network field at several times to confirm scattering behavior
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:417c80e9ff7585b970002bb94877d26fa626447e7d832be986e8a97bfad7196b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Dedalus
  - Python
  - h5py
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spectral-multidomain-boundary-condition-configuration

## Summary

Configure a coupled multi-segment 1D Chebyshev spectral problem in Dedalus (e.g., a nonlinear Schrodinger network) by linking separate 1D domains with continuity and Kirchhoff-flux boundary conditions at shared vertices, then time-integrate with the SBDF2 semi-implicit stepper.

## When to use

Use when a PDE must be solved on a graph/network of coupled 1D segments (e.g., optical network dynamics), requiring vertex-matching boundary conditions and spectral time stepping in Dedalus.

## When NOT to use

- Problem is a single 1D/2D/3D domain with no internal boundaries — a plain Dedalus problem without multi-domain coupling is simpler
- You need eigenvalue/sparse solves (e.g., Rayleigh-Benard growth rates) rather than initial-value time integration
- The equation is linear and analytic solutions or spectral eigenvalue benchmarks (e.g., waves-on-a-string evals (n*pi/Lx)**2) suffice

## Inputs

- Graph/network definition (edges, vertex connections) for a coupled multi-segment 1D problem
- PDE specification in plain-text/symbolic form (e.g., nonlinear Schrodinger equation)
- Initial condition (e.g., a bright soliton placed on a single edge)
- Dedalus example scripts bundled under dedalus/src/dedalus/examples

## Outputs

- HDF5 analysis files containing evolving field snapshots from the simulation output handler
- Matplotlib plots of the network field at several times showing soliton scattering at vertices
- Confirmation that the excitation spreads to fill the whole network over the simulated interval

## How to apply

Build each graph edge as its own 1D Chebyshev basis/problem in Dedalus, then couple segments by adding boundary conditions at the shared vertices: field continuity across adjacent edges and Kirchhoff-flux conservation at each junction. Specify the governing equation (e.g., the nonlinear Schrodinger equation) using Dedalus's symbolic vectorial equation interface, with the nonlinear term handled by the SBDF2 semi-implicit time stepper. Launch via python3 (optionally under mpirun for MPI parallelization) and let the simulation output handler write snapshots to HDF5 analysis files. Load the HDF5 output with h5py and plot with matplotlib at several times to verify the dynamics — for the paper's Fig. 10, a bright soliton placed on one edge should translate to the segment end, scatter at the vertices, and disperse to fill the entire network. Correctness is judged by the solver converging with continuity/Kirchhoff conditions enforced and by the expected scattering-and-filling behavior in the plotted fields.

## Related tools

- **Dedalus** (Constructs the coupled multi-segment 1D Chebyshev problem, applies continuity and Kirchhoff-flux boundary conditions at network vertices, and runs SBDF2 time integration) — https://github.com/DedalusProject/dedalus
- **Python** (Execution language for the example scripts; run under mpirun for MPI parallelization)
- **numpy** (Array handling for initial conditions and field data)
- **h5py** (Loads the HDF5 output written by Dedalus's simulation output handler) — https://github.com/h5py/h5py
- **matplotlib** (Plots the network field at several times to confirm scattering behavior)

## Examples

```
python3 dedalus/src/dedalus/examples/nls_network.py  # or: mpirun -n 4 python3 nls_network.py; then load snapshots with h5py
```

## Evaluation signals

- Continuity and Kirchhoff-flux boundary conditions are enforced at every network vertex without solver errors
- The bright soliton translates to the end of its segment before any scattering occurs
- After scattering, the excitation disperses into the other segments rather than reflecting or vanishing
- Late-time snapshots show the field filling the entire graph network, matching Fig. 10 behavior
- Run completes cleanly under both python3 and mpirun, with HDF5 snapshots written at the expected output cadence

## Limitations

- Requires MPI-enabled HDF5/FFTW builds for parallel runs (configure HDF5 with --enable-parallel and FFTW with --enable-mpi)
- Behavior depends on correct vertex boundary-condition specification; misconfigured Kirchhoff-flux conditions will produce spurious scattering
- Demo run requires locating the bundled NLS network example script in dedalus/src/dedalus/examples

## Evidence

- [results] workflow: locate NLS example script: [evidence span withheld — anchor status: not_found]
- [results] vertex BCs: [evidence span withheld — anchor status: not_found]
- [results] SBDF2 time stepping: [evidence span withheld — anchor status: not_found]
- [results] HDF5 output and plotting: [evidence span withheld — anchor status: not_found]
- [results] finding: scattering fills network: [evidence span withheld — anchor status: not_found]
- [intro] symbolic equation interface: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [intro] spectral methods scope: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [methods] MPI HDF5 build: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
