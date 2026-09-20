---
name: neural-ode-conditioning-parameter-encoding
description: Use when building a neural ODE surrogate for PDR chemistry and you must
  decide which quantities (e.g. Av, G0, cosmic-ray rate, density vs abundances, temperature)
  are injected as conditioning parameters versus evolved as ODE features.
when_to_use_negative:
- When the emulator input is already fully encoded (e.g. a trained checkpoint being
  loaded for inference with src/neuralpdr/inference.py) — re-encoding is unnecessary
- When the target model is not an augmented/latent neural ODE surrogate for 3D-PDR
  chemistry (e.g. a purely feed-forward emulator with a flat input vector)
- When the dataset is the raw Zenodo download (v1/3pdr_dataset_8192.h5, v2/simulations.tgz)
  rather than a processed HDF5 file in data/processed/
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Language for the input-encoding module in src/neuralpdr and the smoke-test
    script
  repo: https://github.com/uclchem/neuralpdr
- name: HDF5
  role: File format of the processed datasets (data/processed/3dpdr_dataset_v*.h5)
    whose arrays define the conditioning/feature split
- name: 3D-PDR
  role: Emulated PDR code whose chemistry sets which quantities are parameters (Av,
    G0, cosmic rays, density) and which are evolved features
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:fd9effb13e8b01b55887703bec074083bd261c748a2ff48f12dbf5b4c580e2f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - Python
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# neural-ode-conditioning-parameter-encoding

## Summary

Encodes physical/chemical inputs for a NeuralPDR surrogate by splitting visual extinction, radiation field, cosmic ray ionisation rate and density as additional conditioning parameters of the Latent Augmented Neural ODE, while chemical abundances and temperature are passed as ordinary features. This split mirrors how 3D-PDR solves cooling, heating and chemistry along lines of sight.

## When to use

Use when building a neural ODE surrogate for PDR chemistry and you must decide which quantities (e.g. Av, G0, cosmic-ray rate, density vs abundances, temperature) are injected as conditioning parameters versus evolved as ODE features.

## When NOT to use

- When the emulator input is already fully encoded (e.g. a trained checkpoint being loaded for inference with src/neuralpdr/inference.py) — re-encoding is unnecessary
- When the target model is not an augmented/latent neural ODE surrogate for 3D-PDR chemistry (e.g. a purely feed-forward emulator with a flat input vector)
- When the dataset is the raw Zenodo download (v1/3pdr_dataset_8192.h5, v2/simulations.tgz) rather than a processed HDF5 file in data/processed/

## Inputs

- Processed HDF5 dataset (e.g. data/processed/3dpdr_dataset_v1.h5, 3dpdr_dataset_v2.h5, 3dpdr_dataset_v3.h5)
- Dataset arrays of chemical abundances and temperature (features)
- Cloud-level physical parameters: visual extinction (Av), radiation field (G0), cosmic ray ionisation rate, density

## Outputs

- Input-encoding module in the neuralpdr model that injects Av, G0, cosmic ray ionisation and density as conditioning parameters
- Structured encoding_split.json listing conditioning parameters vs ordinary features per dataset version
- Smoke-test script confirming a forward pass on dummy input returns output shaped to the feature dimensionality

## How to apply

Load a processed HDF5 dataset (e.g. data/processed/3dpdr_dataset_v2.h5) with h5py and inspect the array shapes of abundances, temperature and cloud-level physical parameters. Implement the input-encoding module in the neuralpdr model code so that visual extinction, radiation field, cosmic ray ionisation rate and density are injected as additional conditioning parameters of the Augmented Neural ODE, while chemical abundances and temperatures are passed as normal features. Write a smoke test that instantiates the model, feeds dataset-shaped dummy input through a forward pass, and checks the output shape matches the feature dimensionality. Record the split in a structured file (encoding_split.json) listing the four conditioning parameters and both feature types per dataset version (v1 uniform 1D clouds, v2 varying-density clouds, v3 3D giant molecular cloud).

## Related tools

- **Python** (Language for the input-encoding module in src/neuralpdr and the smoke-test script) — https://github.com/uclchem/neuralpdr
- **HDF5** (File format of the processed datasets (data/processed/3dpdr_dataset_v*.h5) whose arrays define the conditioning/feature split)
- **3D-PDR** (Emulated PDR code whose chemistry sets which quantities are parameters (Av, G0, cosmic rays, density) and which are evolved features)

## Examples

```
python src/neuralpdr/train.py configs/v2/base.yaml
```

## Evaluation signals

- Smoke-test forward pass on dataset-shaped dummy input runs without error and output shape matches the feature dimensionality (abundances + temperature)
- encoding_split.json contains fields for all four conditioning parameters (Av, G0, cosmic ray ionisation rate, density) and both feature types (abundances, temperature)
- h5py inspection of the processed HDF5 confirms separate array blocks for the four cloud-level physical parameters and the chemical features
- Model trains from a configs/v*/base.yaml configuration without shape mismatches between parameter injection and ODE feature dimension

## Limitations

- The split is defined for the NeuralPDR emulator of 3D-PDR (v1 uniform 1D clouds, v2 varying-density clouds, v3 3D giant molecular cloud); other chemical codes may require a different parameter/feature split
- Densities and cosmic ray ionisations are treated as constant in the README's formulation, so per-timestep variation of these quantities is outside this encoding scheme
- The Neptune callback is deprecated as of March 2026 and should not be relied on when wiring training around the encoding module

## Evidence

- [intro] Parameter vs feature split: "Taking densities (constant), cosmic ray ionisations (constant), visual extinctions, radiation field as additional parameters and the chemical abundances and temperatures as normal features."
- [readme] Dataset format for inspection: "This creates processed HDF5 files in `data/processed/`"
- [readme] Physical rationale: "3D-PDR code solves the cooling, heating and chemistry as we move along lines of sights into a cloud (changing Av)."
