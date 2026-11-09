---
name: tensorboard-embedding-visualization
description: Use when astromorph's inference script (pipeline_02_inference.py) has
  produced embeddings for a set of FITS images (e.g. ALMA protoplanetary disk or Spitzer/Herschel
  infrared dark cloud images) and you need to inspect morphological structure via
  TensorBoard.
when_to_use_negative:
- Quantitative downstream analysis (clustering, anomaly detection, similarity search)
  — TensorBoard is for visual inspection; run those methods directly on the exported
  CSV embeddings instead
- Embeddings that did not come from astromorph's inference pipeline / no ./runs directory
  exists
- The trained network's last_layer truncation or stacksize settings do not match the
  training run — embeddings would be invalid and the visualisation misleading
edam_operation: http://edamontology.org/operation_0337
tools:
- name: astromorph
  role: Provides the configurable inference script (pipeline_02_inference.py), FitsFilelistDataset,
    and AstroMorphologyModel encoder whose embeddings are visualised with TensorBoard
  repo: github.com/onsala-space-observatory/astromorph
- name: TensorBoard
  role: Loads the runs directory produced by inference to render the interactive embedding
    visualisation
- name: PyTorch
  role: Framework underlying the BYOL implementation and AstroMorphologyModel encoder
    that generate the visualised embeddings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:4ffa2a8b1b8444e12d8726682e4a07ca9bc4682b03f58d82f32d0fcd1cb1c0a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA (Atacama Large Millimeter/submillimeter Array)
  - PyTorch
  - Python
  - TensorBoard
  - astromorph
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tensorboard-embedding-visualization

## Summary

Interactive visual inspection of image embeddings produced by the astromorph inference pipeline (BYOL/ResNet18-based AstroMorphologyModel) using TensorBoard, alongside the CSV export of embeddings and metadata, to confirm that embeddings capture morphological differences and similarities across astronomical image samples.

## When to use

Use when astromorph's inference script (pipeline_02_inference.py) has produced embeddings for a set of FITS images (e.g. ALMA protoplanetary disk or Spitzer/Herschel infrared dark cloud images) and you need to inspect morphological structure via TensorBoard.

## When NOT to use

- Quantitative downstream analysis (clustering, anomaly detection, similarity search) — TensorBoard is for visual inspection; run those methods directly on the exported CSV embeddings instead
- Embeddings that did not come from astromorph's inference pipeline / no ./runs directory exists
- The trained network's last_layer truncation or stacksize settings do not match the training run — embeddings would be invalid and the visualisation misleading

## Inputs

- runs/ output directory written by astromorph's inference script (pipeline_02_inference.py)
- CSV file of embedding vectors and associated metadata (export_to_csv = true)
- Trained network weights (.pt) from a BYOL training run on FITS images, e.g. saved_models/newly_trained_network.pt
- Filelist of FITS files (e.g. data/inputfiles.txt from the find command, files under 10 MB)

## Outputs

- Interactive TensorBoard embedding visualisation of the inference-run embeddings
- Sanity-checkable CSV of embedding vectors plus per-source metadata

## How to apply

First run astromorph's configurable inference script (astromorph/src/pipeline_02_inference.py) with a TOML config specifying the datafile (a filelist of FITS files under 10 MB generated with the find command), the trained_network_name pointing to the saved .pt model, and export_to_csv = true; the FitsFilelistDataset loads the images and the trained network — with any last_layer truncation (e.g. layer2) matching the training run — computes the embeddings. Run the script from the main folder of the astromorph repository so output is written under ./runs. Then launch tensorboard --logdir=./runs to view the interactive embedding visualisation and confirm that disks/clouds with similar morphology lie near each other and morphologically distinct objects separate. Judge success by the CSV containing embedding vectors plus metadata and by TensorBoard loading the runs directory without error, consistent with the paper's demonstration on ALMA protoplanetary disk and infrared dark cloud datasets.

## Related tools

- **astromorph** (Provides the configurable inference script (pipeline_02_inference.py), FitsFilelistDataset, and AstroMorphologyModel encoder whose embeddings are visualised with TensorBoard) — github.com/onsala-space-observatory/astromorph
- **TensorBoard** (Loads the runs directory produced by inference to render the interactive embedding visualisation)
- **PyTorch** (Framework underlying the BYOL implementation and AstroMorphologyModel encoder that generate the visualised embeddings)

## Examples

```
tensorboard --logdir=./runs
```

## Evaluation signals

- CSV file with embedding vectors and metadata is produced after inference (export_to_csv = true)
- tensorboard --logdir=./runs starts and loads the embeddings without error
- Similar-morphology objects (e.g. ALMA protoplanetary disks with comparable ring/gap structure) appear close in the embedding space; morphologically distinct objects separate
- Embedding dimensionality matches the training configuration (e.g. representation_size = 128, last_layer truncation identical to training)

## Limitations

- Visual inspection only — TensorBoard does not perform clustering or anomaly detection; those must be run separately on the exported embeddings
- Requires the inference run to have been executed from the main folder of the astromorph repository so that ./runs is populated
- Any last_layer truncation of the ResNet18 network must match the training run, otherwise the embeddings visualised are not comparable

## Evidence

- [readme] To view a visualisation of the embeddings after inference, use TensorBoard: "To view a visualisation of the embeddings after inference, use TensorBoard"
- [readme] export_to_csv = true # Export embeddings and metadata to a CSV file: "export_to_csv = true # Export embeddings and metadata to a CSV file"
- [abstract] astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples: "astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples"
- [abstract] the framework will enable key tasks such as clustering, anomaly detection, and similarity-based exploration: "the framework will enable key tasks such as clustering, anomaly detection, and similarity-based exploration"
