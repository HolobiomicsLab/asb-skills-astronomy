---
name: selectable-backbone-wrapping
description: Use when building a BYOL-based self-supervised pipeline for astronomical
  images and you need to choose or substitute the encoder backbone — e.g. trading
  the full ResNet18 for an earlier truncation like layer2 on small or single-channel
  FITS images.
when_to_use_negative:
- You already have labelled data and a supervised classification objective — BYOL
  backbone wrapping is designed for label-free embedding generation
- You need full control over the training loop internals beyond what BYOL/ByolTrainer
  expose — the README notes ByolTrainer and the training script trade flexibility
  for ease-of-use
- Your input is not image data (e.g. already-extracted feature tables or catalogues)
  — the pipeline expects FITS images or filelists of FITS files
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: astromorph
  role: Provides the BYOL class, ByolTrainer wrapper, and AstroMorphologyModel backbone
    used to wrap selectable encoders
  repo: https://github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework in which the BYOL class is implemented as a torch.nn.Module subclass
    wrapping the configurable backbone
- name: Python 3.12
  role: Runtime environment (venv + pip install -r requirements.txt) for running the
    training and inference scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:4b03121c2d1db68f975ee9c29431bfe9ee25d91239efd5203f7bc5ae685007ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/onsala-space-observatory/astromorph
  tools:
  - PyTorch
  - Python
  - Python 3.12
  - astromorph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# selectable-backbone-wrapping

## Summary

Wraps a configurable convolutional backbone (the light-weight AstroMorphologyModel or a ResNet18 truncated at a chosen layer) inside astromorph's BYOL PyTorch module so that the encoder feeding self-supervised embeddings can be swapped without rewriting the training loop.

## When to use

Use when building a BYOL-based self-supervised pipeline for astronomical images and you need to choose or substitute the encoder backbone — e.g. trading the full ResNet18 for an earlier truncation like layer2 on small or single-channel FITS images.

## When NOT to use

- You already have labelled data and a supervised classification objective — BYOL backbone wrapping is designed for label-free embedding generation
- You need full control over the training loop internals beyond what BYOL/ByolTrainer expose — the README notes ByolTrainer and the training script trade flexibility for ease-of-use
- Your input is not image data (e.g. already-extracted feature tables or catalogues) — the pipeline expects FITS images or filelists of FITS files

## Inputs

- Filelist of FITS files (e.g. data/inputfiles.txt produced with find, files < 10 MB)
- TOML configuration file (example_settings.toml) with network_name, network_settings.last_layer, byol_settings, and data_settings
- Backbone selection: AstroMorphologyModel or n_layer_resnet with last_layer in {layer1, layer2, layer3, layer4}

## Outputs

- A trained BYOL-wrapped encoder network saved as a .pt file (e.g. saved_models/newly_trained_network.pt)
- Image embeddings via the configurable inference script, optionally exported to CSV (export_to_csv = true)

## How to apply

Instantiate the `BYOL` class (a subclass of `torch.nn.Module`) or the easier `ByolTrainer` wrapper, passing the chosen backbone — either the package's light-weight 2D CNN `AstroMorphologyModel` or the `n_layer_resnet` network. To truncate ResNet18, set `last_layer` inside the `network_settings` dictionary of the TOML config (e.g. `last_layer = "layer2"` removes `layer3` and `layer4`), since earlier convolutional layers are usually more generic. Dimensionality of the BYOL components is configured in `[byol_settings]` with `representation_size = 128`, `projection_size = 16`, `projection_hidden_size = 512`, and `use_momentum = true` so the target network is the exponential moving average of the online network. If data are single-channel but the backbone expects 3-channel RGB input, set `stacksize = 3` under `[data_settings]`. After every optimization step, call `update_moving_average()` on the BYOL module, or use `ByolTrainer` which manages this for you.

## Related tools

- **astromorph** (Provides the BYOL class, ByolTrainer wrapper, and AstroMorphologyModel backbone used to wrap selectable encoders) — https://github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework in which the BYOL class is implemented as a torch.nn.Module subclass wrapping the configurable backbone)
- **Python 3.12** (Runtime environment (venv + pip install -r requirements.txt) for running the training and inference scripts)

## Examples

```
python astromorph/src/pipeline_01_training.py -c example_settings.toml  # with [network_settings] last_layer = "layer2"
```

## Evaluation signals

- A forward pass through the wrapped BYOL module runs without error and calls update_moving_average() after every optimization step
- The truncated ResNet18 no longer contains layers after the chosen last_layer (e.g. setting last_layer = 'layer2' removes layer3 and layer4)
- BYOL component dimensions match the config: representation_size=128, projection_size=16, projection_hidden_size=512, use_momentum=true
- The trained network saved as .pt loads in pipeline_02_inference.py and produces embeddings exportable to CSV
- Resulting embeddings are inspectable in TensorBoard (tensorboard --logdir=./runs) and show meaningful morphological clustering

## Limitations

- Truncating ResNet18 removes deeper layers; this may be beneficial since earlier layers are usually more generic, but the full network is the default and truncation depth must be chosen per dataset
- Using the BYOL class directly 'requires some experience with setting up a training routine in PyTorch'; forgetting update_moving_average() after each optimization step breaks the EMA target network
- Python below 3.9 definitely will not work; the project is developed for Python 3.12 with no guarantee for lower versions

## Evidence

- [methods] the `BYOL` class as a PyTorch implementation of the BYOL framework;: "the `BYOL` class as a PyTorch implementation of the BYOL framework;"
- [readme] readme: "It is possible to use only a few of the convolutional layers of the ResNet18 network."
