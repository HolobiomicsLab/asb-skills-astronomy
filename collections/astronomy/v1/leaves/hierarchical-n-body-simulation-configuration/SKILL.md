---
name: hierarchical-n-body-simulation-configuration
description: Use when simulating hierarchical astrophysical systems (e.g. star clusters
  hosting planetary systems or binary stars) with extreme scale separations, where
  direct N-body integration is too costly and subsystem isolation with Ph4/Huayno
  is appropriate.
when_to_use_negative:
- The system is a single-scale N-body problem with no hierarchical subsystems — a
  direct integrator such as Ph4 alone is the appropriate comparison, as done in the
  cluster test with RUN_NEMESIS = 0
- The child integrators must remain under MPI-synchronised parallel control — Nemesis
  deliberately uses sockets for children (not MPI), so it is unsuited to workflows
  requiring constant MPI process synchronisation
- Stellar evolution or galactic tidal fields must be modelled in detail during the
  ZKL validation run — the suggested test protocol requires turning off galactic field,
  stellar evolution, and children collisions
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Nemesis
  role: Multi-scale integration framework being configured and run; splits parents/children
    and synchronises scales via correction kicks
  repo: https://github.com/ErwanH29/nemesis
- name: AMUSE
  role: Framework embedding Nemesis and providing the integrator community codes it
    drives
  repo: https://amuse.readthedocs.io/en/latest/
- name: Ph4
  role: Cluster-optimised direct N-body integrator used for the global parent system
    and as direct N-body comparison
- name: amuse-huayno (Huayno)
  role: Suggested integrator for both parent (_parent_worker) and children (_sub_worker)
    in SHARED10_COLLISIONS mode during the ZKL validation test
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202558060
    title: 'Nemesis: A multi-scale, multi-physics algorithm for astrophysics'
  source_content_hash: sha256:791d244b6f51c0e8d4cf9d3c555a8042e1d26a24b18475baa673188cd878be07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ErwanH29/nemesis
  tools:
  - AMUSE
  - Nemesis
  - Ph4
  - amuse-huayno (Huayno)
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hierarchical-n-body-simulation-configuration

## Summary

Configure and run Nemesis, an AMUSE-embedded multi-scale N-body framework that decouples fast-evolving child subsystems from parent systems, integrating each scale separately and synchronising them via second-order kick-drift-kick correction kicks.

## When to use

Use when simulating hierarchical astrophysical systems (e.g. star clusters hosting planetary systems or binary stars) with extreme scale separations, where direct N-body integration is too costly and subsystem isolation with Ph4/Huayno is appropriate.

## When NOT to use

- The system is a single-scale N-body problem with no hierarchical subsystems — a direct integrator such as Ph4 alone is the appropriate comparison, as done in the cluster test with RUN_NEMESIS = 0
- The child integrators must remain under MPI-synchronised parallel control — Nemesis deliberately uses sockets for children (not MPI), so it is unsuited to workflows requiring constant MPI process synchronisation
- Stellar evolution or galactic tidal fields must be modelled in detail during the ZKL validation run — the suggested test protocol requires turning off galactic field, stellar evolution, and children collisions

## Inputs

- Hierarchical particle set with parents and children (children requiring a syst_id attribute > 0), stored in the ICs/ folder as HDF5
- Worker configuration in src/nemesis.py (_parent_worker, _sub_worker, _stellar_worker)
- Free parameters in src/globals.py (e.g. PARENT_RADIUS_COEFF, EPS, SPLIT_PARAM)
- Command-line run parameters for main.py (e.g. --tend, --dtbridge)

## Outputs

- HDF5 particle snapshots in data/simulation_snapshot/
- Collision event logs in data/collision_snapshot/
- Runtime diagnostics in data/sim_stats/

## How to apply

