---
name: self-supervised-byol-training
description: Use when you have a large collection of unlabelled astronomical images
  (e.g. ALMA protoplanetary disk or Spitzer/Herschel infrared dark cloud FITS files)
  and need morphology-aware embeddings for clustering, anomaly detection, or similarity-based
  exploration.
when_to_use_negative:
- You already have manually labelled data and a supervised classification objective
  — the BYOL stage is explicitly for generating embeddings 'without manually labelled
  data'.
- Your inputs are not images (e.g. already-extracted feature tables or catalogues)
  — this skill operates on FITS imagery through a 2D convolutional backbone.
- You only need inference/visualisation from an existing trained network — use pipeline_02_inference.py
  and TensorBoard instead of retraining.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0769
tools:
- name: astromorph
  role: Provides the BYOL class, ByolTrainer wrapper, FitsFilelistDataset, AstroMorphologyModel
    backbone, and the pipeline_01_training.py training script driven by a TOML config
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework in which the BYOL class is implemented (subclass of pytorch.nn.Module);
    supplies DataLoader-based training and optimization loop
- name: pip
  role: Installs the package dependencies from requirements.txt inside the Python
    3.12 virtual environment
- name: venv
  role: Creates the isolated Python 3.12 virtual environment (python -m venv .venv)
    in which training is run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:bb4865efb494c9404f31a8da4366f31f255ff54a98e4138a4e7bb7b743cc15cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - astromorph
  - pip
  - venv
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# self-supervised-byol-training

## Summary

Self-supervised BYOL pretraining of a convolutional network (e.g. ResNet18 or the light-weight AstroMorphologyModel) on collections of FITS astronomical images using astromorph's BYOL class or ByolTrainer, producing label-free embeddings without manually annotated data.

## When to use

Use when you have a large collection of unlabelled astronomical images (e.g. ALMA protoplanetary disk or Spitzer/Herschel infrared dark cloud FITS files) and need morphology-aware embeddings for clustering, anomaly detection, or similarity-based exploration.

## When NOT to use

- You already have manually labelled data and a supervised classification objective — the BYOL stage is explicitly for generating embeddings 'without manually labelled data'.
- Your inputs are not images (e.g. already-extracted feature tables or catalogues) — this skill operates on FITS imagery through a 2D convolutional backbone.
- You only need inference/visualisation from an existing trained network — use pipeline_02_inference.py and TensorBoard instead of retraining.

## Inputs

- FITS image files referenced by a filelist (data/inputfiles.txt, generated via find with -size -10M -name "**.fits")
- TOML configuration file (e.g. training_settings.toml or example_settings.toml) specifying datafile, network_name, epochs, and optional network_settings/data_settings/byol_settings

## Outputs

- Trained network checkpoint saved to saved_models/ (e.g. saved_models/newly_trained_network.pt)
- Self-supervised image embeddings (consumed downstream by pipeline_02_inference.py)

## How to apply

Build a filelist of FITS inputs with the find command (e.g. files under 10 MB matching the .fits pattern) into data/inputfiles.txt, then write a TOML config with datafile, network_name = 'n_layer_resnet', and optionally epochs (default 10). Optionally truncate ResNet18 by setting [network_settings] last_layer (e.g. 'layer2'), since earlier convolutional layers are usually more generic; other knobs include core_limit, [data_settings] stacksize (e.g. 3 for single-channel images into a 3-channel network), and [byol_settings] representation_size = 128, projection_size = 16, projection_hidden_size = 512. Run the training script from the repository root: python astromorph/src/pipeline_01_training.py -c <config>.toml, which wraps the BYOL module in ByolTrainer over the FitsFilelistDataset with the AstroMorphologyModel backbone. If you use the BYOL class directly instead of ByolTrainer, you must call update_moving_average() after every optimization step so the target network remains the exponential moving average of the online network. Trained checkpoints are saved under saved_models/ (e.g. newly_trained_network.pt) for later use by the inference script.

## Related tools

- **astromorph** (Provides the BYOL class, ByolTrainer wrapper, FitsFilelistDataset, AstroMorphologyModel backbone, and the pipeline_01_training.py training script driven by a TOML config) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework in which the BYOL class is implemented (subclass of pytorch.nn.Module); supplies DataLoader-based training and optimization loop)
- **pip** (Installs the package dependencies from requirements.txt inside the Python 3.12 virtual environment)
- **venv** (Creates the isolated Python 3.12 virtual environment (python -m venv .venv) in which training is run)

## Examples

```
python astromorph/src/pipeline_01_training.py -c training_settings.toml
```

## Evaluation signals

- A trained checkpoint file exists on disk after the run (e.g. saved_models/newly_trained_network.pt).
- The TOML config parsed correctly: datafile points to the generated inputfiles.txt and network_name is 'n_layer_resnet' (or another supported network).
- If using the BYOL class directly, update_moving_average() was called after every optimization step — omitted, the target network will not track the online network and training is invalid.
- Downstream inference on the checkpoint produces embeddings whose visualisation (TensorBoard, --logdir=./runs) shows morphological clustering structure across the sample.
- Training ran for the configured number of epochs (default 10 if unspecified).

## Limitations

- Developed and guaranteed only for Python 3.12; anything below Python 3.9 will definitely not work and lower versions are unsupported.
- Using the raw BYOL class requires PyTorch training-loop experience; forgetting update_moving_average() after each optimization step breaks the target-network update.
- The pipeline scripts trade flexibility for ease of use — deeper customization requires working with the PyTorch-based classes directly.
- ResNet18 truncation via last_layer (e.g. layer2) removes later layers (layer3, layer4); the default n_layer_resnet uses the full network.

## Evidence

- [methods] Python 3.12 requirement: "This project has been developed for Python 3.12."
- [methods] filelist generation: "$ find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [methods] training script invocation: "python astromorph/src/pipeline_01_training.py -c example_settings.toml"
- [readme] moving average caveat: "**NB: remember to call the `update_moving_average()` method after every optimization step.**"
- [readme] package contents: "In this package we provide the following functionalities:

- the `BYOL` class as a PyTorch implementation of the BYOL framework;
- the `ByolTrainer` class wraps around `BYOL` to provide an easy"
- [abstract] label-free embeddings: "By enabling the generation of meaningful image embeddings without manually labelled data, the framework will enable key tasks such as clustering, anomaly detection, and similarity-based exploration."
