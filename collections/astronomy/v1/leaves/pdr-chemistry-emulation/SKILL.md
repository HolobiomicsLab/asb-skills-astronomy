---
name: pdr-chemistry-emulation
description: Use when the goal is to emulate or speed up 3D-PDR chemistry — e.
when_to_use_negative:
- The task is to solve the full 3D-PDR code itself (cooling, heating and chemistry
  along lines of sight) rather than emulate it — the surrogate replaces, not reimplements,
  the solver
- The dataset version's physical setup does not match the trained model (e.g. applying
  a v1 uniform-cloud surrogate to v3 3D giant molecular cloud conditions)
- Only a quick pipeline check is needed and the full Zenodo download is unnecessary
  — use data/test/ datasets instead of invoking the full training workflow
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2258
tools:
- name: Python
  role: Runs the NeuralPDR training, data-processing and inference scripts (train.py,
    v1_add_headers.py, inference.py)
  repo: https://github.com/uclchem/neuralpdr
- name: HDF5
  role: Storage format for both raw Zenodo datasets and processed training-ready files
    under data/processed/
- name: 3D-PDR
  role: The PDR chemistry code whose cooling, heating and chemistry the LANODE surrogate
    emulates
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:a4219b9cbe5fe4dbe741026ce65ec96f80b17e1c9a5c3834429815bbd5b7a233
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pdr-chemistry-emulation

## Summary

Train Augmented Neural Ordinary Differential Equations (Latent ANODEs) as surrogate models/emulators of the 3D-PDR photodissociation-region chemistry, parameter-conditioned on density, cosmic ray ionisation rate, visual extinction (Av) and radiation field (G0), with chemical abundances and temperature as ODE features.

## When to use

Use when the goal is to emulate or speed up 3D-PDR chemistry — e.g. for GPU-based rapid inference, statistical inference of observations, or coupling chemistry into higher-resolution hydrodynamical simulations — from uniform 1D clouds (v1), varying-density clouds (v2) or a 3D giant molecular cloud (v3).

## When NOT to use

- The task is to solve the full 3D-PDR code itself (cooling, heating and chemistry along lines of sight) rather than emulate it — the surrogate replaces, not reimplements, the solver
- The dataset version's physical setup does not match the trained model (e.g. applying a v1 uniform-cloud surrogate to v3 3D giant molecular cloud conditions)
- Only a quick pipeline check is needed and the full Zenodo download is unnecessary — use data/test/ datasets instead of invoking the full training workflow

## Inputs

- Raw Zenodo HDF5/tarball datasets: v1/3pdr_dataset_8192.h5, v2/simulations.tgz, v3/3dpdr_dataset_v3.h5
- Processed training-ready HDF5 files: data/processed/3dpdr_dataset_v1.h5, 3dpdr_dataset_v2.h5, 3dpdr_dataset_v3.h5
- Configuration YAML from configs/v1/, configs/v2/, or configs/v3/ specifying output directory
- Trained model weights file best_weights.pkl (for inference)

## Outputs

- Trained LANODE surrogate model checkpoints and logs in the configured run directory (best_weights.pkl)
- Emulated chemical abundances and temperatures reproducing the dataset's observable column density maps
- GPU-based rapid chemistry inference for statistical inference or hydrodynamical simulation coupling

## How to apply

Download the Zenodo dataset for the chosen cloud model (v1: DOI 10.5281/zenodo.13711173, 3pdr_dataset_8192.h5; v2: 10.5281/zenodo.7310832, simulations.tgz; v3: 10.5281/zenodo.15688233, 3dpdr_dataset_v3.h5) and place it under data/zenodo/ in the prescribed directory structure. Run the header processing script for the version (python scripts/data/v1_add_headers.py, ./scripts/data/workflow_v2.sh, ./scripts/data/workflow_v3.sh, or ./scripts/data/workflow_setup_all.sh for all) to create training-ready HDF5 files in data/processed/ (3dpdr_dataset_v1.h5, _v2.h5, _v3.h5). Encode inputs according to the NeuralPDR scheme: densities (constant), cosmic ray ionisations (constant), visual extinctions and radiation field are injected as additional conditioning parameters, while chemical abundances and temperatures are passed as normal features of the augmented neural ODE. Select a configuration YAML from configs/v1/, configs/v2/ or configs/v3/, set the output directory inside the config, and launch training with python src/neuralpdr/train.py configs/<version>/<config>.yaml; checkpoints and logs are written to the configured run directory. For inference, specify the dataset path, model directory and weights file (best_weights.pkl) to reproduce the original observable column density maps. data/test/ holds small test datasets for quick validation without downloading the full Zenodo datasets.

## Related tools

- **Python** (Runs the NeuralPDR training, data-processing and inference scripts (train.py, v1_add_headers.py, inference.py)) — https://github.com/uclchem/neuralpdr
- **HDF5** (Storage format for both raw Zenodo datasets and processed training-ready files under data/processed/)
- **3D-PDR** (The PDR chemistry code whose cooling, heating and chemistry the LANODE surrogate emulates)

## Examples

```
python src/neuralpdr/train.py configs/v2/base.yaml
```

## Evaluation signals

- Processed outputs exist: data/processed/ contains 3dpdr_dataset_v1.h5 (and v2/v3 equivalents) after running the header processing scripts
- Training run directory contains a loadable best_weights.pkl checkpoint and the training script executed to completion without error
- Inference on a trained model reproduces the original observable column density maps of the dataset
- Forward pass output shape matches the feature dimensionality (chemical abundances + temperature) for dataset-shaped input
- Encoding split is correct: four conditioning parameters (density, cosmic ray ionisation, Av, G0) vs two feature types (abundances, temperature)

## Limitations

- Surrogates emulate 3D-PDR for specific dataset regimes (v1 uniform 1D clouds, v2 varying density, v3 3D giant molecular cloud); generalisation outside the trained dataset's physical complexity is not established
- Neptune callback is deprecated as of March 2026 and will stop functioning / be removed from the codebase
- Substitution in coupled hydrodynamics-chemistry simulations is an opportunity enabled by the speedup, but reproduces column density maps only on the trained datasets

## Evidence

- [intro] emulate the 3D-PDR code for 1D uniform clouds: "We present here Augmented Neural Ordinary Differential Equations that act as surrogate models / emulators for this chemistry."
- [intro] input encoding scheme: "Taking densities (constant), cosmic ray ionisations (constant), visual extinctions, radiation field as additional parameters and the chemical abundances and temperatures as normal features."
- [readme] processing scripts and outputs: "Process the raw datasets into training-ready format by running the header processing scripts"
- [readme] inference command: "python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v2.h5 --model_dir runs/model_name --weights_file best_weights.pkl"
- [abstract] surrogate benefit: "This enables the rapid inference of the chemistry (on the GPU), allowing for the faster statistical inference of observations or increasing the resolution in hydrodynamical simulations"
