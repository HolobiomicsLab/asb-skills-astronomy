---
name: electric-field-spectrum-extraction
description: Use when NuRadioMC simulation outputs for a fixed in-ice neutrino shower
  must be reduced to per-model frequency-domain electric-field spectra at an antenna,
  e.g. to compare Askaryan emission parameterizations.
when_to_use_negative:
- The goal is reconstruction or trigger simulation performance rather than comparison
  of Askaryan emission models for neutrino-induced showers.
- The configuration varies the shower energy, antenna position, or viewing angle between
  runs — spectral differences would then not isolate the emission model.
- The analysis concerns cosmic-ray (air-shower) radio emission rather than neutrino-induced
  in-ice showers, which is a different simulation setup.
edam_operation: http://edamontology.org/operation_0244
tools:
- name: NuRadioMC
  role: Monte Carlo simulation package used to run the fixed-shower simulation once
    per Askaryan emission model and produce the spectra being compared
  repo: https://github.com/nu-radio/NuRadioMC
- name: NuRadioReco
  role: Provides the detector and trigger simulation within NuRadioMC and the units
    convention (NuRadioReco/utilities/units) used for all unit-bearing extracted quantities
  repo: https://github.com/nu-radio/NuRadioMC
- name: Python
  role: Modular Python framework language in which the simulation runs and the spectrum-extraction/aggregation
    script is written
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-020-7612-8
    title: 'NuRadioMC: simulating the radio emission of neutrinos from interaction
      to detector'
  source_content_hash: sha256:2c063ce584a8842e837daa4368e5a1c64caf48adcfaf15c4d595c49324152cb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nu-radio/NuRadioMC
  tools:
  - NuRadioMC
  - NuRadioReco
  - Python
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# electric-field-spectrum-extraction

## Summary

Extracts the frequency-domain electric-field spectrum at a fixed antenna from NuRadioMC simulations of an in-ice neutrino-induced shower, once per available Askaryan emission model/parameterization, enabling a like-for-like comparison of emission models under identical shower and detector geometry.

## When to use

Use when NuRadioMC simulation outputs for a fixed in-ice neutrino shower must be reduced to per-model frequency-domain electric-field spectra at an antenna, e.g. to compare Askaryan emission parameterizations.

## When NOT to use

- The goal is reconstruction or trigger simulation performance rather than comparison of Askaryan emission models for neutrino-induced showers.
- The configuration varies the shower energy, antenna position, or viewing angle between runs — spectral differences would then not isolate the emission model.
- The analysis concerns cosmic-ray (air-shower) radio emission rather than neutrino-induced in-ice showers, which is a different simulation setup.

## Inputs

- NuRadioMC example simulation configuration defining one fixed in-ice neutrino-induced shower (interaction, propagation, and antenna geometry)
- Antenna position and viewing angle specification held constant across runs
- Set of available Askaryan emission models/parameterizations implemented in NuRadioMC

## Outputs

- emission_model_comparison.csv — one frequency-domain electric-field spectrum entry per emission parameterization, all unit-bearing columns in the NuRadioReco/utilities/units convention

## How to apply

Load the NuRadioMC example simulation configuration that defines the fixed in-ice neutrino-induced shower, fixed antenna position, and fixed viewing angle. Run the NuRadioMC simulation chain — neutrino interaction, Askaryan emission, signal propagation, detector response — once per available emission model, keeping everything identical across runs except the emission parameterization. Extract each model's frequency-domain electric-field spectrum at the antenna from the simulation output. Express every unit-bearing variable using the NuRadioReco/utilities/units convention, then assemble one table row per emission parameterization into emission_model_comparison.csv. The fixed single-shower, single-antenna configuration is the rationale: any spectral difference between rows is attributable to the emission model alone, not to the event generator or propagation.

## Related tools

- **NuRadioMC** (Monte Carlo simulation package used to run the fixed-shower simulation once per Askaryan emission model and produce the spectra being compared) — https://github.com/nu-radio/NuRadioMC
- **NuRadioReco** (Provides the detector and trigger simulation within NuRadioMC and the units convention (NuRadioReco/utilities/units) used for all unit-bearing extracted quantities) — https://github.com/nu-radio/NuRadioMC
- **Python** (Modular Python framework language in which the simulation runs and the spectrum-extraction/aggregation script is written)

## Examples

```
pip install NuRadioMC; python -c "from NuRadioReco.utilities import units"
```

## Evaluation signals

- emission_model_comparison.csv contains exactly one spectrum entry per available Askaryan emission model/parameterization.
- Every unit-bearing column is expressed in the NuRadioReco/utilities/units convention.
- The neutrino interaction, signal propagation, and antenna geometry are identical across all per-model runs (only the emission model differs).
- Each spectrum is a frequency-domain electric-field spectrum at the fixed antenna, not a time-domain trace or voltage waveform.
- The NuRadioMC simulation chain covers the full chain: neutrino interaction, Askaryan emission, propagation, and detector response for each run.

## Limitations

- Comparison is only meaningful under a single fixed shower and detector configuration; varying geometry confounds the emission-model comparison.
- NuRadioMC is continuously improved and new features are added, so the set of available emission models/parameterizations depends on the installed version.
- The arXiv version of the citation notes improved ray-tracing equations and FFT normalization documentation relative to the published version, so FFT normalization conventions should be checked against the current NuRadioMC documentation.

## Evidence

- [abstract] Full-chain simulation capability: "simulates everything from the neutrino interaction in a medium, the subsequent Askaryan radio emission, the propagation of the radio signal to the"
- [abstract] Emission model comparison is the paper's stated purpose: "A variety of models and parameterizations for the radio emission of neutrino-induced showers are compared and reviewed."
- [readme] NuRadioMC is the Monte Carlo engine for the runs: "**NuRadioMC**: A Monte Carlo simulation package for radio neutrino detectors."
- [readme] NuRadioReco is used for detector/trigger simulation and shares the repo with NuRadioMC: "NuRadioReco is independent of NuRadioMC and used by NuRadioMC for the detector and trigger simulation."
