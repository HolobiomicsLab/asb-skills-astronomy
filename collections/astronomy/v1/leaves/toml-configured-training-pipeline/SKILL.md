---
name: toml-configured-training-pipeline
description: Use when a collection of unlabelled FITS images must be pre-trained with
  BYOL (morphology embeddings), and a reproducible training run can be specified in
  a TOML config.
when_to_use_negative:
- The data are already labelled, or a supervised classifier is required — the BYOL
  stage is explicitly label-free self-supervised pretraining
- You only need embeddings from an existing checkpoint — run pipeline_02_inference.py
  with trained_network_name instead of retraining
- You need full control over augmentation functions or optimizer beyond the config-file
  surface — use the BYOL PyTorch class directly, since the script trades flexibility
  for ease of use
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: astromorph
  role: Provides the BYOL class, ByolTrainer wrapper, FitsFilelistDataset, n_layer_resnet
    backbone, and the pipeline_01_training.py script driven by the TOML config
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework in which the BYOL module, ResNet18 backbone, and optimizer/EMA update
    mechanics are implemented
- name: pip
  role: Installs the package's requirements.txt inside the Python 3.12 venv before
    training
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:f002491f51ac8b34923b44139c18ea74d10465ba0c3c605f088a1990be373409
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Herschel
  - PyTorch
  - Python
  - Spitzer
  - astromorph
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# toml-configured-training-pipeline

## Summary

Runs astromorph's self-supervised BYOL training stage over a filelist of FITS images using a TOML settings file, producing a trained network checkpoint for later embedding inference.

## When to use

Use when a collection of unlabelled FITS images must be pre-trained with BYOL (morphology embeddings), and a reproducible training run can be specified in a TOML config.

## When NOT to use

- The data are already labelled, or a supervised classifier is required — the BYOL stage is explicitly label-free self-supervised pretraining
- You only need embeddings from an existing checkpoint — run pipeline_02_inference.py with trained_network_name instead of retraining
- You need full control over augmentation functions or optimizer beyond the config-file surface — use the BYOL PyTorch class directly, since the script trades flexibility for ease of use

## Inputs

- Filelist of FITS image paths (e.g. data/inputfiles.txt, generated with find -size -10M -name '**.fits')
- TOML settings file (datafile, network_name, epochs, optional [network_settings].last_layer, [byol_settings], [data_settings])
- Python 3.12 virtual environment with dependencies installed from requirements.txt

## Outputs

- Trained network checkpoint (e.g. saved_models/newly_trained_network.pt)
- Saved ResNet18-based backbone (full or truncated at last_layer) usable by pipeline_02_inference.py

## How to apply

Create a Python 3.12 virtual environment (`python -m venv .venv`) and install requirements with pip, then build a filelist of FITS inputs with the `find` command (e.g. files under 10 MB matching the .fits pattern) stored at data/inputfiles.txt. Write a TOML config with `datafile = "data/inputfiles.txt"`, `network_name = "n_layer_resnet"`, and optional `epochs` (default 10); use `[network_settings] last_layer = "layer2"` to truncate ResNet18 so layer3 and layer4 are removed, which can suit lower-dimension inputs since earlier layers are usually more generic. Run the training script from the repository root: `python astromorph/src/pipeline_01_training.py -c <config>.toml`, which wraps the BYOL module in ByolTrainer over the FitsFilelistDataset with the AstroMorphologyModel backbone. If assembling a custom loop, remember that the target network is an exponential moving average of the online network, so `update_moving_average()` must be called after every optimization step (use_momentum = true in [byol_settings]). The trained checkpoint (e.g. saved_models/newly_trained_network.pt) is later referenced via `trained_network_name` by the inference script, which can reuse the same config because non-relevant options such as `epochs` are ignored.

## Related tools

- **astromorph** (Provides the BYOL class, ByolTrainer wrapper, FitsFilelistDataset, n_layer_resnet backbone, and the pipeline_01_training.py script driven by the TOML config) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework in which the BYOL module, ResNet18 backbone, and optimizer/EMA update mechanics are implemented)
- **pip** (Installs the package's requirements.txt inside the Python 3.12 venv before training)

## Examples

```
python astromorph/src/pipeline_01_training.py -c training_settings.toml
```

## Evaluation signals

- The trained checkpoint file (e.g. saved_models/newly_trained_network.pt) exists on disk after the training run completes
- A build with network_name='n_layer_resnet' and no last_layer yields the full ResNet18, while last_layer='layer2' yields a backbone containing layer1 and layer2 but no layer3 or layer4 (verified by module inspection and a dummy forward pass)
- update_moving_average() is invoked after every optimization step so the target network tracks the exponential moving average of the online network
- The checkpoint loads in pipeline_02_inference.py and, with export_to_csv = true, produces a CSV of embeddings plus metadata (inspectable in TensorBoard via ./runs)

## Limitations

- The project was developed for Python 3.12; lower versions might work but are unguaranteed, and anything below Python 3.9 will definitely not work
- The n_layer_resnet truncation only supports selecting one of the four ResNet18 convolutional stages (layer1–layer4) as the last layer; default is the full ResNet18
- The training script prioritizes ease of use over flexibility — the BYOL class provides the most flexibility but requires PyTorch training-routine experience

## Evidence

- [readme] The settings of the training run are specified through a TOML file. An example of such a file can be found in `example_settings.toml`.: "The settings of the training run are specified through a TOML file.
An example of such a file can be found in `example_settings.toml`."
- [readme] The script should be invoked from the main folder of the repository: python astromorph/src/pipeline_01_training.py -c example_settings.toml: "The script should be invoked from the main folder of the repository:

```bash
python astromorph/src/pipeline_01_training.py -c example_settings.toml
```"
- [readme] If we select `layer2` as the last convolutional layer, `layer3` and `layer4` will be removed from the network.: "If we select `layer2` as the last convolutional layer, `layer3` and `layer4`
will be removed from the network."
- [methods] remember to call the `update_moving_average()` method after every optimization step: "**NB: remember to call the `update_moving_average()` method after every optimization step.**"
- [readme] the `BYOL` class as a PyTorch implementation of the BYOL framework; the `ByolTrainer` class wraps around `BYOL`: "- the `BYOL` class as a PyTorch implementation of the BYOL framework;
- the `ByolTrainer` class wraps around `BYOL` to provide an easy training interface;"
