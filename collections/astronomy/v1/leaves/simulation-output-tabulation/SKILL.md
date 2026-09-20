---
name: simulation-output-tabulation
description: Use when the goal is to compare multiple radio-emission models/parameterizations
  for neutrino-induced showers in NuRadioMC while holding the shower, propagation,
  and antenna configuration fixed, and the deliverable is one spectrum entry per emission
  model in a unit-consistent table.
when_to_use_negative:
- The task is detector/trigger simulation or event reconstruction rather than comparing
  emission models — that is NuRadioReco's role, which is independent of NuRadioMC
  and used by NuRadioMC for detector and trigger simulation.
- The neutrino interaction, propagation, or antenna geometry must vary between runs
  — the comparison is only valid under a fixed shower and detector configuration.
- The analysis does not require a shared units convention across models; mixing unit
  conventions would invalidate the tabulated comparison.
tools:
- name: NuRadioMC
  role: Monte Carlo simulation package used to run the full chain (neutrino interaction,
    Askaryan emission, propagation, detector response) once per emission model under
    the fixed configuration
  repo: github.com/nu-radio/NuRadioMC
- name: NuRadioReco
  role: Provides the units convention (NuRadioReco/utilities/units) for expressing
    unit-bearing spectrum variables, and performs the detector/trigger simulation
    within NuRadioMC
  repo: github.com/nu-radio/NuRadioMC
- name: Python
  role: Language of the NuRadioMC framework; used to execute the simulation chain
    per model, extract spectra, and assemble the CSV table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-020-7612-8
    title: 'NuRadioMC: simulating the radio emission of neutrinos from interaction
      to detector'
  source_content_hash: sha256:a4c70e1b2516b339cfd224bad2cf795c5c82b317c20fbf8ad54be52e358486be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NuRadioMC
  - NuRadioReco
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# simulation-output-tabulation

## Summary

Runs the NuRadioMC simulation chain once per available Askaryan emission model/parameterization under a fixed shower-and-detector configuration, then tabulates the per-model frequency-domain electric-field spectra at the antenna into a single comparison table (emission_model_comparison.csv). It operationalizes the paper's goal of comparing emission parameterizations on identical neutrino interaction, propagation, and antenna geometry.

## When to use

Use when the goal is to compare multiple radio-emission models/parameterizations for neutrino-induced showers in NuRadioMC while holding the shower, propagation, and antenna configuration fixed, and the deliverable is one spectrum entry per emission model in a unit-consistent table.

## When NOT to use

- The task is detector/trigger simulation or event reconstruction rather than comparing emission models — that is NuRadioReco's role, which is independent of NuRadioMC and used by NuRadioMC for detector and trigger simulation.
- The neutrino interaction, propagation, or antenna geometry must vary between runs — the comparison is only valid under a fixed shower and detector configuration.
- The analysis does not require a shared units convention across models; mixing unit conventions would invalidate the tabulated comparison.

## Inputs

- NuRadioMC example simulation configuration file defining a fixed in-ice neutrino-induced shower and fixed antenna position/viewing angle
- Set of available Askaryan emission models/parameterizations implemented in NuRadioMC
- Fixed signal-propagation and antenna geometry settings held constant across runs

## Outputs

- emission_model_comparison.csv — a table with one frequency-domain electric-field spectrum entry per emission parameterization
- Per-model NuRadioMC simulation outputs (frequency-domain spectra at the antenna)

## How to apply

Load the NuRadioMC example simulation configuration that defines a fixed in-ice neutrino-induced shower and a fixed antenna position/viewing angle; this leverages NuRadioMC's full-chain design, which simulates the neutrino interaction, the subsequent Askaryan radio emission, signal propagation, and detector response. Execute the NuRadioMC simulation chain once per available Askaryan emission model/parameterization, keeping the neutrino interaction, signal propagation, and antenna geometry identical across runs so that differences in output are attributable solely to the emission parameterization. From each model's output, extract the frequency-domain electric-field spectrum at the antenna, expressing all unit-bearing variables with the NuRadioReco/utilities/units convention. Assemble the per-model spectra into a single table with one spectrum entry per emission parameterization and save it as emission_model_comparison.csv. Validate by confirming exactly one spectrum entry per available emission model and that every unit-bearing column uses the NuRadioReco units convention.

## Related tools

- **NuRadioMC** (Monte Carlo simulation package used to run the full chain (neutrino interaction, Askaryan emission, propagation, detector response) once per emission model under the fixed configuration) — github.com/nu-radio/NuRadioMC
- **NuRadioReco** (Provides the units convention (NuRadioReco/utilities/units) for expressing unit-bearing spectrum variables, and performs the detector/trigger simulation within NuRadioMC) — github.com/nu-radio/NuRadioMC
- **Python** (Language of the NuRadioMC framework; used to execute the simulation chain per model, extract spectra, and assemble the CSV table)

## Examples

```
from NuRadioReco.utilities import units; import NuRadioMC  # run NuRadioMC per emission model, extract antenna spectra, write emission_model_comparison.csv
```

## Evaluation signals

- The output table contains exactly one spectrum entry per available emission model/parameterization.
- Every unit-bearing column is expressed in the NuRadioReco/utilities/units convention.
- All runs used identical neutrino interaction, signal-propagation, and antenna-geometry settings, so spectral differences trace to the emission model alone.
- Each spectrum entry is a frequency-domain electric-field spectrum at the (fixed) antenna position/viewing angle.

## Limitations

- The comparison is meaningful only under a fixed shower and detector configuration; varying the configuration confounds the model comparison.
- NuRadioReco is a separate framework used by NuRadioMC for detector/trigger simulation — both codes are developed in the same GitHub repository, and the current-version FFT normalization and improved ray-tracing equations are documented in the arXiv version of the paper rather than the published one.
- The article provides no discussion-section content or changelog; consult the online documentation for up-to-date feature coverage.

## Evidence

- [abstract] Comparing emission models on a fixed configuration: "A variety of models and parameterizations for the radio emission of neutrino-induced showers are compared and reviewed."
- [abstract] Full simulation chain covered by NuRadioMC: "simulates everything from the neutrino interaction in a medium, the subsequent Askaryan radio emission, the propagation of the radio signal to the detector and finally the detector response"
- [intro] NuRadioReco's role for detector and trigger simulation: "NuRadioReco is independent of NuRadioMC
and used by NuRadioMC for the detector and trigger simulation."
- [readme] Installation of the tool needed to run the per-model simulations: [evidence span withheld — anchor status: external_doc]
