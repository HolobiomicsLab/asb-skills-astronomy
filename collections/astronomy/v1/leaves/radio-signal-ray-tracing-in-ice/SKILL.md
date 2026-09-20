---
name: radio-signal-ray-tracing-in-ice
description: 'Use when simulating in-ice radio neutrino detection: you have neutrino-interaction
  vertex positions and shower parameters from the event-generator stage and need valid
  ray-path solutions (path length, arrival time, amplitude factor) to each antenna
  position under a given ice refractive-index.'
when_to_use_negative:
- You are modelling the Askaryan radio emission itself or the neutrino interaction
  — those are upstream NuRadioMC simulation stages, not propagation
- You are running detector-response or trigger simulation — NuRadioReco is used by
  NuRadioMC for that stage, not the ray tracer
- You need propagation in an arbitrary 3D index-of-refraction profile with the default
  analytic ray tracer — use the RadioPropa numerical ray tracer instead
edam_operation: http://edamontology.org/operation_2426
tools:
- name: NuRadioMC
  role: Provides the signal-propagation module that ray-traces each interaction vertex
    to each antenna and enumerates valid ray-path solutions in ice
  repo: github.com/nu-radio/NuRadioMC
- name: RadioPropa
  role: Numerical ray tracer added to allow signal propagation in arbitrary 3D index-of-refraction
    profiles
- name: Python
  role: Language of the modular NuRadioMC framework used to load event-generator output
    and run the propagation stage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-020-7612-8
    title: 'NuRadioMC: simulating the radio emission of neutrinos from interaction
      to detector'
  source_content_hash: sha256:994148d62abcde13c9ce8dbd4af6ba73d944cd198da66fe85cd944199940840a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NuRadioMC
  - Python
  - RadioPropa
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# radio-signal-ray-tracing-in-ice

## Summary

Ray-trace Askaryan radio emission from each neutrino-interaction vertex in ice to every detector antenna using NuRadioMC's signal-propagation stage, enumerating valid ray-path solutions with path length, arrival time, and amplitude factor. This is the propagation link between the event generator and detector-response simulation in the NuRadioMC chain.

## When to use

Use when simulating in-ice radio neutrino detection: you have neutrino-interaction vertex positions and shower parameters from the event-generator stage and need valid ray-path solutions (path length, arrival time, amplitude factor) to each antenna position under a given ice refractive-index profile.

## When NOT to use

- You are modelling the Askaryan radio emission itself or the neutrino interaction — those are upstream NuRadioMC simulation stages, not propagation
- You are running detector-response or trigger simulation — NuRadioReco is used by NuRadioMC for that stage, not the ray tracer
- You need propagation in an arbitrary 3D index-of-refraction profile with the default analytic ray tracer — use the RadioPropa numerical ray tracer instead

## Inputs

- Event-generator output from the neutrino-interaction simulation stage (vertex positions and shower parameters)
- Antenna positions from the detector description
- Refractive-index profile of the medium (ice/firn)

## Outputs

- ray_tracing_solutions.hdf5 — per-vertex, per-antenna table of ray-path solutions with path length, signal arrival time, and amplitude factor, written using the NuRadioReco/utilities/units convention

## How to apply

Load the event-generator output (vertex positions and shower parameters) with the NuRadioMC Python framework, then define antenna positions from the detector description and the refractive-index profile of the ice. Run the NuRadioMC signal-propagation module to ray-trace every interaction vertex to every antenna, enumerating the valid ray-path solutions in the medium. For each ray solution, record the path length, the signal arrival time, and the amplitude factor applied to the Askaryan emission at the vertex. Write the per-vertex, per-antenna solution table to ray_tracing_solutions.hdf5 using the NuRadioReco/utilities/units convention for all unit-bearing quantities. Note that the README directs users to the arXiv version of the paper for improved ray-tracing equations, and that arbitrary 3D index-of-refraction profiles require the numerical ray tracer RadioPropa rather than the default analytic tracer.

## Related tools

- **NuRadioMC** (Provides the signal-propagation module that ray-traces each interaction vertex to each antenna and enumerates valid ray-path solutions in ice) — github.com/nu-radio/NuRadioMC
- **RadioPropa** (Numerical ray tracer added to allow signal propagation in arbitrary 3D index-of-refraction profiles)
- **Python** (Language of the modular NuRadioMC framework used to load event-generator output and run the propagation stage)

## Examples

```
pip install NuRadioMC
```

## Evaluation signals

- ray_tracing_solutions.hdf5 exists and is a valid HDF5 file
- The file contains path-length, arrival-time, and amplitude-factor fields for each vertex-antenna pair
- Each vertex-antenna pair has at least one enumerated ray solution where a propagation path exists in the medium
- All unit-bearing quantities conform to the NuRadioReco/utilities/units convention
- Every interaction vertex is traced to every antenna position (complete vertex-to-antenna enumeration)

## Limitations

- The published (2020) paper's ray-tracing equations are superseded: the README advises referring to the arXiv version (arXiv:1906.01670) for improved ray-tracing equations and FFT normalization as used in the current NuRadioMC version
- The default analytic ray tracer does not handle arbitrary 3D index-of-refraction profiles; RadioPropa was added for that purpose (Oeyen et al., PoS(ICRC2021)1027)
- Vertex-antenna pairs with no propagation path in the medium yield no ray solution and must be handled as empty entries

## Evidence

- [abstract] Full-chain simulation including propagation: "simulates everything from the neutrino interaction in a medium, the subsequent Askaryan radio emission, the propagation of the radio signal to the detector and finally the detector response"
- [readme] Improved ray-tracing equations in arXiv version: [evidence span withheld — anchor status: external_doc]
- [readme] RadioPropa for arbitrary 3D n-profiles: [evidence span withheld — anchor status: external_doc]
- [intro] NuRadioMC as Monte Carlo package for radio neutrino detectors: "**NuRadioMC**: A Monte Carlo simulation package for radio neutrino detectors."
