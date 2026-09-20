---
name: sbdf2-timestepper-execution
description: Use when a 1D initial value problem with nonlinear terms must be time-integrated
  with a second-order semi-implicit scheme, e.g. simulating soliton propagation/scattering
  on a multi-segment graph with coupled Chebyshev segments.
when_to_use_negative:
- The problem is a linear eigenvalue problem or steady-state solve rather than time
  evolution (e.g. the Mathieu or Rayleigh-Benard growth-rate problems, which use solve_dense/solve_sparse
  instead of a stepper)
- The domain is not a coupled network of segments; the vertex continuity and Kirchhoff-flux
  coupling machinery would be unnecessary
- A parallel MPI-enabled HDF5 stack is unavailable and the script cannot write its
  output
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Dedalus
  role: Constructs the coupled multi-segment Chebyshev problem, imposes vertex boundary
    conditions, and drives the SBDF2 semi-implicit time integration
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Defines arrays, initial conditions, and imported alongside dedalus.public
    (d3) in every example script
- name: h5py
  role: Loads the HDF5 analysis files written by the simulation output handler to
    access time-evolving snapshots
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Plots the network field at several times to verify soliton scattering behavior
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:5f8c5ef1fc7a2797e4050b7b5d827a2827a291d29dd44904ac2679aca0fb8901
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# sbdf2-timestepper-execution

## Summary

Execute Dedalus's SBDF2 semi-implicit time stepper to evolve a PDE initial value problem forward in time, using the paper's nonlinear Schrodinger network example (Fig. 10) as the method template: a bright soliton on one graph edge scatters at vertices via continuity and Kirchhoff-flux boundary conditions until the excitation fills the entire network.

## When to use

Use when a 1D initial value problem with nonlinear terms must be time-integrated with a second-order semi-implicit scheme, e.g. simulating soliton propagation/scattering on a multi-segment graph with coupled Chebyshev segments.

## When NOT to use

- The problem is a linear eigenvalue problem or steady-state solve rather than time evolution (e.g. the Mathieu or Rayleigh-Benard growth-rate problems, which use solve_dense/solve_sparse instead of a stepper)
- The domain is not a coupled network of segments; the vertex continuity and Kirchhoff-flux coupling machinery would be unnecessary
- A parallel MPI-enabled HDF5 stack is unavailable and the script cannot write its output

## Inputs

- Dedalus example script defining the NLS network problem (multi-segment 1D Chebyshev domain with vertex boundary conditions)
- Bright soliton initial condition placed on a single graph edge
- Simulated time interval and SBDF2 timestepper configuration

## Outputs

- HDF5 analysis files containing evolving field snapshots over time
- Matplotlib plots of the network field at several times showing soliton scattering and network-wide dispersion

## How to apply

Locate the NLS network example script shipped under dedalus/src/dedalus/examples, which builds the coupled multi-segment 1D Chebyshev problem with continuity and Kirchhoff-flux boundary conditions imposed at the network vertices. Run it with python3 (optionally under mpirun for MPI parallelization), specifying the bright soliton as the initial condition on a single edge and letting Dedalus translate the plain-text NLS equations into an efficient, parallelized spectral solver. Time-integrate with the SBDF2 semi-implicit stepper, which treats stiff linear terms implicitly and nonlinear terms explicitly at second order, and let the simulation output handler write field snapshots to HDF5 analysis files. Load the .h5 output with h5py and plot the network field at several times with matplotlib to confirm the soliton translates to the end of its segment, scatters at the vertices, and disperses into the other segments until the excitation fills the entire graph.

## Related tools

- **Dedalus** (Constructs the coupled multi-segment Chebyshev problem, imposes vertex boundary conditions, and drives the SBDF2 semi-implicit time integration) — https://github.com/DedalusProject/dedalus
- **numpy** (Defines arrays, initial conditions, and imported alongside dedalus.public (d3) in every example script)
- **h5py** (Loads the HDF5 analysis files written by the simulation output handler to access time-evolving snapshots) — https://github.com/h5py/h5py
- **matplotlib** (Plots the network field at several times to verify soliton scattering behavior)

## Examples

```
python3 dedalus/src/dedalus/examples/nls.py  # or: mpirun -n 4 python3 dedalus/src/dedalus/examples/nls.py
```

## Evaluation signals

- The soliton translates to the end of its segment before any scattering occurs
- After scattering, the excitation disperses into the other segments and eventually fills the entire graph network over the simulated interval
- HDF5 analysis files are written by the simulation output handler and readable with h5py
- The run enforces continuity and Kirchhoff-flux boundary conditions at the network vertices (no unphysical flux mismatch across junctions)
- Optionally, the script runs under mpirun without changing the physics results

## Limitations

- SBDF2 is second order; accuracy is limited by the time step, and nonlinear terms are treated explicitly
- The NLS network setup depends on coupled multi-segment 1D Chebyshev discretization, so it does not generalize directly to unstructured graph geometries
- Running under MPI requires the parallel HDF5 build described in the paper's methods
- No changelog is available for the dedalus package per the article's discussion

## Evidence

- [abstract] Dedalus translates plain-text equations into parallelized spectral solvers: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [abstract] Diverse example problems including optical network dynamics: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large-scale oceanic flows, low Reynolds number flows, stellar"
- [intro] Symbolic vectorial equation specification in Python: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [readme] h5py is the HDF5 wrapper used to read analysis output: [evidence span withheld — anchor status: external_doc]
