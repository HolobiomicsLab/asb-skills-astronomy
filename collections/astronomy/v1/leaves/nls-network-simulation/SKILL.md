---
name: nls-network-simulation
description: Use when a nonlinear PDE (e.g. nonlinear Schrodinger equation with a
  bright soliton) must be evolved on a graph of coupled 1D segments and you need to
  observe scattering at vertices and spreading across the full network over time.
when_to_use_negative:
- The problem is an eigenvalue calculation (e.g. Mathieu, Rayleigh-Benard growth rates,
  pipe-flow eigenvalues) — use Dedalus's solve_dense/solve_sparse paths rather than
  SBDF2 initial-value integration.
- The PDE is linear and single-domain, so no vertex continuity/Kirchhoff-flux coupling
  or nonlinear term is needed.
- You only need a steady spectral solve with no time evolution of a traveling excitation.
tools:
- name: Dedalus
  role: Constructs the coupled multi-segment Chebyshev NLS problem, applies continuity
    and Kirchhoff-flux boundary conditions at vertices, and time-integrates with SBDF2
    while writing HDF5 analysis snapshots
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Defines the bright soliton initial condition and manipulates field arrays
    when inspecting output
- name: h5py
  role: Loads the HDF5 analysis files written by the simulation output handler for
    post-hoc plotting
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Plots the network field at several simulation times to confirm scattering
    and dispersion
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:b7f301e4667ad45570f45e594b8dc0a582cf27f0ca04efb3a00e20b8163e242e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# nls-network-simulation

## Summary

Simulates nonlinear Schrodinger dynamics on a graph network by time-integrating a bright soliton placed on one edge with Dedalus's SBDF2 semi-implicit stepper, reproducing the paper's Fig. 10 vertex-scattering behavior where the excitation disperses into all network segments.

## When to use

Use when a nonlinear PDE (e.g. nonlinear Schrodinger equation with a bright soliton) must be evolved on a graph of coupled 1D segments and you need to observe scattering at vertices and spreading across the full network over time.

## When NOT to use

- The problem is an eigenvalue calculation (e.g. Mathieu, Rayleigh-Benard growth rates, pipe-flow eigenvalues) — use Dedalus's solve_dense/solve_sparse paths rather than SBDF2 initial-value integration.
- The PDE is linear and single-domain, so no vertex continuity/Kirchhoff-flux coupling or nonlinear term is needed.
- You only need a steady spectral solve with no time evolution of a traveling excitation.

## Inputs

- Bright soliton initial condition placed on a single graph edge
- Nonlinear Schrodinger equation problem specification on coupled 1D Chebyshev segments with continuity and Kirchhoff-flux vertex conditions
- SBDF2 timestepper configuration (timestep, stop time, simulation output cadence)

## Outputs

- HDF5 (h5py-readable) analysis files containing time snapshots of the complex field on each network segment
- Matplotlib plots of the network field at several times showing soliton translation, vertex scattering, and network-wide dispersion

## How to apply

Locate the bundled NLS network example under dedalus/src/dedalus/examples, which constructs a coupled multi-segment 1D Chebyshev problem encoding the nonlinear Schrodinger equation with continuity and Kirchhoff-flux boundary conditions imposed at the network vertices. Run the script with python3 (optionally under mpirun for MPI parallelization) and time-integrate the initial bright soliton with Dedalus's SBDF2 semi-implicit stepper through its vertex scattering events. Configure the simulation output handler to write evolving field snapshots to HDF5 analysis files. Load the HDF5 output with h5py and plot the network field at several times with matplotlib to verify that the soliton first translates to the end of its segment, scatters at the vertices, and eventually fills the entire graph network.

## Related tools

- **Dedalus** (Constructs the coupled multi-segment Chebyshev NLS problem, applies continuity and Kirchhoff-flux boundary conditions at vertices, and time-integrates with SBDF2 while writing HDF5 analysis snapshots) — https://github.com/DedalusProject/dedalus
- **numpy** (Defines the bright soliton initial condition and manipulates field arrays when inspecting output)
- **h5py** (Loads the HDF5 analysis files written by the simulation output handler for post-hoc plotting) — https://github.com/h5py/h5py
- **matplotlib** (Plots the network field at several simulation times to confirm scattering and dispersion)

## Examples

```
mpirun -n 4 python3 dedalus/src/dedalus/examples/nls_graph.py
```

## Evaluation signals

- The soliton first translates to the end of its initial segment before interacting with vertices
- After the first vertex encounter, the excitation scatters into the other segments rather than reflecting entirely back
- Over the simulated time interval the field amplitude spreads to fill the entire graph network, matching Fig. 10
- Continuity and Kirchhoff-flux boundary conditions hold at every vertex in each HDF5 snapshot
- HDF5 analysis files are readable with h5py and contain a consistent time series of segment-wise snapshots

## Limitations

- The graph-coupled NLS setup requires continuity and Kirchhoff-flux conditions at every vertex; omitting them breaks the multi-segment coupling.
- Runs intended for MPI should be launched under mpirun; serial python3 execution will not parallelize the coupled solve.
- SBDF2 is only second-order, so large timesteps relative to soliton speed may smear the scattering event.

## Evidence

- [abstract] NLS example purpose: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large"
- [abstract] Dedalus translates equations to solvers: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [intro] Python interface with symbolic equations: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [intro] Dedalus role for spectral PDEs: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [readme] h5py role in reading HDF5 output: [evidence span withheld — anchor status: external_doc]
