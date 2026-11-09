---
name: hdf5-dataset-loading
description: Use when a downstream task needs the processed 3D-PDR chemistry data
  (chemical abundances, temperatures, densities, cosmic ray ionisation rates, Av,
  radiation field) as HDF5 input for LANODE surrogate training, inference, or benchmarking.
when_to_use_negative:
- The goal is training or inference on the raw Zenodo files directly — they must first
  be converted into processed HDF5 files in data/processed/ via the header-processing
  scripts
- The task targets a chemistry code or emulator other than 3D-PDR / NeuralPDR — the
  HDF5 layout and neuralpdr HDF5 interface are specific to these v1/v2/v3 datasets
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Python
  role: Runs the neuralpdr data-processing scripts and the train/inference entry points
    that load the HDF5 datasets
  repo: https://github.com/uclchem/neuralpdr
- name: HDF5
  role: File format of the processed dataset files (data/processed/3dpdr_dataset_v1/v2/v3.h5)
    accessed through the neuralpdr HDF5 interface and --dataset_path
- name: 3D-PDR
  role: Original photodissociation-region chemistry code whose outputs the v1–v3 datasets
    encode (abundances, temperatures, column density maps)
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:a7b2e701b4bbefc798a119f52d8cd2b7e8d815b88bd2329d79c3ad17b83e2d0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hdf5-dataset-loading

## Summary

Load the NeuralPDR 3D-PDR chemistry datasets (v1 uniform 1D clouds, v2 varying-density clouds, v3 3D giant molecular cloud) from processed HDF5 files via the neuralpdr package's HDF5 interface, so that surrogate-model training or inference can consume them.

## When to use

Use when a downstream task needs the processed 3D-PDR chemistry data (chemical abundances, temperatures, densities, cosmic ray ionisation rates, Av, radiation field) as HDF5 input for LANODE surrogate training, inference, or benchmarking.

## When NOT to use

- The goal is training or inference on the raw Zenodo files directly — they must first be converted into processed HDF5 files in data/processed/ via the header-processing scripts
- The task targets a chemistry code or emulator other than 3D-PDR / NeuralPDR — the HDF5 layout and neuralpdr HDF5 interface are specific to these v1/v2/v3 datasets

## Inputs

- Raw Zenodo dataset v1: data/zenodo/v1/3pdr_dataset_8192.h5
- Raw Zenodo dataset v2: data/zenodo/v2/simulations.tgz
- Raw Zenodo dataset v3: data/zenodo/v3/3dpdr_dataset_v3.h5
- Processed HDF5 files: data/processed/3dpdr_dataset_v1.h5, 3dpdr_dataset_v2.h5, 3dpdr_dataset_v3.h5
- Quick-validation test datasets in data/test/

## Outputs

- In-memory dataset object (densities, cosmic ray ionisation rates, visual extinction, radiation field as parameters; chemical abundances and temperatures as features) consumed by train.py or inference.py
- Predicted chemical abundances and temperatures from surrogate inference when used downstream

## How to apply

First obtain the raw datasets from the three Zenodo deposits and place them in data/zenodo/ following the expected structure (v1/3pdr_dataset_8192.h5, v2/simulations.tgz, v3/3dpdr_dataset_v3.h5). Run the header-processing scripts (workflow_setup_all.sh, or v1_add_headers.py / workflow_v2.sh / workflow_v3.sh individually) to create training-ready processed HDF5 files in data/processed/ (3dpdr_dataset_v1.h5, 3dpdr_dataset_v2.h5, 3dpdr_dataset_v3.h5). Then load the processed file via the HDF5 interface in the neuralpdr package and pass its path as --dataset_path to src/neuralpdr/inference.py together with --model_dir and --weights_file. For quick validation without downloading full Zenodo data, use the small test datasets in data/test/. Each dataset version corresponds to a different cloud model, so select the file matching the intended physical complexity (v1 uniform 1D, v2 varying density, v3 3D giant molecular cloud).

## Related tools

- **Python** (Runs the neuralpdr data-processing scripts and the train/inference entry points that load the HDF5 datasets) — https://github.com/uclchem/neuralpdr
- **HDF5** (File format of the processed dataset files (data/processed/3dpdr_dataset_v1/v2/v3.h5) accessed through the neuralpdr HDF5 interface and --dataset_path)
- **3D-PDR** (Original photodissociation-region chemistry code whose outputs the v1–v3 datasets encode (abundances, temperatures, column density maps))

## Examples

```
python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v2.h5 --model_dir runs/model_name --weights_file best_weights.pkl
```

## Evaluation signals

- The expected processed files exist in data/processed/: 3dpdr_dataset_v1.h5, 3dpdr_dataset_v2.h5, and 3dpdr_dataset_v3.h5 after running the workflow scripts
- The dataset version selected (v1/v2/v3) matches the intended cloud model — v1 uniform 1D clouds, v2 varying density clouds, v3 3D giant molecular cloud
- inference.py accepts the --dataset_path argument without a file-format or missing-file error and the run completes
- For quick validation, the test datasets in data/test/ load successfully without downloading the full Zenodo deposits

## Limitations

- The Neptune logging callback will be deprecated as of March 2026 and its call will stop functioning; avoid depending on it during dataset/model runs
- Full datasets must be downloaded from three separate Zenodo deposits (10.5281/zenodo.13711173, 10.5281/zenodo.7310832, 10.5281/zenodo.15688233) and placed in the exact data/zenodo/ directory structure before processing; raw files cannot be used directly
- v2 ships as an archive (simulations.tgz) and requires the workflow_v2.sh script rather than a simple header-adding step like v1

## Evidence

- [readme] Processed datasets and file placement: "Download the datasets from Zenodo (links below) and place them in `data/zenodo/` following this structure"
- [readme] Processed output files: "This creates processed HDF5 files in `data/processed/`"
- [readme] Dataset-version semantics: "The datasets correspond to different cloud models: v1 (uniform 1D clouds), v2 (varying density clouds), v3 (3D giant molecular cloud)."
- [readme] Test datasets for quick validation: "Alternatively, use the test datasets in `data/test/` for quick validation without downloading the full Zenodo datasets."
- [methods] Inference invocation with dataset path: "python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v2.h5 --model_dir runs/model_name --weights_file best_weights.pkl"
- [intro] Model parameters and features encoded in the datasets: "Taking densities (constant), cosmic ray ionisations (constant), visual extinctions, radiation field as additional parameters and the chemical abundances and temperatures as normal features."
