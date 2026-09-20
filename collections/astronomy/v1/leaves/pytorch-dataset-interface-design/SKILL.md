---
name: pytorch-dataset-interface-design
description: Use when a PyTorch training or inference pipeline must consume a set
  of astronomical FITS files of varying dimensions/resolutions listed in a filelist,
  and normalized fixed-shape image tensors are needed as model input.
when_to_use_negative:
- Input is not a set of FITS files but already-extracted embeddings or a feature table
  from the inference/export step (export_to_csv output).
- Data is a single large FITS data file with a companion mask file rather than a filelist
  of many FITS files.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: astromorph
  role: Package whose namespace exposes FitsFilelistDataset and MinMaxNorm as the
    data-ingestion mechanism for the BYOL pipeline
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework providing the Dataset/nn.Module interfaces the custom FITS dataset
    must conform to
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:b4f9975ed4e5d0ab755ddcabab0e6f616b05ab7a07cdfbf95cc76967c9b18c07
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - astromorph
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pytorch-dataset-interface-design

## Summary

Design a PyTorch-compatible Dataset class that ingests lists of FITS files (single-channel images or multi-channel spectral stacks) and applies MinMaxNorm preprocessing, as used by the astromorph BYOL pipeline for data ingestion.

## When to use

Use when a PyTorch training or inference pipeline must consume a set of astronomical FITS files of varying dimensions/resolutions listed in a filelist, and normalized fixed-shape image tensors are needed as model input.

## When NOT to use

- Input is not a set of FITS files but already-extracted embeddings or a feature table from the inference/export step (export_to_csv output).
- Data is a single large FITS data file with a companion mask file rather than a filelist of many FITS files.

## Inputs

- filelist text file of FITS file paths (e.g. data/inputfiles.txt produced with the find command)
- FITS files (single-channel 2D images or multi-channel stacked/spectral cubes)
- stacksize setting from the TOML config ([data_settings] stacksize = 3)

## Outputs

- FitsFilelistDataset class (PyTorch Dataset over FITS files)
- MinMaxNorm normalization transform
- normalized image tensors consumable by torch DataLoader and the BYOL/ByolTrainer pipeline

## How to apply

Build a FitsFilelistDataset as a PyTorch Dataset that reads FITS file paths from a filelist file (e.g. data/inputfiles.txt generated with the find command), opens each FITS file, and handles both single-channel images and multi-channel (stacked) images via a stacksize option (e.g. stacksize = 3 when the network expects 3-channel RGB-style input but images are single-channel). Pair it with a MinMaxNorm transform so pixel values are scaled to a common range before tensors are returned, since BYOL training is sensitive to inconsistent input scales. Export both symbols from the astromorph package namespace alongside BYOL, ByolTrainer, and AstroMorphologyModel so they can be imported directly and fed to a torch DataLoader. Validate by confirming the module imports and that a small script instantiating the dataset on a sample filelist returns normalized image tensors without error.

## Related tools

- **astromorph** (Package whose namespace exposes FitsFilelistDataset and MinMaxNorm as the data-ingestion mechanism for the BYOL pipeline) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework providing the Dataset/nn.Module interfaces the custom FITS dataset must conform to)

## Examples

```
from astromorph import FitsFilelistDataset, MinMaxNorm; ds = FitsFilelistDataset("data/inputfiles.txt", stacksize=3, transform=MinMaxNorm())
```

## Evaluation signals

- astromorph imports both new names successfully, e.g. `from astromorph import FitsFilelistDataset, MinMaxNorm`
- Instantiating the dataset on a sample filelist runs without error and yields image tensors
- Returned tensors are normalized to the MinMaxNorm range (consistent scale across files of different resolutions)
- With stacksize = 3, single-channel FITS inputs are stacked into 3-channel tensors the network expects
- The dataset is usable as the data source for a torch DataLoader in a ByolTrainer training run

## Limitations

- Only Python 3.9+ is supported (developed for Python 3.12); anything below 3.9 will definitely not work.
- The documented find-based filelist example excludes FITS files of 10 MB or larger, so very large files need adjusted filtering.
- The README notes PyTorch-based classes offer flexibility but the BYOL class requires experience setting up a training routine (including calling update_moving_average() after every optimization step).

## Evidence

- [methods] a FilelistDataset for easy handling of sets of FITS files;: "a FilelistDataset for easy handling of sets of FITS files;"
- [methods] find-filelist generation: "$ find /full/path/to/datadirectory/ -type f -size -10M -name "**.fits" > data/inputfiles.txt"
- [readme] stacksize for single-channel images: "If the network expects 3-channel RGB images, but you have single-channel images [data_settings] stacksize = 3"
- [readme] BYOL class PyTorch implementation: "the `BYOL` class as a PyTorch implementation of the BYOL framework;"
