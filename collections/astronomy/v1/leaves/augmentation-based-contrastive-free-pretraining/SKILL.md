---
name: augmentation-based-contrastive-free-pretraining
description: Use when you need label-free image embeddings from FITS astronomical
  imaging (e.g. ALMA protoplanetary disks, Spitzer/Herschel infrared dark clouds)
  to enable clustering, anomaly detection, or similarity-based exploration without
  manually labelled data.
when_to_use_negative:
- When labelled training data is already available and a supervised classifier is
  the intended end point
- When the input is not imaging data (e.g. spectra, feature tables, or already-extracted
  morphological measurements), since BYOL operates on raw image views via augmentation
- When quick command-line use without PyTorch experience is needed and even the ByolTrainer
  wrapper's configuration burden is undesirable
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3318
tools:
- name: astromorph
  role: Python package providing the BYOL class, ByolTrainer wrapper, training/inference
    scripts, and FilelistDataset for FITS files
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework in which the BYOL class is implemented as a torch.nn.Module subclass
    with online and target (EMA) encoders and projector MLP
- name: TensorBoard
  role: Visualization of resulting embeddings after inference via tensorboard --logdir=./runs
- name: venv + pip
  role: 'Environment setup: python -m venv .venv and pip install -r requirements.txt
    (Python 3.12 target)'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:89c6d23c9a4b45f1d9a2ba2a01a8afac0b32192fb186f2ce8233d3de299e0555
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - TensorBoard
  - astromorph
  - venv + pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# augmentation-based-contrastive-free-pretraining

## Summary

Self-supervised pretraining of image embeddings with BYOL, an augmentation-based, contrastive-free method that trains an online network against an exponential-moving-average target network without negative pairs, as implemented in the astromorph PyTorch package for astronomical imaging.

## When to use

Use when you need label-free image embeddings from FITS astronomical imaging (e.g. ALMA protoplanetary disks, Spitzer/Herschel infrared dark clouds) to enable clustering, anomaly detection, or similarity-based exploration without manually labelled data.

## When NOT to use

- When labelled training data is already available and a supervised classifier is the intended end point
- When the input is not imaging data (e.g. spectra, feature tables, or already-extracted morphological measurements), since BYOL operates on raw image views via augmentation
- When quick command-line use without PyTorch experience is needed and even the ByolTrainer wrapper's configuration burden is undesirable

## Inputs

- Filelist of FITS image files (e.g. ALMA protoplanetary disk images, Spitzer/Herschel infrared dark cloud images), each preferably smaller than 10 MB
- TOML configuration file (e.g. example_settings.toml) specifying datafile, network_name, epochs, and [byol_settings]
- Optional binary mask FITS file for masked-data input

## Outputs

- Trained BYOL network saved as a .pt file (e.g. saved_models/newly_trained_network.pt)
- Image embeddings with metadata, exportable to CSV (export_to_csv = true)
- Embedding visualizations viewable in TensorBoard (--logdir=./runs)

## How to apply

Build a filelist of FITS files smaller than 10 MB using the `find` command (e.g. `find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt`), then run BYOL pretraining either through the `BYOL` class (a `torch.nn.Module` subclass offering full flexibility) or the easier `ByolTrainer` wrapper. Configure dimensionality via `byol_settings`: `representation_size = 128`, `projection_size = 16`, `projection_hidden_size = 512`, and `use_momentum = true` so the target network is the exponential moving average of the online network. Optionally truncate a ResNet18 backbone at `last_layer = "layer2"` (removing layer3/layer4) since earlier convolutional layers are usually more generic, or use the lightweight `AstroMorphologyModel` backbone; single-channel images can be stacked to the 3 channels ResNet expects via `stacksize = 3`. Rationale: BYOL feeds two independently augmented views of each image through the online and target branches, avoiding negative pairs; after every optimization step you must call `update_moving_average()` to keep the target network synchronized as required by the original procedure (arXiv:2006.07733). Run inference with the saved network (`saved_models/newly_trained_network.pt`) to produce embeddings, optionally exporting embeddings and metadata to CSV.

## Related tools

- **astromorph** (Python package providing the BYOL class, ByolTrainer wrapper, training/inference scripts, and FilelistDataset for FITS files) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework in which the BYOL class is implemented as a torch.nn.Module subclass with online and target (EMA) encoders and projector MLP)
- **TensorBoard** (Visualization of resulting embeddings after inference via tensorboard --logdir=./runs)
- **venv + pip** (Environment setup: python -m venv .venv and pip install -r requirements.txt (Python 3.12 target))

## Examples

```
python astromorph/src/pipeline_01_training.py -c example_settings.toml
```

## Evaluation signals

- Smoke-test: instantiate BYOL, run a forward pass on two augmented views, and call update_moving_average() without error
- Confirm update_moving_average() is invoked after every optimization step; omitting it desynchronizes the EMA target network and violates the BYOL procedure
- Check configured BYOL dimensions: representation_size=128, projection_size=16, projection_hidden_size=512, use_momentum=true
- If last_layer truncation is set, verify ResNet18 layers after the chosen layer (e.g. layer3/layer4 when last_layer="layer2") are removed
- Post-training, embeddings should support meaningful clustering/anomaly detection and capture morphological differences and similarities across large samples (verifiable in TensorBoard)

## Limitations

- The BYOL class requires some experience with setting up a training routine in PyTorch; ByolTrainer and the training script trade flexibility for ease of use
- Must remember to call update_moving_average() after every optimization step when using the BYOL class directly
- Developed for Python 3.12; lower versions might work but are not guaranteed, and anything below Python 3.9 will definitely not work
- ResNet18 expects 3-channel RGB images, so single-channel astronomical images need stacking (stacksize = 3)
- Validation of BYOL usage here reflects the package's design/mechanism; scientific meaningfulness was demonstrated only on the paper's two test datasets

## Evidence

- [methods] the `BYOL` class as a PyTorch implementation of the BYOL framework;: "the `BYOL` class as a PyTorch implementation of the BYOL framework;"
- [methods] remember to call update_moving_average() after every optimization step: "**NB: remember to call the `update_moving_average()` method after every optimization step.**"
- [readme] BYOL settings and defaults: "representation_size = 128
projection_size = 16
projection_hidden_size = 512
use_momentum = true # Target network is exponential MA of online network."
- [readme] Reduced ResNet18 rationale: "This might be beneficial, as the earlier layers are usually more generic."
- [abstract] Package purpose and embeddings: "By enabling the generation of meaningful image embeddings without manually labelled data, the framework will enable key tasks such as clustering, anomaly detection, and similarity-based exploration."
