---
name: self-supervised-encoder-setup
description: Use when preparing BYOL pretraining of an encoder on FITS images (e.g.
  ALMA protoplanetary disk or Spitzer/Herschel infrared dark cloud data) and you must
  select a backbone (network_name) or truncate ResNet18 at a chosen stage such as
  layer2.
when_to_use_negative:
- You need maximum flexibility for a custom training routine and already have PyTorch
  experience — use the BYOL class directly rather than the config-driven n_layer_resnet
  path
- Your encoder must be deep/semantic rather than generic (e.g. you need layer4 features)
  — truncating at an early last_layer such as layer2 removes layer3 and layer4 and
  is not appropriate
- The task is inference/embedding extraction on an already-trained network rather
  than encoder setup and pretraining (use pipeline_02_inference.py instead)
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0769
tools:
- name: astromorph
  role: Package providing the BYOL class, ByolTrainer, configurable training script,
    and the n_layer_resnet backbone selection mechanism
  repo: https://github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework in which the encoder backbone is built as a pytorch.nn.Module subclass
    and the BYOL framework is implemented
- name: pip
  role: Installs the astromorph requirements (pip install -r requirements.txt) needed
    to run the training script
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:851ff62d7606368ea12c46a02ca8aac9757115119774182722cb5253320fa7b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/onsala-space-observatory/astromorph
  tools:
  - PyTorch
  - Python
  - astromorph
  - pip
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# self-supervised-encoder-setup

## Summary

Configure and build the self-supervised (BYOL) encoder backbone for astromorph, including the n_layer_resnet variant whose ResNet18 truncation depth is selectable via the TOML training configuration (last_layer keyword). This lets the encoder be adapted to astronomical FITS inputs of varying dimensions and resolutions by keeping only the earlier, more generic convolutional stages.

## When to use

Use when preparing BYOL pretraining of an encoder on FITS images (e.g. ALMA protoplanetary disk or Spitzer/Herschel infrared dark cloud data) and you must select a backbone (network_name) or truncate ResNet18 at a chosen stage such as layer2.

## When NOT to use

- You need maximum flexibility for a custom training routine and already have PyTorch experience — use the BYOL class directly rather than the config-driven n_layer_resnet path
- Your encoder must be deep/semantic rather than generic (e.g. you need layer4 features) — truncating at an early last_layer such as layer2 removes layer3 and layer4 and is not appropriate
- The task is inference/embedding extraction on an already-trained network rather than encoder setup and pretraining (use pipeline_02_inference.py instead)

## Inputs

- TOML training configuration file (e.g. training_settings.toml with datafile, epochs, network_name, [network_settings].last_layer, [byol_settings])
- Filelist of FITS files (e.g. data/inputfiles.txt generated with find)
- PyTorch / Python 3.12 environment with astromorph installed from requirements.txt

## Outputs

- A PyTorch encoder module (full ResNet18, or a truncated ResNet18 containing only layer1 and layer2 when last_layer = 'layer2') wired into the BYOL pipeline
- A trained self-supervised network saved under a trained_network_name such as saved_models/newly_trained_network.pt

## How to apply

Read the TOML training configuration to obtain network_name and the optional [network_settings].last_layer key (e.g. last_layer = "layer2"). Construct a ResNet18 backbone as a pytorch.nn.Module subclass; in the default case (no last_layer given) retain all four convolutional stages layer1, layer2, layer3, and layer4, so the full ResNet18 is used. When last_layer is specified, remove every convolutional stage that follows it — selecting layer2 removes layer3 and layer4 from the network — because earlier layers are usually more generic. Wire the builder into the training-config path keyed on network_name so it can be selected via the config file, then pass the config to the training script. Optionally set [byol_settings] such as representation_size = 128, projection_size = 16, projection_hidden_size = 512, and use_momentum = true; remember that with the BYOL class the update_moving_average() method must be called after every optimization step.

## Related tools

- **astromorph** (Package providing the BYOL class, ByolTrainer, configurable training script, and the n_layer_resnet backbone selection mechanism) — https://github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework in which the encoder backbone is built as a pytorch.nn.Module subclass and the BYOL framework is implemented)
- **pip** (Installs the astromorph requirements (pip install -r requirements.txt) needed to run the training script)

## Examples

```
python astromorph/src/pipeline_01_training.py -c training_settings.toml
```

## Evaluation signals

- A build with network_name='n_layer_resnet' and no last_layer yields the full ResNet18 (layer1 through layer4 all present), verified by module inspection
- With last_layer='layer2', module inspection shows the backbone contains layer1 and layer2 but no layer3 or layer4 modules
- A dummy forward pass through the constructed backbone succeeds for the intended input dimensions
- The TOML config with network_name='n_layer_resnet' selects the correct builder via the training script (python astromorph/src/pipeline_01_training.py -c training_settings.toml)
- During training with the BYOL class, update_moving_average() is invoked after every optimization step

## Limitations

- Truncating at an early last_layer (e.g. layer2) removes layer3 and layer4, which changes the feature level captured; earlier layers are usually more generic, which may or may not suit the target data
- The package has been developed for Python 3.12; lower versions might work but there is no guarantee, and anything below Python 3.9 will definitely not work
- The streamlined config-driven training path trades flexibility for ease of use; the BYOL class provides the most flexibility but requires experience with setting up a training routine in PyTorch

## Evidence

- [readme] It is possible to use only a few of the convolutional layers of the ResNet18 network.: "It is possible to use only a few of the convolutional layers of the ResNet18 network."
- [readme] last_layer keyword usage: "To invoke this possibility, use the `last_layer` keyword inside the `network_settings` dictionary.
By default, `n_layer_resnet` will use the full ResNet18 network."
- [abstract] deeper customization via PyTorch: "The package is built with usability in mind, and offers streamlined pipeline scripts for ease of use as well as deeper customization options via PyTorch-based classes."
- [methods] update_moving_average requirement: "**NB: remember to call the `update_moving_average()` method after every optimization step.**"
- [methods] training script invocation: "python astromorph/src/pipeline_01_training.py -c example_settings.toml"