Clone github.com/ErwanH29/nemesis, install dependencies (conda install --file requirements.txt) plus the recommended AMUSE packages huayno, ph4, seba, symple, kepler, then compile the C++ correction-kick files with cd src/cpp && make in the same Python environment used to run Nemesis. Generate hierarchical initial conditions with a particle_initialiser.py script (saved under ICs/, with children carrying a syst_id attribute > 0), and configure the workers in src/nemesis.py: _parent_worker for the cluster-optimised parent integrator (Ph4 or Huayno), _sub_worker for the child integrator, and _stellar_worker for stellar evolution — any worker change must be mirrored in _worker_list. Tune free parameters in src/globals.py, notably PARENT_RADIUS_COEFF, PARENT_RADIUS_MIN/MAX, EPS, and SPLIT_PARAM, then launch via python main.py (e.g. --tend=1Myr --dtbridge=100yr). Scales are synchronised by the kick-drift-kick bridge and the C++ grav_correctors, and outputs are written under data/ as simulation_snapshot/ HDF5 files, collision_snapshot/ logs, and sim_stats/ diagnostics.

## Related tools

- **Nemesis** (Multi-scale integration framework being configured and run; splits parents/children and synchronises scales via correction kicks) — https://github.com/ErwanH29/nemesis
- **AMUSE** (Framework embedding Nemesis and providing the integrator community codes it drives) — https://amuse.readthedocs.io/en/latest/
- **Ph4** (Cluster-optimised direct N-body integrator used for the global parent system and as direct N-body comparison)
- **amuse-huayno (Huayno)** (Suggested integrator for both parent (_parent_worker) and children (_sub_worker) in SHARED10_COLLISIONS mode during the ZKL validation test)

## Examples

```
python main.py --tend=1Myr --dtbridge=100yr
```

## Evaluation signals

- ZKL test: secular-evolution traces are invariant across PARENT_RADIUS_COEFF = 1e-5, 100, and 1000 au in src/globals.py, matching the direct N-body (Ph4) result
- Energy conservation is improved or comparable relative to the direct N-body comparison over the run
- Cluster test: results with RUN_NEMESIS = 1 are indistinguishable from direct N-body (RUN_NEMESIS = 0) on both global and local scales
- Output directories data/simulation_snapshot/, data/collision_snapshot/, and data/sim_stats/ are populated with snapshots, collision logs, and diagnostics
- Any change to a worker is reflected in _worker_list, otherwise runtime handling of integrators fails

## Limitations

- Nemesis must be run from the same Python environment used to compile the C++ extension
- Runs can be resumed, but if simulation parameters are changed when resuming, only a warning is printed and the simulation proceeds anyway
- Sockets (not MPI) are used for child subsystems; interrupting or suspending workers mid-communication may lead to crashes, especially with a large number of children
- Wall-clock time scales roughly as t_sim ∝ 1/δt_nem; it stays flat only while cores exceed the number of systems, and grows at worst linearly with excess systems
- Stellar radii in snapshots correspond to ZAMS radii, not evolved radii

## Evidence

- [readme] Nemesis works by dynamically decoupling subsystems from their parent environment.: "Nemesis works by dynamically decoupling subsystems from their parent environment."
- [readme] The global system (parents) is integrated with a cluster-optimised integrator (i.e Ph4).: "The global system ("parents") is integrated with a cluster-optimised integrator (i.e Ph4``"
- [intro] Synchronisation between scales occurs through a second-order kick-drift-kick scheme.: "Synchronisation between scales (children and parents) occurs through a second-order kick-drift-kick scheme."
- [abstract] The Nemesis code yields indistinguishable results on both the global and local scales when compared with the direct N-body code Ph4.: "The Nemesis code yields indistinguishable results on both the global and local scales when compared with the direct N -body code Ph4"
- [readme] Children require a syst_id particle attribute, with syst_id > 0 forming children systems.: "Children require a syst_id particle attribute, with syst_id > 0 forming children systems."
- [readme] Change PARENT_RADIUS_COEFF in src/globals to 1e-5 au, 100 au and 1000 au.: "Change `PARENT_RADIUS_COEFF` in `src/globals` to 1e-5 au, 100 au and 1000 au."
