---
name: astronomical-image-morphology-embedding
description: Use when you have a large collection of unlabelled astronomical FITS
  images (e.g. ALMA disk observations) and need label-free embeddings that capture
  morphological differences for clustering, anomaly detection, or similarity-based
  exploration.
when_to_use_negative:
- Your data is already a labelled feature table or catalog — BYOL self-supervised
  embedding of raw images is unnecessary
- Images exceed the pipeline's 10 MB per-file filelist convention or are not FITS
  format without adaptation
- You need supervised classification with predefined labels rather than label-free
  morphological pattern discovery
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: astromorph
  role: Python package implementing BYOL tailored for astronomical imaging; provides
    BYOL, ByolTrainer, AstroMorphologyModel, FitsFilelistDataset, and pipeline_01_training.py
    / pipeline_02_inference.py scripts
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Underlying deep-learning framework; the BYOL class is a pytorch.nn.Module
    subclass and the pipeline offers deeper customization via PyTorch-based classes
- name: TensorBoard
  role: Visualization of the embedding projector after inference via tensorboard --logdir=./runs
- name: ALMA (Atacama Large Millimeter/submillimeter Array)
  role: Source instrument for the protoplanetary disk FITS images used as one of the
    science demonstrations
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:097b4455027eb224b195b02241dc9da5f7507bb26738699eb62d97a5a856a9b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA (Atacama Large Millimeter/submillimeter Array)
  - PyTorch
  - Python
  - TensorBoard
  - astromorph
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# astronomical-image-morphology-embedding

## Summary

A self-supervised (BYOL-based) pipeline, implemented in the astromorph Python package, that generates meaningful image embeddings from FITS astronomical images without manually labelled data, demonstrated on ALMA protoplanetary disk images and Spitzer/Herschel infrared dark clouds.

## When to use

Use when you have a large collection of unlabelled astronomical FITS images (e.g. ALMA disk observations) and need label-free embeddings that capture morphological differences for clustering, anomaly detection, or similarity-based exploration.

## When NOT to use

- Your data is already a labelled feature table or catalog — BYOL self-supervised embedding of raw images is unnecessary
- Images exceed the pipeline's 10 MB per-file filelist convention or are not FITS format without adaptation
- You need supervised classification with predefined labels rather than label-free morphological pattern discovery

## Inputs

- Collection of astronomical FITS image files (< 10 MB each), e.g. ALMA protoplanetary disk continuum images
- Filelist of FITS file paths (data/inputfiles.txt)
- TOML configuration file (datafile, network_name, trained_network_name, export_to_csv, network_settings.last_layer)
- Trained BYOL network checkpoint (.pt, e.g. saved_models/newly_trained_network.pt)

## Outputs

- CSV file of image embedding vectors with associated metadata
- TensorBoard projector log in ./runs for embedding visualization

## How to apply

Build a filelist of FITS files under 10 MB using the find command (e.g. `find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt`), set up a Python 3.12 virtual environment, and install requirements with pip. Write a TOML config specifying datafile, network_name (e.g. the ResNet18-based AstroMorphologyModel, optionally truncated via last_layer such as 'layer2'), and run pipeline_01_training.py for BYOL self-supervised pretraining, remembering to call update_moving_average() after every optimization step. Then run the configurable inference script pipeline_02_inference.py with trained_network_name and export_to_csv = true so the FitsFilelistDataset-loaded images are embedded and exported to CSV. Optionally visualize the embeddings with TensorBoard via `tensorboard --logdir=./runs`. Success is judged by a CSV of embedding vectors plus metadata, and a TensorBoard projector showing morphological groupings.

## Related tools

- **astromorph** (Python package implementing BYOL tailored for astronomical imaging; provides BYOL, ByolTrainer, AstroMorphologyModel, FitsFilelistDataset, and pipeline_01_training.py / pipeline_02_inference.py scripts) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Underlying deep-learning framework; the BYOL class is a pytorch.nn.Module subclass and the pipeline offers deeper customization via PyTorch-based classes)
- **TensorBoard** (Visualization of the embedding projector after inference via tensorboard --logdir=./runs)
- **ALMA (Atacama Large Millimeter/submillimeter Array)** (Source instrument for the protoplanetary disk FITS images used as one of the science demonstrations)

## Examples

```
python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt
```

## Evaluation signals

- pipeline_02_inference.py run with export_to_csv = true produces a CSV containing embedding vectors and per-object metadata
- TensorBoard launched with tensorboard --logdir=./runs successfully loads and displays the embedding projector with per-object metadata
- Embedding vectors align one-to-one with the rows of the input FITS filelist
- Inspecting the projector shows embeddings that capture morphological differences and similarities across the disk sample, as demonstrated in the paper

## Limitations

- Project developed and tested for Python 3.12; anything below Python 3.9 will definitely not work
- Filelist convention filters to FITS files smaller than 10 MB
- If using a truncated ResNet18 (last_layer, e.g. layer2), inference must reuse the same last_layer setting as the training run
- BYOL requires calling update_moving_average() after every optimization step when using the BYOL class directly

## Evidence

- [abstract] We present astromorph, a Python package that implements the BYOL method in a way tailored for astronomical imaging.: "We present astromorph , a Python package that implements the BYOL method in a way tailored for astronomical imaging."
- [abstract] In both cases embeddings capture morphological differences and similarities across large samples.: "In both cases we demonstrate how astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples."
- [methods] find command for FITS filelist of files under 10 MB.: "$ find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [readme] export_to_csv option writes embeddings and metadata to a CSV file.: "export_to_csv = true # Export embeddings and metadata to a CSV file"
- [readme] TensorBoard visualization command.: "tensorboard --logdir=./runs"
