---
name: self-supervised-byol-loss-design
description: Use when you need to pretrain image embeddings without labels from unlabeled
  astronomical FITS imagery (e.g., ALMA protoplanetary disk images, Spitzer/Herschel
  infrared dark cloud images), and want a flexible BYOL module rather than a fixed
  pipeline.
when_to_use_negative:
- You already have manually labelled data and a supervised classification objective
  — BYOL's label-free pretraining is designed for unlabeled corpora
- You only need embeddings quickly via the packaged pipeline scripts and do not require
  customization — the training script (pipeline_01_training.py) with a TOML config
  is simpler than building a custom `BYOL` training routine
- The input is not imagery (e.g., already-extracted feature tables or spectra) — this
  BYOL design targets 2D astronomical images
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: astromorph
  role: Python package providing the `BYOL` class, `ByolTrainer` wrapper, `AstroMorphologyModel`
    backbone, FilelistDataset for FITS files, and configurable training/inference
    scripts
  repo: https://github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework in which the `BYOL` class is implemented as a `torch.nn.Module`
    subclass and trained with standard PyTorch optimizers and DataLoaders
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:ec7632fda3a66b6904850ced7707364807874069f004316394299995578a2b9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/onsala-space-observatory/astromorph
  tools:
  - PyTorch
  - Python
  - astromorph
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# self-supervised-byol-loss-design

## Summary

Design and implement the Bootstrap Your Own Latents (BYOL) self-supervised framework as a PyTorch module for astronomical imaging, following the astromorph package's architecture: online and target encoders, projector MLP, symmetric augmentation-based loss without negative pairs, and an explicit update_moving_average() step after every optimization step.

## When to use

Use when you need to pretrain image embeddings without labels from unlabeled astronomical FITS imagery (e.g., ALMA protoplanetary disk images, Spitzer/Herschel infrared dark cloud images), and want a flexible BYOL module rather than a fixed pipeline.

## When NOT to use

- You already have manually labelled data and a supervised classification objective — BYOL's label-free pretraining is designed for unlabeled corpora
- You only need embeddings quickly via the packaged pipeline scripts and do not require customization — the training script (pipeline_01_training.py) with a TOML config is simpler than building a custom `BYOL` training routine
- The input is not imagery (e.g., already-extracted feature tables or spectra) — this BYOL design targets 2D astronomical images

## Inputs

- Unlabeled astronomical FITS images (e.g., filelist of FITS files generated with `find ... -name '**.fits'`)
- Backbone network instance (e.g., `AstroMorphologyModel` 2D CNN or ResNet18 with optional `last_layer` truncation)
- PyTorch `DataLoader` instances for training and test sets

## Outputs

- A trained `BYOL` / `ByolTrainer` model checkpoint (e.g., saved_models/newly_trained_network.pt)
- Image embeddings of dimensionality `representation_size` (default 128) usable for clustering, anomaly detection, and similarity-based exploration

## How to apply

Fix the framework definition against the original BYOL paper (arXiv:2006.07733): an online network, a target network maintained as the exponential moving average of the online network, a projector MLP, and a symmetric augmentation-based loss without negative pairs. Implement a `BYOL` class as a subclass of `torch.nn.Module`, wrapping a configurable backbone such as `AstroMorphologyModel` or a reduced ResNet18 truncated at a chosen `last_layer` (e.g. 'layer2'). Configure the projector with `representation_size` (default 128), `projection_size` (default 16), `projection_hidden_size` (default 512), and a `use_momentum` flag (default true) controlling whether the target network is the EMA of the online network. Implement the loss by feeding two independently augmented views of each image through the online and target branches, and implement `update_moving_average()` — the README explicitly warns to call this method after every optimization step, or the target network will lag incorrectly. For a lower-effort route, wrap the module in `ByolTrainer`, which handles the training loop, augmentation function, optimizer, and learning rate, requiring only two PyTorch `DataLoader` instances (train and test).

## Related tools

- **astromorph** (Python package providing the `BYOL` class, `ByolTrainer` wrapper, `AstroMorphologyModel` backbone, FilelistDataset for FITS files, and configurable training/inference scripts) — https://github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework in which the `BYOL` class is implemented as a `torch.nn.Module` subclass and trained with standard PyTorch optimizers and DataLoaders)

## Examples

```
model = ByolTrainer(AstroMorphologyModel(), representation_size=128); model.train_model(train_data=train_data, test_data=test_data, epochs=10)
```

## Evaluation signals

- A smoke test that instantiates `BYOL`, runs a forward pass on augmented image views, and calls `update_moving_average()` without error
- `update_moving_average()` is invoked after every optimization step, as required by the BYOL procedure
- Projector dimensions match the configured defaults or TOML settings: representation_size=128, projection_size=16, projection_hidden_size=512, use_momentum=true
- Reduced-ResNet backbones contain no layers beyond the configured `last_layer` (e.g., layer2 removed leaves only layer1–layer2)
- Downstream embeddings are scientifically meaningful and capture morphological differences/similarities across samples, as verified via TensorBoard visualization or clustering/anomaly detection

## Limitations

- The `BYOL` class provides the most flexibility but requires experience setting up a PyTorch training routine; forgetting to call `update_moving_average()` after every optimization step breaks the EMA target network
- The project is developed and tested for Python 3.12; anything below Python 3.9 will definitely not work
- Customization requires rebuilding the training loop, whereas `ByolTrainer` and the training script trade flexibility for ease of use

## Evidence

- [methods] the `BYOL` class as a PyTorch implementation of the BYOL framework;: "the `BYOL` class as a PyTorch implementation of the BYOL framework;"
- [readme] **NB: remember to call the `update_moving_average()` method after every optimization step.**: "**NB: remember to call the `update_moving_average()` method after every optimization step.**"
- [readme] The `BYOL` class is a subclass of `pytorch.nn.Module`, and can therefore be used like any other PyTorch module.: "The `BYOL` class is a subclass of `pytorch.nn.Module`, and can therefore be used like any other PyTorch module."
- [abstract] We present astromorph , a Python package that implements the BYOL method in a way tailored for astronomical imaging.: "We present astromorph , a Python package that implements the BYOL method in a way tailored for astronomical imaging."
- [readme] If we select `layer2` as the last convolutional layer, `layer3` and `layer4` will be removed from the network.: "If we select `layer2` as the last convolutional layer, `layer3` and `layer4` will be removed from the network."
