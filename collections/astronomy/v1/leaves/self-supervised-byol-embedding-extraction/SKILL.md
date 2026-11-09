---
name: self-supervised-byol-embedding-extraction
description: Use when you have a directory of unlabeled FITS images of astronomical
  objects (e.g. ALMA disks or Spitzer/Herschel infrared dark clouds) and need morphology-based
  embeddings without manual labels.
when_to_use_negative:
- Inputs are already feature vectors or a tabular feature table rather than image
  FITS files — use a clustering method directly instead.
- The task requires labeled, supervised classification with ground-truth morphological
  classes; astromorph is explicitly a label-free/self-supervised approach.
- No pretrained checkpoint is available and only a handful of images exist — BYOL
  self-supervised pretraining needs a sufficiently large sample to learn meaningful
  embeddings.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: astromorph
  role: Python package providing the BYOL implementation, training/inference pipeline
    scripts, and CSV embedding export used to extract embeddings
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework underlying the BYOL class, ByolTrainer, and the AstroMorphologyModel
    convolutional network used at inference
- name: TensorBoard
  role: Visualization of the exported embeddings after inference
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:7a9f9fb2232bd9ef28ae9afc6b74a1989c632a5ad466c90db4f8b9e45261867b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Herschel
  - PyTorch
  - Spitzer
  - TensorBoard
  - astromorph
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# self-supervised-byol-embedding-extraction

## Summary

Run astromorph's BYOL-based inference pipeline to extract label-free image embeddings from FITS astronomical images (e.g. Spitzer/Herschel infrared dark clouds) and export them with metadata to CSV for downstream clustering or visual inspection.

## When to use

Use when you have a directory of unlabeled FITS images of astronomical objects (e.g. ALMA disks or Spitzer/Herschel infrared dark clouds) and need morphology-based embeddings without manual labels.

## When NOT to use

- Inputs are already feature vectors or a tabular feature table rather than image FITS files — use a clustering method directly instead.
- The task requires labeled, supervised classification with ground-truth morphological classes; astromorph is explicitly a label-free/self-supervised approach.
- No pretrained checkpoint is available and only a handful of images exist — BYOL self-supervised pretraining needs a sufficiently large sample to learn meaningful embeddings.

## Inputs

- FITS images of astronomical objects (e.g. Spitzer/Herschel infrared dark cloud images, ALMA protoplanetary disk images)
- text filelist of FITS file paths (generated with the `find` command)
- TOML configuration file with `datafile`, `trained_network_name`, and `export_to_csv` keywords
- pretrained network checkpoint (.pt file, e.g. saved_models/newly_trained_network.pt)

## Outputs

- CSV file containing embeddings and metadata for each input image
- TensorBoard visualization of the embedding space (./runs)

## How to apply

Build a filelist of FITS files with `find` (e.g. filtering on file size, such as `-size -10M`, and the .fits extension) and reference it via the `datafile` keyword of a TOML config. Point the config at a pretrained checkpoint with `trained_network_name` (e.g. `saved_models/newly_trained_network.pt`) and set `export_to_csv = true`. Run `pipeline_02_inference.py` with the config; command-line flags (e.g. `-n <trained-network-file>`) can be combined with the config file, and CLI options overrule config settings. Collect the exported CSV of embeddings plus metadata, and optionally inspect separation in TensorBoard (`tensorboard --logdir=./runs`). The underlying BYOL model was trained via self-supervised pretraining (BYOL class / ByolTrainer with `AstroMorphologyModel`), requiring `update_moving_average()` after every optimization step if training custom networks.

## Related tools

- **astromorph** (Python package providing the BYOL implementation, training/inference pipeline scripts, and CSV embedding export used to extract embeddings) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework underlying the BYOL class, ByolTrainer, and the AstroMorphologyModel convolutional network used at inference)
- **TensorBoard** (Visualization of the exported embeddings after inference)

## Examples

```
python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt; tensorboard --logdir=./runs
```

## Evaluation signals

- The exported CSV exists and contains an embedding vector plus metadata for every file in the input filelist.
- Embedding table shows scientifically meaningful separation reflecting morphological differences and similarities across the sample (e.g. infrared dark clouds grouping by morphology).
- TensorBoard projection of ./runs shows interpretable clustering rather than random dispersion.
- Inference runs without error when the config file is combined with CLI flags, with CLI options correctly overruling config settings.
- If a custom reduced ResNet18 was configured (e.g. last_layer = 'layer2'), the output embedding dimensionality matches the configured representation_size (e.g. 128).

## Limitations

- The project is developed for Python 3.12; lower versions may work but anything below Python 3.9 will definitely not work.
- When training custom BYOL models, `update_moving_average()` must be called after every optimization step or the target network is stale.
- Filelist-based input examples filter on FITS file size (e.g. < 10 MB); very large images may need filtering or handling outside this pipeline.
- A pretrained network checkpoint must exist (from pipeline_01_training.py) before inference can run; the README notes training is detailed in Boschman et al. (in preparation).

## Evidence

- [other] For the infrared dark cloud (Spitzer and Herschel) science case, astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across the sample.: [evidence span withheld — anchor status: not_found]
- [abstract] In both cases we demonstrate how astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples.: "In both cases we demonstrate how astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples."
- [methods] Export embeddings and metadata to CSV: "export_to_csv = true # Export embeddings and metadata to a CSV file"
- [readme] TensorBoard visualization command: "To view a visualisation of the embeddings after inference, use TensorBoard with:"
- [readme] CLI overrules config: "The options given in the command line will overrule the settings specified in the config file."
