---
name: astronomical-multi-channel-image-ingestion
description: Use when input is a directory of FITS images (single-channel 2D images
  or multi-channel spectral cubes) that must feed a PyTorch pipeline such as BYOL
  pretraining.
when_to_use_negative:
- Input is already a feature table or embedding CSV rather than raw FITS imagery
- Data is a single large FITS file with an accompanying binary mask file rather than
  a set of per-object FITS files
- Pixel-value scaling other than min-max is required (e.g. background-subtracted or
  calibrated surface-brightness units that must be preserved)
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: astromorph
  role: Provides FitsFilelistDataset for opening single- and multi-channel FITS files
    and MinMaxNorm for pixel normalization, exported from the package namespace alongside
    BYOL, ByolTrainer, and AstroMorphologyModel
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: FitsFilelistDataset is implemented as a PyTorch-compatible Dataset consumable
    by torch DataLoader instances for training and test sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:7057aff958097934f9319e42b824c21bc087c1119fabd49a246681240d62a02e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - astromorph
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# astronomical-multi-channel-image-ingestion

## Summary

Ingest FITS files of varying dimensions and channel counts into PyTorch training via astromorph's FitsFilelistDataset, with MinMaxNorm scaling pixel values to a common range before tensors are returned.

## When to use

Use when input is a directory of FITS images (single-channel 2D images or multi-channel spectral cubes) that must feed a PyTorch pipeline such as BYOL pretraining.

## When NOT to use

- Input is already a feature table or embedding CSV rather than raw FITS imagery
- Data is a single large FITS file with an accompanying binary mask file rather than a set of per-object FITS files
- Pixel-value scaling other than min-max is required (e.g. background-subtracted or calibrated surface-brightness units that must be preserved)

## Inputs

- FITS file paths listed in a filelist file (e.g. data/inputfiles.txt generated with find)
- Single-channel 2D FITS images
- Multi-channel FITS spectral cubes / stacked images
- stacksize setting (e.g. 3) for RGB-style input when the network expects 3-channel images

## Outputs

- Normalized image tensors (pixel values scaled by MinMaxNorm to a common range)
- PyTorch DataLoader-ready dataset for BYOL training or inference

## How to apply

Build a filelist of FITS paths using the `find` command (e.g. files under 10 MB matching '**.fits' written to data/inputfiles.txt). Implement or instantiate FitsFilelistDataset, a PyTorch-compatible Dataset that opens each FITS file and supports both single-channel images and multi-channel stacked images via the stacksize option (e.g. stacksize = 3 for RGB-style input, so a 3-channel network can consume single-channel images). Apply the MinMaxNorm transform so pixel values from different instruments and resolutions (e.g. ALMA protoplanetary-disk images, Spitzer/Herschel infrared dark-cloud images) are scaled to a common range before being returned. Wrap the dataset in PyTorch DataLoader instances for training or inference with ByolTrainer or the pipeline scripts.

## Related tools

- **astromorph** (Provides FitsFilelistDataset for opening single- and multi-channel FITS files and MinMaxNorm for pixel normalization, exported from the package namespace alongside BYOL, ByolTrainer, and AstroMorphologyModel) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (FitsFilelistDataset is implemented as a PyTorch-compatible Dataset consumable by torch DataLoader instances for training and test sets)

## Examples

```
find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt
```

## Evaluation signals

- astromorph imports the exported names FitsFilelistDataset and MinMaxNorm without error
- A small script instantiating the dataset on a sample filelist runs without error and returns normalized image tensors
- Single-channel FITS files are stacked to the configured stacksize (e.g. 3 channels for RGB-style input) matching the network's expected channel count
- Normalized tensors fall in the common min-max range across images of differing original dimensions and resolutions

## Limitations

- Python 3.12 is the developed-for version; anything below Python 3.9 will definitely not work
- Filelist ingestion expects individual FITS files (e.g. <10 MB in the documented example), not single large multi-object FITS files with masks
- MinMaxNorm scales to a common range but does not address other preprocessing needs such as background subtraction or resampling to a common resolution

## Evidence

- [methods] astromorph provides a FilelistDataset component for easy handling of sets of FITS files, serving as the data-ingestion mechanism for the BYOL pipeline.: "a FilelistDataset for easy handling of sets of FITS files;"
- [readme] The stacksize option lets single-channel images feed a 3-channel RGB-style network.: "# If the network expects 3-channel RGB images, but you have single-channel images
[data_settings]
stacksize = 3"
- [readme] Filelists of FITS paths are generated with the find command.: "find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [readme] PyTorch DataLoaders are the intended consumption interface for the dataset in ByolTrainer training.: "For training, one only needs two PyTorch `DataLoader` instances for the training- and test-set."
