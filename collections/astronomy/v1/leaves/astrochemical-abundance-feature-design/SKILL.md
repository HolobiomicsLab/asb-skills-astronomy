---
name: astrochemical-abundance-feature-design
description: Use when building a neural-ODE surrogate for an astrochemical solver
  (e.g. 3D-PDR) and you must decide which physical quantities parameterise the ODE
  vector field versus which quantities are integrated as state features.
when_to_use_negative:
- The target is the raw 3D-PDR solver output rather than a learned surrogate — the
  feature/parameter split only applies to the neural ODE emulator, not to the physical
  code
- The dataset has not been processed into training-ready HDF5 format via the header
  scripts (v1_add_headers.py, workflow_v2.sh, workflow_v3.sh) — the abundance/temperature
  arrays and parameter fields will not yet be in inspectable shape
- The surrogate must handle time- or space-varying quantities that the chosen dataset
  version held constant (e.g. density and cosmic ray rate are constant in v1/v2);
  re-derive the conditioning split per dataset version
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Implementation language for the input-encoding module, smoke-test script,
    and training entry point (python src/neuralpdr/train.py configs/v2/base.yaml)
  repo: https://github.com/uclchem/neuralpdr
- name: HDF5
  role: File format of the processed datasets whose abundance, temperature and parameter
    arrays are inspected with h5py to determine shapes before defining the encoding
    split
- name: 3D-PDR
  role: Photodissociation-region chemistry code being emulated; its inputs (Av, G0,
    cosmic ray rate, density) and states (abundances, temperature) define the conditioning/feature
    split
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:a72d9f61e8376ec6126fc3396481fe3ae8be13b6d0aa9f101eaa900bfa9ef571
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - Python
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# astrochemical-abundance-feature-design

## Summary

Defines the input-encoding split for a NeuralPDR-style Latent Augmented Neural ODE surrogate of 3D-PDR chemistry: visual extinction (Av), radiation field (G0), cosmic ray ionisation rate and density are injected as conditioning parameters, while chemical abundances and temperature are evolved as ordinary features of the augmented neural ODE.

## When to use

Use when building a neural-ODE surrogate for an astrochemical solver (e.g. 3D-PDR) and you must decide which physical quantities parameterise the ODE vector field versus which quantities are integrated as state features.

## When NOT to use

- The target is the raw 3D-PDR solver output rather than a learned surrogate — the feature/parameter split only applies to the neural ODE emulator, not to the physical code
- The dataset has not been processed into training-ready HDF5 format via the header scripts (v1_add_headers.py, workflow_v2.sh, workflow_v3.sh) — the abundance/temperature arrays and parameter fields will not yet be in inspectable shape
- The surrogate must handle time- or space-varying quantities that the chosen dataset version held constant (e.g. density and cosmic ray rate are constant in v1/v2); re-derive the conditioning split per dataset version

## Inputs

- Processed HDF5 dataset (data/processed/3dpdr_dataset_v1.h5, _v2.h5 or _v3.h5) containing chemical abundances, temperatures, and cloud-level physical parameters
- Raw Zenodo datasets placed in data/zenodo/ (v1/3pdr_dataset_8192.h5, v2/simulations.tgz, v3/3dpdr_dataset_v3.h5) processed via header scripts
- Model configuration YAML from configs/v1/, configs/v2/, or configs/v3/

## Outputs

- Input-encoding module in the neuralpdr model code that injects Av, G0, cosmic ray ionisation rate and density as conditioning parameters and abundances/temperature as ODE features
- encoding_split.json listing conditioning parameters versus ordinary features per dataset version
- Smoke-test script confirming a forward pass on dummy input returns output shaped to the feature dimensionality

## How to apply

Load a processed HDF5 dataset (e.g. data/processed/3dpdr_dataset_v1.h5, v2, or v3) with h5py and inspect the array shapes of the chemical abundances, temperature, and the cloud-level physical parameters (Av, G0, cosmic ray ionisation rate, density). Implement the input-encoding module so that visual extinction and radiation field — and, depending on dataset version, the constant densities and constant cosmic ray ionisations — are passed as additional conditioning parameters, while chemical abundances and temperatures are passed as normal features of the Augmented Neural ODE. For v1 (uniform 1D clouds) and v2 (varying-density clouds), density and cosmic ray rate are constant per run; for the v3 3D giant molecular cloud dataset the per-cell parameter values come from the simulation. Record the chosen split in a structured file (encoding_split.json) listing the four conditioning parameters and the two feature types per dataset version. Validate with a smoke test that feeds dataset-shaped dummy input through a forward pass of the model built from a configs/ YAML file and confirms the output shape matches the feature dimensionality.

## Related tools

- **Python** (Implementation language for the input-encoding module, smoke-test script, and training entry point (python src/neuralpdr/train.py configs/v2/base.yaml)) — https://github.com/uclchem/neuralpdr
- **HDF5** (File format of the processed datasets whose abundance, temperature and parameter arrays are inspected with h5py to determine shapes before defining the encoding split)
- **3D-PDR** (Photodissociation-region chemistry code being emulated; its inputs (Av, G0, cosmic ray rate, density) and states (abundances, temperature) define the conditioning/feature split)

## Examples

```
python src/neuralpdr/train.py configs/v2/base.yaml
```

## Evaluation signals

- Smoke-test forward pass on dataset-shaped dummy input completes without error and output shape matches the feature dimensionality (abundances + temperature)
- encoding_split.json contains fields for all four conditioning parameters (Av, G0, cosmic ray ionisation rate, density) and both feature types (chemical abundances, temperature)
- h5py inspection of the processed HDF5 files confirms abundance/temperature array shapes are consistent with the feature dimensionality declared in the model
- The split matches the published scheme: 'densities (constant), cosmic ray ionisations (constant), visual extinctions, radiation field as additional parameters and the chemical abundances and temperatures as normal features'
- Training with a configs/ YAML (e.g. configs/v2/base.yaml) runs to completion with checkpoints and logs saved to the configured directory

## Limitations

- In dataset versions v1 and v2, densities and cosmic ray ionisations are constant, so treating them as conditioning parameters assumes per-run constancy that does not generalise to fully varying conditions
- The Neptune logging callback is deprecated as of March 2026 and its call will stop functioning, so experiment tracking tied to this workflow must be migrated
- The scheme was validated on three specific datasets (uniform 1D clouds, varying-density clouds, 3D giant molecular cloud); applying it to other codes or parameterisations requires re-deriving the split

## Evidence

- [intro] Conditioning parameters vs features split: "Taking densities (constant), cosmic ray ionisations (constant), visual extinctions, radiation field as additional parameters and the chemical abundances and temperatures as normal features."
- [methods] Processed HDF5 data location: "This creates processed HDF5 files in `data/processed/`"
- [methods] Training entry point: "python src/neuralpdr/train.py configs/v2/base.yaml"
- [readme] Dataset versions and emulated code: "In this project we emulate the Photodissociation Region (PDR) code 3D-PDR for 1D-dimensional uniform clouds (v1), clouds of varying density (v2) and a 3D model of a giant molecular cloud (v3)"
- [readme] Dataset structure for preparing inputs: "Download the datasets from Zenodo (links below) and place them in `data/zenodo/`"
