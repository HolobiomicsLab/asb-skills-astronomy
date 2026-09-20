---
name: askaryan-trace-to-detector-signal-conversion
description: Use when input is radio field traces propagated to detector positions
  by NuRadioMC and the goal is detector-level voltage traces and per-station trigger
  decisions.
when_to_use_negative:
- Input is a raw neutrino interaction or Askaryan emission model rather than already-propagated
  field traces — the upstream NuRadioMC stages handle that
- Goal is reconstructing physical parameters from measured data rather than simulating
  the detector response of simulated fields
edam_operation: http://edamontology.org/operation_0300
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: NuRadioReco
  role: Applies the detector station description to field traces and runs the trigger
    simulation modules to produce detector-level signals and trigger decisions
  repo: https://github.com/nu-radio/NuRadioMC
- name: NuRadioMC
  role: Upstream Monte Carlo framework whose propagated radio signals are fed into
    the detector and trigger simulation
  repo: https://github.com/nu-radio/NuRadioMC
- name: Python
  role: Language in which the modular NuRadioMC/NuRadioReco simulation chain is implemented
    and invoked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-020-7612-8
    title: 'NuRadioMC: simulating the radio emission of neutrinos from interaction
      to detector'
  source_content_hash: sha256:a62853bcdb09ba971d189425772703c4010918ce3561c55a52545cddbbca142f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nu-radio/NuRadioMC
  tools:
  - NuRadioMC
  - NuRadioReco
  - Python
  edam_operation: http://edamontology.org/operation_0300
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# askaryan-trace-to-detector-signal-conversion

## Summary

Converts propagated Askaryan radio field traces from the NuRadioMC simulation chain into detector-level station signals by applying the detector description in NuRadioReco, the stage NuRadioMC delegates to for detector and trigger simulation.

## When to use

Use when input is radio field traces propagated to detector positions by NuRadioMC and the goal is detector-level voltage traces and per-station trigger decisions.

## When NOT to use

- Input is a raw neutrino interaction or Askaryan emission model rather than already-propagated field traces — the upstream NuRadioMC stages handle that
- Goal is reconstructing physical parameters from measured data rather than simulating the detector response of simulated fields

## Inputs

- Propagated radio field traces (Askaryan emission propagated to detector positions) from the NuRadioMC signal-propagation stage
- Detector station description file defining antenna positions and hardware chain

## Outputs

- Detector-level station signals (voltage traces per station)
- Per-event trigger decisions, e.g. written to trigger_decisions.csv with event identifier, station identifier, and triggered flag

## How to apply

Load the propagated radio field traces with NuRadioReco's event-reading utilities, then apply the detector station description to obtain detector-level signals. Express every quantity carrying a physical unit through the NuRadioReco/utilities/units convention, since unit mismatches silently corrupt thresholds. Feed the detector-level signals of each station through NuRadioReco's trigger simulation modules, which apply the trigger threshold per event, and collect per-event trigger decisions. NuRadioReco is used for this stage because it is the framework NuRadioMC delegates detector and trigger simulation to, even though the two codes are independent.

## Related tools

- **NuRadioReco** (Applies the detector station description to field traces and runs the trigger simulation modules to produce detector-level signals and trigger decisions) — https://github.com/nu-radio/NuRadioMC
- **NuRadioMC** (Upstream Monte Carlo framework whose propagated radio signals are fed into the detector and trigger simulation) — https://github.com/nu-radio/NuRadioMC
- **Python** (Language in which the modular NuRadioMC/NuRadioReco simulation chain is implemented and invoked)

## Examples

```
from NuRadioReco.utilities import units  # convert all field/voltage quantities to NuRadioReco unit convention before trigger evaluation
```

## Evaluation signals

- trigger_decisions.csv exists and contains a decision record for each simulated event
- Each record includes the required fields: event identifier, station identifier, and triggered flag
- All physical quantities in the conversion pass through the NuRadioReco/utilities/units convention (no raw SI/unitless mixing)
- Per-station trigger thresholds are applied to detector-level signals for every event, not a subset

## Limitations

- NuRadioReco is independent of NuRadioMC; keeping both developed together is a convenience of the shared GitHub repository, so version mismatches can affect the detector/trigger simulation interface
- The detector simulation depends on a correct detector station description; an inaccurate description yields wrong detector-level signals even if the field traces are correct

## Evidence

- [intro] NuRadioReco delegates detector/trigger stage: "NuRadioReco is independent of NuRadioMC
and used by NuRadioMC for the detector and trigger simulation."
- [intro] NuRadioReco role definition: "**NuRadioReco**: A reconstruction and detector simulation framework for radio detectors of high-energy neutrinos and cosmic-rays."
- [abstract] Full simulation chain context: "simulates everything from the neutrino interaction in a medium, the subsequent Askaryan radio emission, the propagation of the radio signal to the detector and finally the detector response"
- [readme] Shared repository for both codes: "To simplify the development and continuous integration testing, both codes are developed
in the same github repository."
- [readme] Installable framework: [evidence span withheld — anchor status: external_doc]
