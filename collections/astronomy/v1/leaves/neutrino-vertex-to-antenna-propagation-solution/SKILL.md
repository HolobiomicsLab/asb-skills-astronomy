---
name: neutrino-vertex-to-antenna-propagation-solution
description: 'Use when simulating ultra-high-energy neutrino detection in ice: you
  have event-generator output (vertex positions, shower parameters) and antenna positions
  from the detector description, and need per-vertex/per-antenna ray-path solutions
  before detector simulation.'
when_to_use_negative:
- The Askaryan radio emission at the vertex has not yet been modelled — propagation
  follows, not replaces, the emission stage of the NuRadioMC chain
- You need detector response or trigger simulation — that stage is handled by NuRadioReco,
  which is used by NuRadioMC for detector and trigger simulation
- Radio emission from cosmic-ray air showers rather than in-ice neutrino vertex-to-antenna
  propagation is the goal
edam_operation: http://edamontology.org/operation_2426
tools:
- name: NuRadioMC
  role: Provides the signal-propagation (ray-tracing) module that enumerates ray-path
    solutions from each neutrino interaction vertex to each antenna in the ice
  repo: https://github.com/nu-radio/NuRadioMC
- name: Python
  role: The framework language; the simulation chain and the ray_tracing_solutions.hdf5
    output workflow are driven via the NuRadioMC Python framework
  repo: https://github.com/nu-radio/NuRadioMC
- name: NuRadioReco
  role: Supplies the units convention (NuRadioReco/utilities/units) used for all unit-bearing
    quantities in the ray-tracing solution table; independent of NuRadioMC but used
    by NuRadioMC
  repo: https://github.com/nu-radio/NuRadioMC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-020-7612-8
    title: 'NuRadioMC: simulating the radio emission of neutrinos from interaction
      to detector'
  source_content_hash: sha256:575447a536909a18dbfa5daedd4f4bb6e35fdfe954d42f4c84bc07f09854da39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nu-radio/NuRadioMC
  tools:
  - NuRadioMC
  - NuRadioReco
  - Python
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# neutrino-vertex-to-antenna-propagation-solution

## Summary

Ray-trace every neutrino interaction vertex to every detector antenna in the ice using the NuRadioMC signal-propagation module, enumerating all valid ray-path solutions under the medium's refractive-index profile. This is the signal-propagation stage of the NuRadioMC Monte Carlo chain between Askaryan emission modelling and detector response.

## When to use

Use when simulating ultra-high-energy neutrino detection in ice: you have event-generator output (vertex positions, shower parameters) and antenna positions from the detector description, and need per-vertex/per-antenna ray-path solutions before detector simulation.

## When NOT to use

- The Askaryan radio emission at the vertex has not yet been modelled — propagation follows, not replaces, the emission stage of the NuRadioMC chain
- You need detector response or trigger simulation — that stage is handled by NuRadioReco, which is used by NuRadioMC for detector and trigger simulation
- Radio emission from cosmic-ray air showers rather than in-ice neutrino vertex-to-antenna propagation is the goal

## Inputs

- Event-generator output from the NuRadioMC neutrino-interaction simulation stage (vertex positions and shower parameters)
- Antenna positions from the detector description
- Refractive-index profile of the medium (ice/firn)

## Outputs

- ray_tracing_solutions.hdf5 — per-vertex, per-antenna table with ray solutions, path lengths, arrival times, and amplitude factors

## How to apply

Load the event-generator output from the neutrino-interaction stage of the NuRadioMC Python framework, which supplies vertex positions and shower parameters. Define antenna positions from the detector description together with the refractive-index (firn/ice) profile of the medium. Run the NuRadioMC signal-propagation module to ray-trace every interaction vertex to every antenna, enumerating all valid ray-path solutions in the medium — NuRadioMC provides a revisited analytical ray tracer, and RadioPropa can be added for numerical tracing through arbitrary 3D index-of-refraction profiles. For each ray solution record the path length, signal arrival time, and the amplitude factor applied to the Askaryan emission at the vertex, using the NuRadioReco/utilities/units convention for all unit-bearing quantities. Write the per-vertex, per-antenna solution table to ray_tracing_solutions.hdf5; validation requires the file to exist, be valid HDF5, and contain path-length, arrival-time, and amplitude-factor fields for each vertex-antenna pair with at least one enumerated ray solution where a propagation path exists.

## Related tools

- **NuRadioMC** (Provides the signal-propagation (ray-tracing) module that enumerates ray-path solutions from each neutrino interaction vertex to each antenna in the ice) — https://github.com/nu-radio/NuRadioMC
- **Python** (The framework language; the simulation chain and the ray_tracing_solutions.hdf5 output workflow are driven via the NuRadioMC Python framework) — https://github.com/nu-radio/NuRadioMC
- **NuRadioReco** (Supplies the units convention (NuRadioReco/utilities/units) used for all unit-bearing quantities in the ray-tracing solution table; independent of NuRadioMC but used by NuRadioMC) — https://github.com/nu-radio/NuRadioMC

## Evaluation signals

- ray_tracing_solutions.hdf5 exists and is a valid HDF5 file
- The file contains path-length, arrival-time, and amplitude-factor fields for each vertex-antenna pair
- At least one ray solution is present for each vertex-antenna pair where a propagation path exists in the medium
- All unit-bearing quantities follow the NuRadioReco/utilities/units convention
- Ray-tracing equations match the improved equations documented in the arXiv version of the paper (arXiv:1906.01670), per the README citation note

## Limitations

- Vertices with no propagation path to an antenna in the given refractive-index profile yield no ray solution — absence of a solution is a valid outcome, not an error
- The default revisited ray tracer is analytic; arbitrary 3D index-of-refraction profiles require the numerical ray tracer RadioPropa, added via the Oeyen et al. PoS(ICRC2021)1027 work
- The README notes the arXiv version should be consulted for improved ray-tracing equations and FFT normalization as used in the current NuRadioMC version
- Birefringence effects on in-ice propagation are modelled only through the separate Heyer & Glaser first-principle calculation added to NuRadioMC

## Evidence

- [abstract] Full simulation chain including propagation stage: "simulates everything from the neutrino interaction in a medium, the subsequent Askaryan radio emission, the propagation of the radio signal to the detector and finally the detector response"
- [readme] Tool identity and scope: "**NuRadioMC**: A Monte Carlo simulation package for radio neutrino detectors."
- [readme] Improved ray-tracing equations pointer: [evidence span withheld — anchor status: external_doc]
- [readme] Numerical ray tracer for arbitrary 3D refractivity profiles: [evidence span withheld — anchor status: external_doc]
- [intro] NuRadioReco independence and role within NuRadioMC: "NuRadioReco is independent of NuRadioMC and used by NuRadioMC for the detector and trigger simulation."
