---
name: inice-medium-propagation-setup
description: Use when a NuRadioMC simulation must specify the in-ice propagation medium
  and ray tracer (e.g., a firn index-of-refraction profile) so Askaryan signals travel
  from the interaction vertex to the detector.
when_to_use_negative:
- The goal is reconstruction or detector/trigger simulation of measured data — NuRadioReco
  is the independent framework used for detector and trigger simulation, not this
  medium-propagation setup.
- The simulation stage is the neutrino interaction/event generation itself (vertex
  sampling and flux weighting), which precedes the propagation stage.
- 'The medium is not in-ice: NuRadioMC is scoped as a Monte Carlo simulation package
  for radio neutrino detectors with the neutrino interaction in a medium (ice), so
  air-shower or non-ice propagation setups fall outside this skill.'
edam_operation: http://edamontology.org/operation_3562
tools:
- name: NuRadioMC
  role: 'Hosts the propagation stage: reads the medium definition from the simulation
    configuration and ray-traces the Askaryan radio signal from the interaction vertex
    to the detector'
  repo: github.com/nu-radio/NuRadioMC
- name: RadioPropa
  role: Numerical ray tracer added to NuRadioMC to allow signal propagation in arbitrary
    3D index-of-refraction profiles and to test firn ice models
- name: NuRadioReco
  role: Provides the units convention (NuRadioReco/utilities/units) for medium and
    signal quantities and performs the downstream detector and trigger simulation
  repo: github.com/nu-radio/NuRadioMC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1140/epjc/s10052-020-7612-8
    title: 'NuRadioMC: simulating the radio emission of neutrinos from interaction
      to detector'
  source_content_hash: sha256:10cccaf9b2ec689fe80678d5405c76e337021fd5ef13f5a1da021c6bd15734fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NuRadioMC
  - NuRadioReco
  - Python
  - RadioPropa
  edam_operation: http://edamontology.org/operation_3562
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# inice-medium-propagation-setup

## Summary

Defines the in-ice propagation medium (index-of-refraction/firn profile) and ray-tracing setup in NuRadioMC so that Askaryan radio emission from the neutrino interaction is correctly propagated to the detector before the detector-response stage.

## When to use

Use when a NuRadioMC simulation must specify the in-ice propagation medium and ray tracer (e.g., a firn index-of-refraction profile) so Askaryan signals travel from the interaction vertex to the detector.

## When NOT to use

- The goal is reconstruction or detector/trigger simulation of measured data — NuRadioReco is the independent framework used for detector and trigger simulation, not this medium-propagation setup.
- The simulation stage is the neutrino interaction/event generation itself (vertex sampling and flux weighting), which precedes the propagation stage.
- The medium is not in-ice: NuRadioMC is scoped as a Monte Carlo simulation package for radio neutrino detectors with the neutrino interaction in a medium (ice), so air-shower or non-ice propagation setups fall outside this skill.

## Inputs

- NuRadioMC simulation configuration file specifying the propagation medium definition (index-of-refraction/firn profile) and ray-tracing settings
- .nur event file with sampled neutrino interaction vertices and shower parameters from the event generator
- Detector geometry (antenna positions) to which the radio signal must be ray-traced

## Outputs

- Ray-traced propagation paths and signal properties for the configured medium, carried in the .nur event file
- Radio signals at detector positions ready for the NuRadioReco-based detector and trigger simulation stage

## How to apply

Parse the NuRadioMC simulation configuration file with Python to extract the propagation medium definition alongside the neutrino energy range and flux model choice, expressing all dimensional quantities through the NuRadioReco/utilities/units convention. Select the ray tracer appropriate to the medium: the built-in ray tracer handles the standard in-ice index-of-refraction profiles used by NuRadioMC, while the RadioPropa numerical ray tracer is required for signal propagation in arbitrary 3D index-of-refraction profiles; firn-model choice matters, as Oeyen et al. quantified the effects of firn ice models on radio neutrino simulations with RadioPropa. The medium setup sits between the event-generator output (.nur file with sampled vertices and shower parameters) and the downstream detector and trigger simulation that NuRadioMC delegates to NuRadioReco. For the ray-tracing equations actually implemented in the current version, consult the arXiv version of the paper, which documents improved ray-tracing equations and FFT normalization. If polarization effects in the firn must be modelled, enable the birefringence module added by Heyer & Glaser (2023).

## Related tools

- **NuRadioMC** (Hosts the propagation stage: reads the medium definition from the simulation configuration and ray-traces the Askaryan radio signal from the interaction vertex to the detector) — github.com/nu-radio/NuRadioMC
- **RadioPropa** (Numerical ray tracer added to NuRadioMC to allow signal propagation in arbitrary 3D index-of-refraction profiles and to test firn ice models)
- **NuRadioReco** (Provides the units convention (NuRadioReco/utilities/units) for medium and signal quantities and performs the downstream detector and trigger simulation) — github.com/nu-radio/NuRadioMC

## Evaluation signals

- The parsed configuration resolves a named propagation medium definition consistent with the chosen ray tracer (built-in tracer for standard profiles, RadioPropa for arbitrary 3D profiles).
- Ray-tracing solutions are found between the simulated vertices and the receiver antennas within the defined index-of-refraction profile, with no unexplained ray-tracing failures.
- All dimensional quantities in the medium setup follow the NuRadioReco/utilities/units convention, so propagated amplitudes and times are physically consistent downstream.
- The resulting .nur file is loadable and feeds the NuRadioReco-based detector and trigger simulation without stage-mismatch errors.
- Comparing results across firn ice models (as in the Oeyen et al. RadioPropa study) changes the propagation output only in physically expected ways, confirming the medium is applied as configured.

## Limitations

- Propagation in arbitrary 3D index-of-refraction profiles requires the RadioPropa numerical ray tracer, which was added later than the original NuRadioMC release; the built-in tracer is limited to the profiles it supports.
- Birefringence effects in the ice are only available through the modelling added by Heyer & Glaser (2023); earlier versions of NuRadioMC lack it.
- The published paper's ray-tracing equations differ from the current implementation — the README instructs users to refer to the arXiv version for improved ray-tracing equations and FFT normalization.

## Evidence

- [abstract] Full simulation chain including the propagation stage: "simulates everything from the neutrino interaction in a medium, the subsequent Askaryan radio emission, the propagation of the radio signal to the detector and finally the detector response"
- [other] Medium definition is extracted from the simulation configuration: [evidence span withheld — anchor status: not_found]
- [readme] RadioPropa extends medium handling to arbitrary 3D index-of-refraction profiles: [evidence span withheld — anchor status: external_doc]
- [readme] Firn ice model choice affects radio neutrino simulations: [evidence span withheld — anchor status: external_doc]
- [readme] Refer to arXiv version for current ray-tracing equations: [evidence span withheld — anchor status: external_doc]
- [intro] Detector and trigger simulation is handled by NuRadioReco: "NuRadioReco is independent of NuRadioMC and used by NuRadioMC for the detector and trigger simulation."
