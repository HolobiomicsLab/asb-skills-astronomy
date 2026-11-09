---
name: radio-neutrino-detector-simulation
description: Use when you need to simulate neutrino-induced radio signals from interaction
  through trigger with NuRadioMC/NuRadioReco, compare Askaryan emission parameterizations,
  or compute effective volumes for detector design decisions.
when_to_use_negative:
- The task is data analysis or event reconstruction of already-recorded detector data
  rather than forward simulation — that is NuRadioReco's reconstruction role, not
  NuRadioMC's simulation chain
- The emission model comparison is not the goal and shower/detector configurations
  would vary between runs, making per-model spectra non-comparable
- Signals originate from cosmic-ray air showers in air rather than neutrino-induced
  showers in ice, outside NuRadioMC's in-ice neutrino detector scope
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: NuRadioMC
  role: Monte Carlo simulation of the neutrino interaction in the medium, Askaryan
    radio emission, and signal propagation; executed once per emission model under
    a fixed configuration
  repo: github.com/nu-radio/NuRadioMC
- name: NuRadioReco
  role: Independent framework used by NuRadioMC to perform the detector and trigger
    simulation on the propagated radio field traces
  repo: github.com/nu-radio/NuRadioMC
- name: Python
  role: Modular Python-based implementation language in which the simulation chain
    and its configuration are driven
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-020-7612-8
    title: 'NuRadioMC: simulating the radio emission of neutrinos from interaction
      to detector'
  source_content_hash: sha256:726ba086b1378d4381425166171c1342b556c255349a5ed74c3e74829da87e0b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NuRadioMC
  - NuRadioReco
  - Python
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# radio-neutrino-detector-simulation

## Summary

End-to-end Monte Carlo simulation of in-ice ultra-high-energy neutrino detectors with NuRadioMC and NuRadioReco: neutrino interaction, Askaryan radio emission, signal propagation, and detector/trigger response. It also enables systematic comparison of Askaryan emission models under a fixed shower/detector configuration.

## When to use

Use when you need to simulate neutrino-induced radio signals from interaction through trigger with NuRadioMC/NuRadioReco, compare Askaryan emission parameterizations, or compute effective volumes for detector design decisions.

## When NOT to use

- The task is data analysis or event reconstruction of already-recorded detector data rather than forward simulation — that is NuRadioReco's reconstruction role, not NuRadioMC's simulation chain
- The emission model comparison is not the goal and shower/detector configurations would vary between runs, making per-model spectra non-comparable
- Signals originate from cosmic-ray air showers in air rather than neutrino-induced showers in ice, outside NuRadioMC's in-ice neutrino detector scope

## Inputs

- NuRadioMC example simulation configuration (fixed in-ice neutrino shower, antenna position, viewing angle)
- Available Askaryan emission models/parameterizations in NuRadioMC (one run per model)
- Propagated radio field traces from the signal-propagation stage (for NuRadioReco trigger simulation)
- Detector station description (antenna/trigger configuration)

## Outputs

- emission_model_comparison.csv: one frequency-domain electric-field spectrum entry per emission parameterization, in NuRadioReco units
- trigger_decisions.csv: per-event records with event identifier, station identifier, and triggered flag
- detector_performance.csv: one row per neutrino-energy bin with detected-event counts and effective volumes

## How to apply

Load a NuRadioMC example simulation configuration defining a fixed in-ice neutrino-induced shower and fixed antenna position/viewing angle, then execute the simulation chain once per Askaryan emission model/parameterization while keeping the neutrino interaction, signal propagation, and antenna geometry identical across runs — this is what makes the emission-model comparison meaningful. Extract the frequency-domain electric-field spectrum at the antenna from each model's output, expressing every unit-bearing variable through the NuRadioReco/utilities/units convention, and assemble per-model spectra into a single comparison table (e.g. emission_model_comparison.csv). For the full interaction-to-trigger pipeline, let NuRadioReco perform the detector and trigger simulation within the NuRadioMC run: load the propagated field traces with NuRadioReco's event-reading utilities, apply the detector station description to obtain detector-level signals, and evaluate the trigger modules by applying the trigger threshold per station and event. Aggregate triggered events into neutrino-energy bins, compute the effective volume per bin from detected-event counts and the simulated volume, and write energy-binned counts and effective volumes to detector_performance.csv. The rationale is that NuRadioMC deliberately covers the complete chain from neutrino interaction to detector response, so a single validated run exercises the entire pipeline needed for instrumental design decisions.

## Related tools

- **NuRadioMC** (Monte Carlo simulation of the neutrino interaction in the medium, Askaryan radio emission, and signal propagation; executed once per emission model under a fixed configuration) — github.com/nu-radio/NuRadioMC
- **NuRadioReco** (Independent framework used by NuRadioMC to perform the detector and trigger simulation on the propagated radio field traces) — github.com/nu-radio/NuRadioMC
- **Python** (Modular Python-based implementation language in which the simulation chain and its configuration are driven)

## Examples

```
pip install NuRadioMC
```

## Evaluation signals

- emission_model_comparison.csv contains exactly one spectrum entry per available emission model, with all unit-bearing columns in the NuRadioReco/utilities/units convention
- trigger_decisions.csv exists, contains a decision record for every simulated event, and includes event identifier, station identifier, and triggered flag
- detector_performance.csv parses as a table with one row per neutrino-energy bin, non-negative detected-event counts, and effective volumes spanning the simulated energy range
- Neutrino interaction, signal propagation, and antenna geometry are verified identical across the per-model runs (fixed configuration)
- All simulated events pass through the full chain: interaction, Askaryan emission, propagation, detector response, and trigger

## Limitations

- The published 2020 paper differs from the current NuRadioMC version — the README directs users to the arXiv version for improved ray-tracing equations and FFT-normalization documentation
- Features such as numerical ray tracing with RadioPropa (arbitrary 3D index-of-refraction profiles), birefringence modelling, secondary muon/tau interactions, and multi-shower interference were added in later papers, not the 2020 baseline
- No quantitative performance threshold is reported in the paper for effective volumes; validation is structural (one row per energy bin, non-negative counts)

## Evidence

- [abstract] NuRadioMC simulates the full chain from neutrino interaction through Askaryan radio emission, signal propagation, and detector response: "simulates everything from the neutrino interaction in a medium, the subsequent Askaryan radio emission, the propagation of the radio signal to the detector and finally the detector response"
- [intro] NuRadioReco is the detector and trigger simulation component within NuRadioMC: "NuRadioReco is independent of NuRadioMC
and used by NuRadioMC for the detector and trigger simulation."
- [abstract] NuRadioMC compares and reviews a variety of Askaryan emission models and parameterizations: "A variety of models and parameterizations for the radio emission of neutrino-induced showers are compared and reviewed."
- [readme] NuRadioReco's role as reconstruction and detector simulation framework: "**NuRadioReco**: A reconstruction and detector simulation framework for radio detectors of high-energy neutrinos and cosmic-rays."
- [readme] Cite the arXiv version for up-to-date ray tracing and FFT normalization: [evidence span withheld — anchor status: external_doc]
