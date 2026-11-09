---
name: self-supervised-embedding-inference
description: Use when a trained astromorph network checkpoint (.pt) exists and you
  need embedding vectors for a new batch of FITS images — e.g. ALMA protoplanetary
  disk images — to inspect morphological groupings.
when_to_use_negative:
- No trained checkpoint exists yet — the BYOL training stage (pipeline_01_training.py)
  must be completed first, as inference requires trained_network_name.
- Input is already a feature/embedding table or a metadata catalogue rather than image
  files; inference operates on FITS images loaded via FitsFilelistDataset.
- The goal is supervised classification with manually labelled data — astromorph's
  inference stage is explicitly a label-free embedding step, not a label-trained predictor.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: astromorph
  role: Supplies pipeline_02_inference.py, the FitsFilelistDataset for loading FITS
    sets, and the AstroMorphologyModel encoder whose trained weights produce the embeddings
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Backend on which the BYOL module, AstroMorphologyModel (ResNet18-based), and
    DataLoader run during the inference pass
- name: TensorBoard
  role: Embedding projector used to visualise the exported vectors with per-object
    metadata from ./runs
- name: Python 3.12
  role: Runtime environment (venv + pip install -r requirements.txt) in which the
    inference script is executed
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:c8ca5b0cdad3c8a369c5a0c900a6443982b6ed87adfd1caa0fd2d66055af7161
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA (Atacama Large Millimeter/submillimeter Array)
  - PyTorch
  - Python
  - Python 3.12
  - TensorBoard
  - astromorph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# self-supervised-embedding-inference

## Summary

Inference stage of the astromorph pipeline: run a trained BYOL/ResNet18-based encoder (AstroMorphologyModel) over a filelist of FITS images (e.g. ALMA protoplanetary disk images) to compute label-free embeddings, export them with metadata to CSV, and inspect them in the TensorBoard embedding projector.

## When to use

Use when a trained astromorph network checkpoint (.pt) exists and you need embedding vectors for a new batch of FITS images — e.g. ALMA protoplanetary disk images — to inspect morphological groupings.

## When NOT to use

- No trained checkpoint exists yet — the BYOL training stage (pipeline_01_training.py) must be completed first, as inference requires trained_network_name.
- Input is already a feature/embedding table or a metadata catalogue rather than image files; inference operates on FITS images loaded via FitsFilelistDataset.
- The goal is supervised classification with manually labelled data — astromorph's inference stage is explicitly a label-free embedding step, not a label-trained predictor.

## Inputs

- Filelist of FITS image paths (e.g. data/inputfiles.txt), built with find and typically restricted to files < 10 MB
- Trained network checkpoint saved as .pt (e.g. saved_models/newly_trained_network.pt)
- TOML configuration file specifying datafile, trained_network_name, export_to_csv, and [network_settings] last_layer

## Outputs

- CSV file containing embedding vectors with per-image metadata (export_to_csv = true)
- TensorBoard projector checkpoint (embedding tensor plus metadata/labels file) in the ./runs log directory

## How to apply

Build a filelist of FITS image paths with the find command (e.g. files under 10 MB written to data/inputfiles.txt), mirroring the package's filelist convention. Write a TOML config that points the datafile keyword at this filelist, sets trained_network_name to the saved checkpoint (e.g. saved_models/newly_trained_network.pt), and enables export_to_csv = true; non-relevant training options such as epochs are ignored, so the training config can be reused. Keep the [network_settings] last_layer truncation (e.g. "layer2") identical to the training run, since the AstroMorphologyModel encoder output — and the embedding dimensionality tied to representation_size (default 128) — depends on that truncation. Run python astromorph/src/pipeline_02_inference.py -c <config>.toml (CLI flags like -n overrule config settings), letting the FitsFilelistDataset load the images and the BYOL-trained encoder compute one embedding vector per image. Export embeddings plus metadata to CSV, write a TensorBoard projector checkpoint (embedding tensor + metadata/labels) into ./runs, and launch tensorboard --logdir=./runs to visually inspect morphological differences and similarities across the sample.

## Related tools

- **astromorph** (Supplies pipeline_02_inference.py, the FitsFilelistDataset for loading FITS sets, and the AstroMorphologyModel encoder whose trained weights produce the embeddings) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Backend on which the BYOL module, AstroMorphologyModel (ResNet18-based), and DataLoader run during the inference pass)
- **TensorBoard** (Embedding projector used to visualise the exported vectors with per-object metadata from ./runs)
- **Python 3.12** (Runtime environment (venv + pip install -r requirements.txt) in which the inference script is executed)

## Examples

```
python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt
```

## Evaluation signals

- A CSV file with embedding vectors and aligned metadata is produced when export_to_csv = true, with one row per input FITS file.
- TensorBoard started with 'tensorboard --logdir=./runs' loads the embedding projector and shows per-object metadata aligned one-to-one with the embedding rows.
- Embedding dimensionality matches the trained model's representation_size (default 128) and the last_layer truncation used at training (e.g. layer2).
- Morphologically similar ALMA protoplanetary disks group together in the projector, consistent with the paper's finding that embeddings capture morphological differences and similarities across large samples.
- Every path listed in data/inputfiles.txt is processed by FitsFilelistDataset, matching the find command's -size -10M selection.

## Limitations

- Inference requires a previously trained checkpoint (trained_network_name pointing to a saved .pt file); no embeddings can be produced from an untrained network.
- If training used a truncated ResNet18 (e.g. last_layer = "layer2"), inference must repeat the same truncation or the encoder output will not match the trained network.
- The demonstrated filelist convention selects FITS files smaller than 10 MB via find -size -10M, excluding larger files; single-channel images fed to a 3-channel network require [data_settings] stacksize = 3.

## Evidence

- [abstract] We present astromorph , a Python package that implements the BYOL method in a way tailored for astronomical imaging.: "We present astromorph , a Python package that implements the BYOL method in a way tailored for astronomical imaging."
- [abstract] In both cases we demonstrate how astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples.: "In both cases we demonstrate how astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples."
- [methods] export_to_csv = true # Export embeddings and metadata to a CSV file: "export_to_csv = true # Export embeddings and metadata to a CSV file"
- [readme] a configurable inference script for easy inspection of the resulting embeddings.: "a configurable inference script for easy inspection of the resulting embeddings."
- [readme] $ find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt: "$ find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [readme] python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt: "python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt"
