---
name: fits-image-embedding-inference
description: Use when you have a trained astromorph checkpoint and a set of FITS images
  (e.g. Spitzer/Herschel infrared dark clouds) and need label-free morphological embeddings
  for each image.
when_to_use_negative:
- No pretrained checkpoint exists yet — run the BYOL self-supervised training script
  (pipeline_01_training.py) first
- Input is not FITS imagery (e.g. already-extracted feature tables or catalog data)
  — astromorph operates on image files via FitsFilelistDataset
- The goal is labelled classification with manually annotated data — astromorph is
  explicitly a label-free, self-supervised approach
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: astromorph
  role: Provides the configurable inference script (pipeline_02_inference.py) that
    runs the pretrained BYOL network on FITS filelists and exports embeddings to CSV
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Underlying framework whose classes implement the BYOL model loaded from the
    trained checkpoint during inference
  repo: github.com/onsala-space-observatory/astromorph
- name: TensorBoard
  role: Used to visualize the exported embeddings after inference via tensorboard
    --logdir=./runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:cc061edfbd7aaf5257de1a675fc70584de0a198313dbd8c18d935019bb427ea5
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
yes_rate: 0.75
n_claims: 8
---

# fits-image-embedding-inference

## Summary

Run a pretrained astromorph BYOL network over a filelist of FITS astronomical images (e.g. infrared dark clouds from Spitzer/Herschel) to export self-supervised embeddings plus metadata to CSV for downstream clustering, anomaly detection, or similarity exploration.

## When to use

Use when you have a trained astromorph checkpoint and a set of FITS images (e.g. Spitzer/Herschel infrared dark clouds) and need label-free morphological embeddings for each image.

## When NOT to use

- No pretrained checkpoint exists yet — run the BYOL self-supervised training script (pipeline_01_training.py) first
- Input is not FITS imagery (e.g. already-extracted feature tables or catalog data) — astromorph operates on image files via FitsFilelistDataset
- The goal is labelled classification with manually annotated data — astromorph is explicitly a label-free, self-supervised approach

## Inputs

- Filelist of FITS image paths (e.g. data/inputfiles.txt) built with the find command, filtering on file size and .fits extension
- TOML config file with datafile, trained_network_name, and export_to_csv keywords (training config can be reused; non-relevant options are ignored)
- Pretrained network checkpoint file (e.g. saved_models/newly_trained_network.pt)

## Outputs

- CSV file containing embeddings and metadata for each FITS image in the sample
- TensorBoard visualization of the embeddings under ./runs

## How to apply

Build a filelist of FITS files with the `find` command (e.g. filtering on `-size -10M` and the `**.fits` extension) and reference it via the `datafile` keyword of a TOML config file. Point the config at the pretrained checkpoint with `trained_network_name` (e.g. `saved_models/newly_trained_network.pt`) and set `export_to_csv = true` to export embeddings and metadata to a CSV file. Run the inference script `pipeline_02_inference.py` with the config (`-c`); command-line flags such as `-d <data-file> -n <trained-network-file>` may be combined with the config file, and command-line options overrule config-file settings. Inspect the resulting embeddings with TensorBoard (`tensorboard --logdir=./runs`). Non-relevant config keywords like `epochs` are ignored during inference, so the training config can be reused.

## Related tools

- **astromorph** (Provides the configurable inference script (pipeline_02_inference.py) that runs the pretrained BYOL network on FITS filelists and exports embeddings to CSV) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Underlying framework whose classes implement the BYOL model loaded from the trained checkpoint during inference) — github.com/onsala-space-observatory/astromorph
- **TensorBoard** (Used to visualize the exported embeddings after inference via tensorboard --logdir=./runs)

## Examples

```
python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt
```

## Evaluation signals

- The exported CSV file exists and contains an embedding column/table plus metadata for every image in the input filelist
- The embedding table shows scientifically meaningful separation reflecting morphological differences across the infrared dark cloud sample
- Similar images (similar morphology) receive nearby embeddings, as inspectable in the TensorBoard projection (./runs)
- The inference run completes without errors and the trained_network_name path resolves to an existing .pt checkpoint

## Limitations

- Requires a pretrained network; without a checkpoint from pipeline_01_training.py no embeddings can be produced
- Python version matters: the project targets Python 3.12 and anything below Python 3.9 will definitely not work
- FITS files larger than the size filter used in the find command (e.g. -10M) are excluded from the filelist; masked-data options may not be available in this version of the README

## Evidence

- [methods] inference with config and CSV export: "export_to_csv = true # Export embeddings and metadata to a CSV file"
- [readme] command-line overrule of config: "It is even possible to use a combination of config file and command line options.
The options given in the command line will overrule the settings specified in the config file."
- [abstract] meaningful embeddings finding: "In both cases we demonstrate how astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples."
- [methods] filelist construction: "$ find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [readme] TensorBoard visualization: "To view a visualisation of the embeddings after inference, use TensorBoard with:

tensorboard --logdir=./runs"
