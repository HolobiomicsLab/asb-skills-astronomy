---
name: infrared-dark-cloud-image-analysis
description: Use when you have a sample of FITS images (e.g. Spitzer/Herschel infrared
  dark clouds) and need label-free morphological embeddings for clustering, anomaly
  detection, or similarity-based exploration.
when_to_use_negative:
- Input is already an embedding table or feature table rather than raw FITS images
  — inference and BYOL pretraining are unnecessary.
- The task requires labelled classification with ground-truth morphological labels
  — astromorph is a label-free BYOL pipeline, not a supervised classifier.
- Python version below 3.9 — the project targets Python 3.12 and anything below 3.9
  definitely does not work.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3321
tools:
- name: astromorph
  role: Self-supervised BYOL pipeline providing pipeline_02_inference.py for embedding
    FITS images and exporting embeddings to CSV
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Backend implementing the BYOL class and the AstroMorphologyModel network used
    in inference
- name: Spitzer
  role: Instrument providing infrared dark cloud imaging that constitutes one of the
    two test-case datasets
- name: Herschel
  role: Instrument providing infrared dark cloud imaging that constitutes one of the
    two test-case datasets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:b2df3f87c695914be296553a8c3814badd93d79e7036cdd3b86638be3cd5fedf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Herschel
  - PyTorch
  - Spitzer
  - astromorph
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3321
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# infrared-dark-cloud-image-analysis

## Summary

Applies the astromorph self-supervised (BYOL) pipeline to infrared dark cloud images observed with Spitzer and Herschel, producing CSV embeddings that capture morphological differences and similarities across the sample without manual labels.

## When to use

Use when you have a sample of FITS images (e.g. Spitzer/Herschel infrared dark clouds) and need label-free morphological embeddings for clustering, anomaly detection, or similarity-based exploration.

## When NOT to use

- Input is already an embedding table or feature table rather than raw FITS images — inference and BYOL pretraining are unnecessary.
- The task requires labelled classification with ground-truth morphological labels — astromorph is a label-free BYOL pipeline, not a supervised classifier.
- Python version below 3.9 — the project targets Python 3.12 and anything below 3.9 definitely does not work.

## Inputs

- FITS images of infrared dark clouds observed with Spitzer and/or Herschel
- Filelist of FITS file paths (e.g. data/inputfiles.txt) produced with the find command
- TOML config file referencing the filelist (datafile) and a pretrained checkpoint (trained_network_name, e.g. saved_models/newly_trained_network.pt)

## Outputs

- CSV file of image embeddings plus per-image metadata for the infrared dark cloud sample (export_to_csv = true)
- TensorBoard visualization of the embeddings (./runs directory)

## How to apply

Build a filelist of FITS files with the find command (e.g. filtering on file size, such as files smaller than 10 MB, and the .fits extension) and reference it via the datafile keyword of a TOML config file. Point the config at the pretrained checkpoint using trained_network_name (e.g. saved_models/newly_trained_network.pt) and set export_to_csv = true; non-relevant options such as epochs are ignored, so the training config can be reused. Run the inference script pipeline_02_inference.py with the config (or pass options via flags like -d, -m, -n; command-line options overrule the config). Collect the exported CSV of embeddings plus metadata, and optionally inspect with TensorBoard (tensorboard --logdir=./runs). Judge success by verifying the CSV exists and that the embedding table shows scientifically meaningful separation reflecting morphological differences across the infrared dark cloud sample.

## Related tools

- **astromorph** (Self-supervised BYOL pipeline providing pipeline_02_inference.py for embedding FITS images and exporting embeddings to CSV) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Backend implementing the BYOL class and the AstroMorphologyModel network used in inference)
- **Spitzer** (Instrument providing infrared dark cloud imaging that constitutes one of the two test-case datasets)
- **Herschel** (Instrument providing infrared dark cloud imaging that constitutes one of the two test-case datasets)

## Examples

```
python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt; tensorboard --logdir=./runs
```

## Evaluation signals

- The CSV embeddings file exists and contains embeddings plus metadata for the infrared dark cloud sample.
- The embedding table shows scientifically meaningful separation reflecting morphological differences across the sample.
- Similar infrared dark clouds are mapped to similar embeddings, capturing morphological similarities (inspectable via TensorBoard).
- The inference run resolves the trained_network_name checkpoint (e.g. saved_models/newly_trained_network.pt) without config/flag conflicts, since command-line options overrule the config.

## Limitations

- The pipeline was developed for Python 3.12; anything below Python 3.9 will definitely not work.
- FITS files larger than the size filter used when building the filelist (e.g. 10 MB in the example) are excluded, so very large images must be handled separately.
- Meaningful embeddings require a pretrained checkpoint (e.g. from pipeline_01_training.py); inference alone does not train the network.

## Evidence

- [other] For the infrared dark cloud (Spitzer and Herschel) science case, astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across the sample.: [evidence span withheld — anchor status: not_found]
- [readme] Find the filenames and store them in data/inputfiles.txt: "find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [readme] Export embeddings and metadata to a CSV file: "export_to_csv = true # Export embeddings and metadata to a CSV file"
- [readme] Command-line options overrule config settings: "The options given in the command line will overrule the settings specified in the config file."
- [abstract] In both test cases embeddings capture morphological differences and similarities across large samples: "In both cases we demonstrate how astromorph produces scientifically meaningful embeddings that capture morphological differences and similarities across large samples."
