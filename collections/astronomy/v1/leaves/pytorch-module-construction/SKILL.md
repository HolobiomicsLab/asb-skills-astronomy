---
name: pytorch-module-construction
description: Use when a training pipeline must accept a user-configurable neural backbone,
  e.g. when input images are small/low-resolution and the full ResNet18 (layer1–layer4)
  is too deep, or the backbone must be chosen from a config file.
when_to_use_negative:
- Input images are large/high-resolution and benefit from the full four-stage ResNet18
  receptive field — keep the default (no last_layer).
- The goal is a ready-made backbone without custom depth control — the package's built-in
  AstroMorphologyModel or the streamlined training script already suffices.
- The task needs a non-ResNet architecture family; this skill only implements truncation
  of ResNet18's four convolutional stages.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Framework in which the backbone is implemented as an nn.Module subclass and
    stages are added/removed
  repo: https://github.com/onsala-space-observatory/astromorph
- name: astromorph
  role: Provides the BYOL class, ByolTrainer, and the TOML-configured training script
    (pipeline_01_training.py) into which the custom backbone is wired
  repo: https://github.com/onsala-space-observatory/astromorph
- name: Python
  role: Implementation language (project developed for Python 3.12)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:46e2e1a56881c3e138441df14f994a35ed649585403872a548d92f47370c6ab1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/onsala-space-observatory/astromorph
  tools:
  - PyTorch
  - Python
  - astromorph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# pytorch-module-construction

## Summary

Build a custom PyTorch nn.Module backbone — here a truncated ResNet18 ('n_layer_resnet') whose depth is selected via the TOML training config — so that BYOL training in astromorph can be adapted to input images of varying dimensions and resolutions.

## When to use

Use when a training pipeline must accept a user-configurable neural backbone, e.g. when input images are small/low-resolution and the full ResNet18 (layer1–layer4) is too deep, or the backbone must be chosen from a config file.

## When NOT to use

- Input images are large/high-resolution and benefit from the full four-stage ResNet18 receptive field — keep the default (no last_layer).
- The goal is a ready-made backbone without custom depth control — the package's built-in AstroMorphologyModel or the streamlined training script already suffices.
- The task needs a non-ResNet architecture family; this skill only implements truncation of ResNet18's four convolutional stages.

## Inputs

- TOML training configuration file with network_name and optional [network_settings].last_layer key
- ResNet18 architecture from PyTorch (torchvision-style stages layer1, layer2, layer3, layer4)

## Outputs

- A pytorch.nn.Module backbone ('n_layer_resnet'), either the full ResNet18 or a truncated variant containing only layers up to last_layer
- A config-keyed network builder selectable via network_name in the TOML file

## How to apply

Subclass pytorch.nn.Module and construct a ResNet18 backbone registered under the network_name 'n_layer_resnet' in the training-config path. Read the TOML config for network_name and the optional [network_settings].last_layer key (e.g. last_layer = "layer2"): when it is absent, retain all four convolutional stages layer1–layer4 (the full ResNet18); when it is set, drop every stage after the named one (selecting layer2 removes layer3 and layer4), since earlier layers are usually more generic. Wire the builder into the config-driven training script so the choice is made purely from the TOML file, keeping the BYOL wrapper (update_moving_average after every optimization step) untouched.

## Related tools

- **PyTorch** (Framework in which the backbone is implemented as an nn.Module subclass and stages are added/removed) — https://github.com/onsala-space-observatory/astromorph
- **astromorph** (Provides the BYOL class, ByolTrainer, and the TOML-configured training script (pipeline_01_training.py) into which the custom backbone is wired) — https://github.com/onsala-space-observatory/astromorph
- **Python** (Implementation language (project developed for Python 3.12))

## Examples

```
[network_settings]
last_layer = "layer2"
# in training_settings.toml: datafile = "data/inputfiles.txt"; network_name = "n_layer_resnet"
python astromorph/src/pipeline_01_training.py -c training_settings.toml
```

## Evaluation signals

- network_name='n_layer_resnet' with no last_layer yields the full ResNet18 (layer1–layer4 all present) as verified by module inspection
- last_layer='layer2' yields a backbone containing layer1 and layer2 but no layer3 or layer4 modules
- A dummy forward pass through the constructed backbone succeeds and produces a valid feature tensor
- The backbone is selectable purely via the TOML config (network_name + [network_settings].last_layer) without code changes
- Training still runs when the custom backbone is plugged into ByolTrainer, with update_moving_average() called after every optimization step

## Limitations

- Truncation only supports the four named ResNet18 stages (layer1–layer4); selecting layer2 removes layer3 and layer4 — no arbitrary cut points.
- By default n_layer_resnet uses the full ResNet18; other networks require their own builders keyed on network_name.
- Deeper customization via PyTorch classes requires experience with setting up a PyTorch training routine; the streamlined scripts trade flexibility for ease of use.

## Evidence

- [readme] Reduced ResNet18 network: "It is possible to use only a few of the convolutional layers of the ResNet18 network. There are four convolutional layers in ResNet18, named `layer1`, `layer2`, `layer3`, and `layer4`."
- [readme] last_layer semantics: "If we select `layer2` as the last convolutional layer, `layer3` and `layer4` will be removed from the network."
- [readme] last_layer config key: "To invoke this possibility, use the `last_layer` keyword inside the `network_settings` dictionary. By default, `n_layer_resnet` will use the full ResNet18 network."
- [abstract] customization rationale: "The package is built with usability in mind, and offers streamlined pipeline scripts for ease of use as well as deeper customization options via PyTorch-based classes."
