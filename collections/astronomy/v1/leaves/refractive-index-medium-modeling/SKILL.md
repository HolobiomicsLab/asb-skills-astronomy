---
name: refractive-index-medium-modeling
description: Use when simulating in-ice radio-neutrino detection where signal propagation
  depends on the medium's depth-dependent index-of-refraction profile, e.g. ray-tracing
  neutrino-interaction vertices to antennas in NuRadioMC.
when_to_use_negative:
- The medium's index-of-refraction profile is arbitrary 3D and you are not prepared
  to use the RadioPropa numerical ray tracer — the default ray tracer is not built
  for that case.
- The task concerns radio-emission modelling or detector/trigger response (handled
  by NuRadioReco within NuRadioMC), not propagation of the signal through the ice.
- Birefringence effects in the ice must be modelled — this requires the separate first-principle
  birefringence calculation added to NuRadioMC by Heyer & Glaser (2023).
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: NuRadioMC
  role: 'Runs the signal-propagation stage: ray-traces every neutrino-interaction
    vertex to every antenna given the medium''s refractive-index profile, enumerating
    valid ray-path solutions'
  repo: https://github.com/nu-radio/NuRadioMC
- name: RadioPropa
  role: Numerical ray tracer enabling signal propagation in arbitrary 3D index-of-refraction
    profiles within NuRadioMC
- name: NuRadioReco
  role: Supplies the utilities/units convention used for all unit-bearing quantities
    written to ray_tracing_solutions.hdf5
  repo: https://github.com/nu-radio/NuRadioMC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-020-7612-8
    title: 'NuRadioMC: simulating the radio emission of neutrinos from interaction
      to detector'
  source_content_hash: sha256:a6b4b52668da5009435e9451b40f6128498e9df135277e86e3d2f280cdad62a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nu-radio/NuRadioMC
  tools:
  - NuRadioMC
  - NuRadioReco
  - Python
  - RadioPropa
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# refractive-index-medium-modeling

## Summary

Define and apply the refractive-index (firn/ice) profile of the propagation medium so that NuRadioMC can ray-trace Askaryan radio signals from neutrino-interaction vertices to every antenna, enumerating valid ray-path solutions with their path lengths, arrival times, and amplitude factors.

## When to use

Use when simulating in-ice radio-neutrino detection where signal propagation depends on the medium's depth-dependent index-of-refraction profile, e.g. ray-tracing neutrino-interaction vertices to antennas in NuRadioMC.

## When NOT to use

- The medium's index-of-refraction profile is arbitrary 3D and you are not prepared to use the RadioPropa numerical ray tracer — the default ray tracer is not built for that case.
- The task concerns radio-emission modelling or detector/trigger response (handled by NuRadioReco within NuRadioMC), not propagation of the signal through the ice.
- Birefringence effects in the ice must be modelled — this requires the separate first-principle birefringence calculation added to NuRadioMC by Heyer & Glaser (2023).

## Inputs

- Event-generator output: neutrino-interaction vertex positions and shower parameters
- Antenna positions from the detector description
- Refractive-index profile of the medium (ice/firn model)
- NuRadioMC Python framework environment

## Outputs

- ray_tracing_solutions.hdf5: per-vertex, per-antenna table of enumerated ray solutions with path length, signal arrival time, and amplitude factor fields

## How to apply

Load the event-generator output (vertex positions and shower parameters) and define antenna positions from the detector description together with the refractive-index profile of the ice. Run the NuRadioMC signal-propagation module to ray-trace every interaction vertex to every antenna, enumerating the valid ray-path solutions in the medium; for arbitrary 3D index-of-refraction profiles, use the numerical ray tracer RadioPropa added to NuRadioMC. For each ray solution, record the path length, the signal arrival time, and the amplitude factor applied to the Askaryan emission at the vertex. Write the per-vertex, per-antenna solution table to ray_tracing_solutions.hdf5 using the NuRadioReco/utilities/units convention for all unit-bearing quantities. Validate the output and, for equation fidelity, refer to the arXiv version of the NuRadioMC paper, which documents improved ray-tracing equations as used in the current NuRadioMC version.

## Related tools

- **NuRadioMC** (Runs the signal-propagation stage: ray-traces every neutrino-interaction vertex to every antenna given the medium's refractive-index profile, enumerating valid ray-path solutions) — https://github.com/nu-radio/NuRadioMC
- **RadioPropa** (Numerical ray tracer enabling signal propagation in arbitrary 3D index-of-refraction profiles within NuRadioMC)
- **NuRadioReco** (Supplies the utilities/units convention used for all unit-bearing quantities written to ray_tracing_solutions.hdf5) — https://github.com/nu-radio/NuRadioMC

## Examples

```
from NuRadioReco.utilities import units  # unit convention for ray_tracing_solutions.hdf5 path-length, arrival-time, and amplitude-factor fields
```

## Evaluation signals

- ray_tracing_solutions.hdf5 exists and is a valid HDF5 file.
- The file contains path-length, arrival-time, and amplitude-factor fields for each vertex-antenna pair.
- Each vertex-antenna pair with a propagation path in the medium has at least one enumerated ray solution.
- All unit-bearing quantities in the output follow the NuRadioReco/utilities/units convention.
- Ray-tracing equations match the improved equations documented in the arXiv version as used in the current NuRadioMC version.

## Limitations

- The published paper's ray-tracing equations are superseded: the arXiv version should be consulted for improved ray-tracing equations and FFT normalization as used in the current NuRadioMC version.
- Propagation in arbitrary 3D index-of-refraction profiles requires the RadioPropa ray tracer, which was added later (Oeyen et al., PoS(ICRC2021)1027).
- Birefringence effects are not part of the baseline propagation model and require the dedicated first-principle calculation of Heyer & Glaser (2023).
- Some vertex-antenna pairs have no enumerated ray solution because no propagation path exists in the medium.

## Evidence

- [abstract] NuRadioMC simulates the full chain from interaction to detector: "simulates everything from the neutrino interaction in a medium, the subsequent Askaryan radio emission, the propagation of the radio signal to the detector and finally the detector response"
- [readme] RadioPropa enables arbitrary 3D index-of-refraction profiles: [evidence span withheld — anchor status: external_doc]
- [readme] Use the arXiv version for improved ray-tracing equations: [evidence span withheld — anchor status: external_doc]
- [readme] In-situ calibration measures the index-of-refraction profile: [evidence span withheld — anchor status: external_doc]
