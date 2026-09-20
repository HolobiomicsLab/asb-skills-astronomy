---
name: pdr-chemical-solver-runtime-comparison
description: 'Use when you need to quantify whether a neural surrogate can substitute
  a PDR chemistry code: identical density, cosmic-ray ionisation, visual extinction
  (Av) and radiation-field inputs are available and per-case speedup factors must
  be tabulated.'
when_to_use_negative:
- No trained surrogate checkpoint (model_dir with best_weights.pkl) exists — the comparison
  cannot run without a trained LANODE model
- The input dataset is a raw Zenodo archive (e.g. v2/simulations.tgz) rather than
  a processed HDF5 file in data/processed/ — run the header-processing workflow scripts
  first
- The goal is training or hyperparameter tuning of the surrogate rather than runtime
  benchmarking of inference vs the solver
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3474
tools:
- name: neuralpdr
  role: Provides the inference entry point (src/neuralpdr/inference.py), HDF5 dataset
    interface, and trained LANODE surrogate weights used for timed GPU inference
  repo: https://github.com/uclchem/neuralpdr
- name: 3D-PDR
  role: The original photodissociation-region chemical solver whose wall-clock runtime
    is measured as the baseline in the speedup calculation
- name: HDF5
  role: File format of the processed benchmark datasets (data/processed/3dpdr_dataset_v2.h5)
    loaded for surrogate inference
- name: Python
  role: Language in which the inference benchmark, timing, and speedup_results.csv
    tabulation are executed
  repo: https://github.com/uclchem/neuralpdr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:c6dada211a6ff9dcbf386f5128b9d4e122b06e55426bf15e40e07a13954592a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - Python
  - neuralpdr
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pdr-chemical-solver-runtime-comparison

## Summary

Benchmarks the wall-clock runtime of the 3D-PDR photodissociation-region chemical solver against Latent Augmented Neural ODE surrogate inference on identical input parameter sets, quantifying the computational speedup claimed for the NeuralPDR surrogate models.

## When to use

Use when you need to quantify whether a neural surrogate can substitute a PDR chemistry code: identical density, cosmic-ray ionisation, visual extinction (Av) and radiation-field inputs are available and per-case speedup factors must be tabulated.

## When NOT to use

- No trained surrogate checkpoint (model_dir with best_weights.pkl) exists — the comparison cannot run without a trained LANODE model
- The input dataset is a raw Zenodo archive (e.g. v2/simulations.tgz) rather than a processed HDF5 file in data/processed/ — run the header-processing workflow scripts first
- The goal is training or hyperparameter tuning of the surrogate rather than runtime benchmarking of inference vs the solver

## Inputs

- Processed HDF5 dataset file, e.g. data/processed/3dpdr_dataset_v2.h5 (or 3dpdr_dataset_v1.h5 / 3dpdr_dataset_v3.h5)
- Trained LANODE model artifacts: model directory under runs/ and weights file best_weights.pkl
- Input parameter sets for 3D-PDR: densities, cosmic ray ionisation rates, visual extinction, radiation field

## Outputs

- speedup_results.csv with per-case surrogate runtime, 3D-PDR solver runtime, and speedup_factor columns
- Reproduced observable column density maps from surrogate inference for fidelity comparison against the dataset originals

## How to apply

Load the processed HDF5 dataset (e.g. data/processed/3dpdr_dataset_v2.h5) via the neuralpdr HDF5 interface, then run surrogate inference with src/neuralpdr/inference.py using --model_dir pointing at the runs directory and --weights_file best_weights.pkl, recording GPU wall-clock time. Next, run the original 3D-PDR chemical solver on the identical parameter sets (densities, cosmic ray ionisation rates, visual extinction, radiation field) and record its wall-clock runtime. Compute the speedup factor as the ratio of 3D-PDR solver runtime to surrogate inference runtime for each benchmarked case, and write the per-case runtimes plus the speedup_factor column to speedup_results.csv. The benchmark is valid only if the surrogate also reproduces the original observable column density maps, so runtime comparison should be paired with a fidelity check against the dataset's observable maps.

## Related tools

- **neuralpdr** (Provides the inference entry point (src/neuralpdr/inference.py), HDF5 dataset interface, and trained LANODE surrogate weights used for timed GPU inference) — https://github.com/uclchem/neuralpdr
- **3D-PDR** (The original photodissociation-region chemical solver whose wall-clock runtime is measured as the baseline in the speedup calculation)
- **HDF5** (File format of the processed benchmark datasets (data/processed/3dpdr_dataset_v2.h5) loaded for surrogate inference)
- **Python** (Language in which the inference benchmark, timing, and speedup_results.csv tabulation are executed) — https://github.com/uclchem/neuralpdr

## Examples

```
python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v2.h5 --model_dir runs/model_name --weights_file best_weights.pkl
```

## Evaluation signals

- The benchmark script runs to completion and speedup_results.csv exists with a 'speedup_factor' field
- Every speedup_factor value exceeds 1, consistent with the paper's claim that surrogate models provide speedup
- Surrogate inference reproduces the original observable column density maps of the dataset while running faster
- Inference is executed against a processed HDF5 file (data/processed/3dpdr_dataset_*.h5) with dataset, model_dir, and weights_file all specified
- Wall-clock timing recorded on the GPU for the surrogate and on identical input parameter sets for 3D-PDR, so the ratio is apples-to-apples

## Limitations

- The surrogate was trained only on specific parameter regimes — densities (constant) and cosmic ray ionisations (constant) in v1, visual extinction and radiation field as parameters — so speedup claims hold within those emulated regimes
- Speedup alone is insufficient: the surrogate must also reproduce the original observable column density maps for the substitution to be scientifically valid
- Discussion section content was absent from the extracted article, so no extended discussion-level caveats are available beyond the abstract and intro claims
- The Neptune experiment-tracking callback in the codebase is deprecated as of March 2026 and its call will stop functioning

## Evidence

- [abstract] Surrogate models provide speedup over original solver while reproducing column density maps: "We show that these surrogate models can provide speedup and reproduce the original observable column density maps of the dataset."
- [readme] Inference invocation specifying dataset, model directory and weights file: "python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v2.h5 --model_dir runs/model_name --weights_file best_weights.pkl"
- [intro] Emulator input parameters for the chemistry: "Taking densities (constant), cosmic ray ionisations (constant), visual extinctions, radiation field as additional parameters and the chemical abundances and temperatures as normal features."
- [intro] 3D-PDR code role as the emulated solver: "In this project we emulate the Photodissociation Region (PDR) code 3D-PDR for 1D-dimensional uniform clouds (v1), clouds of varying density (v2) and a 3D model of a giant molecular cloud (v3)"
- [readme] Processed HDF5 outputs used as inference inputs: "This creates processed HDF5 files in `data/processed/`"
