---
name: toml-pipeline-configuration
description: Use when running astromorph's training or inference pipeline scripts
  and you need to declaratively set data input, network, and export options (e.g.
  embedding a filelist of FITS images and exporting embeddings to CSV) rather than
  editing script code.
when_to_use_negative:
- Inputs are single FITS files with a binary mask only — use the datafile/maskfile
  (or -d/-m command-line) route, though note the maskfile configuration is commented
  out in the README and still expects the same TOML structure.
- You need PyTorch-level flexibility beyond what the pipeline scripts expose (e.g.
  custom BYOL training loops) — use the BYOL or ByolTrainer classes directly instead.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: astromorph
  role: Package whose configurable training script (pipeline_01_training.py) and inference
    script (pipeline_02_inference.py) consume the TOML configuration file
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Underlying framework whose classes (BYOL, AstroMorphologyModel) the config
    options such as network_name and last_layer select parameters for
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:9c774b65b465b3a238afa5c953d03c45d21ddee5692170965db8646bbfc7e667
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA (Atacama Large Millimeter/submillimeter Array)
  - PyTorch
  - Python
  - astromorph
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# toml-pipeline-configuration

## Summary

Configure astromorph's self-supervised BYOL training and inference scripts through a TOML settings file (e.g. example_settings.toml), specifying the FITS filelist, trained network location, ResNet18 truncation, and CSV export options so the same configuration can drive pipeline_01_training.py and pipeline_02_inference.py.

## When to use

Use when running astromorph's training or inference pipeline scripts and you need to declaratively set data input, network, and export options (e.g. embedding a filelist of FITS images and exporting embeddings to CSV) rather than editing script code.

## When NOT to use

- Inputs are single FITS files with a binary mask only — use the datafile/maskfile (or -d/-m command-line) route, though note the maskfile configuration is commented out in the README and still expects the same TOML structure.
- You need PyTorch-level flexibility beyond what the pipeline scripts expose (e.g. custom BYOL training loops) — use the BYOL or ByolTrainer classes directly instead.

## Inputs

- TOML settings file (e.g. example_settings.toml)
- Filelist of FITS file paths (e.g. data/inputfiles.txt, generated with find -size -10M)
- Trained network checkpoint (e.g. saved_models/newly_trained_network.pt)

## Outputs

- Trained BYOL/AstroMorphologyModel network checkpoint (via pipeline_01_training.py)
- Embeddings and associated metadata exported to a CSV file (when export_to_csv = true)

## How to apply

Author a TOML file with a top-level 'datafile' keyword pointing to the filelist generated with find (e.g. data/inputfiles.txt), a 'network_name' such as n_layer_resnet, and an optional 'epochs' keyword (default 10). For inference on ALMA protoplanetary disk images or other FITS data, add 'trained_network_name' pointing to the saved model (e.g. saved_models/newly_trained_network.pt) and 'export_to_csv = true' so embeddings and metadata are written to CSV. To truncate ResNet18, set last_layer (e.g. 'layer2') inside the [network_settings] dictionary, matching whatever truncation was used during training; BYOL dimensions (representation_size, projection_size, projection_hidden_size) and core_limit or data_settings.stacksize (e.g. 3 for single-channel images) can also be set. Pass the file with -c/--config-file and run from the repository root; the non-relevant options (e.g. epochs during inference) are ignored, so the training config can be reused, and command-line flags such as -n override config settings.

## Related tools

- **astromorph** (Package whose configurable training script (pipeline_01_training.py) and inference script (pipeline_02_inference.py) consume the TOML configuration file) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Underlying framework whose classes (BYOL, AstroMorphologyModel) the config options such as network_name and last_layer select parameters for)

## Examples

```
python astromorph/src/pipeline_02_inference.py -c example_settings.toml -n saved_models/newly_trained_network.pt
```

## Evaluation signals

- The TOML file parses with the expected keywords: datafile, network_name, epochs, trained_network_name, export_to_csv, and [network_settings].last_layer match the trained run.
- pipeline_02_inference.py accepts the config via -c and produces the CSV file with embedding vectors and metadata when export_to_csv = true.
- TensorBoard can load the resulting runs (tensorboard --logdir=./runs) and display the embeddings.
- Command-line options (e.g. -n saved model path) correctly overrule the corresponding config-file settings.
- The last_layer truncation used at inference matches the truncation recorded in the training config, so embedding dimensions are consistent.

## Limitations

- Non-relevant options such as epochs are ignored at inference — a reused training config may look over-specified but this is by design.
- Anything below Python 3.9 will definitely not work and the project is developed for Python 3.12, so the surrounding environment constrains configuration of the venv and dependencies.
- The last_layer default for n_layer_resnet is the full ResNet18; mismatched truncation between training and inference configs will change the embedding output.
- Single-channel images fed to a network expecting 3-channel RGB require the data_settings.stacksize = 3 option.

## Evidence

- [readme] TOML settings file: "The settings of the training run are specified through a TOML file.
An example of such a file can be found in `example_settings.toml`."
- [methods] filelist keyword: "$ find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [readme] inference config: "trained_network_name = "saved_models/newly_trained_network.pt"

export_to_csv = true # Export embeddings and metadata to a CSV file"
- [readme] config reuse and overrides: "The non-relevant options (e.g. `epochs`) will be ignored, so you can reuse the config file from the training run."
- [readme] last_layer truncation: "To invoke this possibility, use the `last_layer` keyword inside the `network_settings` dictionary."
