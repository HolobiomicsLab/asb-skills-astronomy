---
name: embedding-csv-export
description: Use when you have FITS astronomical images and a pretrained astromorph
  checkpoint, and need a machine-readable CSV table of image embeddings plus metadata
  for clustering, similarity exploration, or morphological comparison.
when_to_use_negative:
- No trained network checkpoint is available yet — run pipeline_01_training.py first
  to obtain a .pt file before inference.
- Data are not FITS image files but an already-computed feature or embedding table
  — inference is unnecessary.
- Very large FITS files (e.g. above the ~10 MB size filter used to build the filelist)
  that may exceed the pipeline's expected input size.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3489
- http://edamontology.org/topic_0006
tools:
- name: astromorph
  role: Provides pipeline_02_inference.py, the configurable inference script that
    runs a trained BYOL network on the FITS filelist and exports embeddings plus metadata
    to CSV
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Underlying framework implementing the BYOL class and ByolTrainer used to produce
    the embeddings exported at inference time
- name: TensorBoard
  role: Optional post-export visualisation of the resulting embeddings via tensorboard
    --logdir=./runs
- name: Spitzer
  role: Infrared dark cloud imaging instrument providing one source of the FITS input
    sample
- name: Herschel
  role: Infrared dark cloud imaging instrument providing one source of the FITS input
    sample
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:a18e5ebb26ac4cbf04053b81083094797cc82007d74eb31c4327c8ea1b23cdf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Herschel
  - PyTorch
  - Spitzer
  - TensorBoard
  - astromorph
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3489
  - http://edamontology.org/topic_0006
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# embedding-csv-export

## Summary

Export FITS image embeddings and metadata to CSV from a trained astromorph (BYOL) network via the inference pipeline, as demonstrated on the infrared dark cloud sample observed with Spitzer and Herschel.

## When to use

Use when you have FITS astronomical images and a pretrained astromorph checkpoint, and need a machine-readable CSV table of image embeddings plus metadata for clustering, similarity exploration, or morphological comparison.

## When NOT to use

- No trained network checkpoint is available yet — run pipeline_01_training.py first to obtain a .pt file before inference.
- Data are not FITS image files but an already-computed feature or embedding table — inference is unnecessary.
- Very large FITS files (e.g. above the ~10 MB size filter used to build the filelist) that may exceed the pipeline's expected input size.

## Inputs

- FITS astronomical images (e.g. Spitzer/Herschel infrared dark cloud images), referenced via a find-generated filelist (data/inputfiles.txt)
- TOML config file with datafile, trained_network_name, and export_to_csv = true
- Trained network checkpoint (.pt file, e.g. saved_models/newly_trained_network.pt)

## Outputs

- CSV file containing embeddings and metadata for each image in the sample
- TensorBoard logs under ./runs for embedding visualisation

## How to apply

Build a filelist of the FITS images with the find command (e.g. filtering on file size such as -size -10M and the **.fits name pattern) and reference it via the datafile keyword of a TOML config. Point the config at the pretrained checkpoint with the trained_network_name keyword (e.g. saved_models/newly_trained_network.pt) and set export_to_csv = true. Run the configurable inference script pipeline_02_inference.py with the config file; command-line options may be combined with or overrule the config file (e.g. -n saved_models/newly_trained_network.pt). The pipeline runs the trained network over each image in the filelist and writes the embeddings and metadata to a CSV file; for single-channel images fed to a 3-channel network, set [data_settings] stacksize = 3. Optionally inspect the exported embeddings with TensorBoard (tensorboard --logdir=./runs).

## Related tools

- **astromorph** (Provides pipeline_02_inference.py, the configurable inference script that runs a trained BYOL network on the FITS filelist and exports embeddings plus metadata to CSV) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Underlying framework implementing the BYOL class and ByolTrainer used to produce the embeddings exported at inference time)
- **TensorBoard** (Optional post-export visualisation of the resulting embeddings via tensorboard --logdir=./runs)
- **Spitzer** (Infrared dark cloud imaging instrument providing one source of the FITS input sample)
- **Herschel** (Infrared dark cloud imaging instrument providing one source of the FITS input sample)

## Examples

```
python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt
```

## Evaluation signals

- The CSV embeddings file exists and contains embeddings plus metadata for every image in the filelist.
- Embedding values are finite floats with dimensionality matching the configured representation_size (default 128).
- The embedding table shows scientifically meaningful separation reflecting morphological differences across the sample (e.g. between infrared dark cloud morphologies).
- TensorBoard visualisation under ./runs renders the embeddings with sensible groupings.
- Inference succeeds with the checkpoint path matching the network_name/last_layer settings used at training time (e.g. saved_models/newly_trained_network.pt).

## Limitations

- The config file must supply trained_network_name; reusing a training config without it (non-relevant options like epochs are ignored) is required for inference.
- Command-line options overrule the config file, so conflicting -n/-d flags can silently change the run.
- If the network expects 3-channel RGB images but inputs are single-channel, [data_settings] stacksize = 3 must be set.
- The filelist approach filters files by size (e.g. -size -10M); larger files are silently excluded from the embedding export.

## Evidence

- [other] For the infrared dark cloud (Spitzer and Herschel) science case, astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across the sample.: [evidence span withheld — anchor status: not_found]
- [readme] export_to_csv = true # Export embeddings and metadata to a CSV file: "export_to_csv = true # Export embeddings and metadata to a CSV file"
- [methods] find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt: "find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [readme] a configurable inference script for easy inspection of the resulting embeddings: "a configurable inference script for easy inspection of the resulting embeddings"
- [readme] If the network expects 3-channel RGB images, but you have single-channel images: "If the network expects 3-channel RGB images, but you have single-channel images"
