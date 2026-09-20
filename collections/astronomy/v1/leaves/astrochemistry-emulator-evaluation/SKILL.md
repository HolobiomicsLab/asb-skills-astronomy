---
name: astrochemistry-emulator-evaluation
description: Use when a trained neural surrogate for PDR chemistry (3D-PDR emulation)
  must be validated against reference observables — specifically reproducing observable
  column density maps from predicted chemical abundances and temperatures.
when_to_use_negative:
- Input is only raw Zenodo data (e.g. v3/3dpdr_dataset_v3.h5 in data/zenodo/) — header-processing
  workflow scripts must first create the processed HDF5 dataset
- No trained surrogate checkpoint exists yet — train via 'python src/neuralpdr/train.py
  configs/<version>/base.yaml' before evaluating
- Goal is validating emulators of a different chemistry code (e.g. UCLCHEM 1D uniform
  clouds v1/v2) without corresponding 3D-PDR reference column density maps
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: NeuralPDR
  role: Provides the inference script (src/neuralpdr/inference.py) used to run the
    trained Latent Augmented Neural ODE surrogate and generate predicted maps
  repo: https://github.com/uclchem/neuralpdr
- name: 3D-PDR
  role: Original PDR code whose observable column density maps serve as the reference
    for emulator validation
- name: HDF5
  role: File format of the processed dataset (3dpdr_dataset_v3.h5) consumed during
    inference
- name: Python
  role: Runtime for the inference, map generation, and metrics-table computation scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:f86348a89b949c652207b63823861ba5b74d43a4faa38f665362d35d15bcf296
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - NeuralPDR
  - Python
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# astrochemistry-emulator-evaluation

## Summary

Evaluates a trained astrochemistry surrogate emulator (e.g. Latent Augmented Neural ODE emulating 3D-PDR) by running inference on a processed HDF5 dataset and comparing the surrogate's predicted observable column density maps against the original PDR-code reference maps via mean relative error.

## When to use

Use when a trained neural surrogate for PDR chemistry (3D-PDR emulation) must be validated against reference observables — specifically reproducing observable column density maps from predicted chemical abundances and temperatures.

## When NOT to use

- Input is only raw Zenodo data (e.g. v3/3dpdr_dataset_v3.h5 in data/zenodo/) — header-processing workflow scripts must first create the processed HDF5 dataset
- No trained surrogate checkpoint exists yet — train via 'python src/neuralpdr/train.py configs/<version>/base.yaml' before evaluating
- Goal is validating emulators of a different chemistry code (e.g. UCLCHEM 1D uniform clouds v1/v2) without corresponding 3D-PDR reference column density maps

## Inputs

- Processed HDF5 dataset (data/processed/3dpdr_dataset_v3.h5) for the 3D giant molecular cloud model
- Trained surrogate checkpoint in the model runs directory (best_weights.pkl weights file)
- Original 3D-PDR reference observable column density maps from the v3 Zenodo deposit (doi:10.5281/zenodo.15688233)

## Outputs

- Surrogate-predicted observable column density maps generated from predicted chemical abundances and temperatures
- Map-comparison figures (predicted vs reference maps)
- Metrics table containing the mean_relative_error field

## How to apply

Load the processed v3 dataset (data/processed/3dpdr_dataset_v3.h5, HDF5 format) for the 3D giant molecular cloud model and the trained surrogate checkpoint (best_weights.pkl) from the model runs directory. Run inference with src/neuralpdr/inference.py, passing --dataset_path, --model_dir, and --weights_file. Generate the surrogate's observable column density maps from the predicted chemical abundances and temperatures, then compare them against the original 3D-PDR reference maps from the v3 Zenodo deposit (doi:10.5281/zenodo.15688233) and compute the mean relative error. Save map-comparison figures and a metrics table containing the mean_relative_error field. Judgment: the metrics table must exist, contain mean_relative_error, and the value must fall within the tolerance reported in the paper; visual agreement of the maps may additionally be assessed by expert review.

## Related tools

- **NeuralPDR** (Provides the inference script (src/neuralpdr/inference.py) used to run the trained Latent Augmented Neural ODE surrogate and generate predicted maps) — https://github.com/uclchem/neuralpdr
- **3D-PDR** (Original PDR code whose observable column density maps serve as the reference for emulator validation)
- **HDF5** (File format of the processed dataset (3dpdr_dataset_v3.h5) consumed during inference)
- **Python** (Runtime for the inference, map generation, and metrics-table computation scripts)

## Examples

```
python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v3.h5 --model_dir runs/model_name --weights_file best_weights.pkl
```

## Evaluation signals

- Metrics table file exists after inference
- Metrics table contains a mean_relative_error field
- mean_relative_error value falls within the tolerance reported in the paper
- Map-comparison figures are saved and predicted column density maps visually agree with the original 3D-PDR reference maps (assessable by expert review)
- Inference completes using the specified weights file (best_weights.pkl) without schema mismatches against the processed HDF5 dataset

## Limitations

- Validation is quantitative only via mean relative error within the paper-reported tolerance; visual agreement relies on subjective expert review
- Evaluation is specific to the trained v3 surrogate on the 3D giant molecular cloud dataset; v1 (uniform 1D clouds) and v2 (varying density clouds) datasets have different physical complexity
- The Neptune callback used during training runs will be deprecated as of March 2026 and will stop functioning
- The article's discussion section content is absent beyond a changelog notice, so no additional evaluation caveats could be traced

## Evidence

- [abstract] We show that these surrogate models can provide speedup and reproduce the original observable column density maps of the dataset.: "We show that these surrogate models can provide speedup and reproduce the original observable column density maps of the dataset."
- [methods] Run inference on a trained model by specifying the dataset, model directory, and weights file: "Run inference on a trained model by specifying the dataset, model directory, and weights file:"
- [readme] This creates processed HDF5 files in `data/processed/`: "This creates processed HDF5 files in `data/processed/`"
- [readme] The third dataset can be found at: https://doi.org/10.5281/zenodo.15688233: "The third dataset can be found at: https://doi.org/10.5281/zenodo.15688233"
