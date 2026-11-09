---
name: gpu-chemistry-emulator-speedup-quantification
description: Use when a trained neural-ODE chemistry surrogate (e.g. from the neuralpdr
  repo) must be shown to beat the original 3D-PDR solver in wall-clock time, and per-case
  speedup factors need to be tabulated.
when_to_use_negative:
- The surrogate model has not yet been trained or best_weights.pkl does not exist
  — validate surrogate accuracy (column density maps) before benchmarking speed
- The goal is emulator accuracy rather than runtime comparison — use inference outputs
  vs 3D-PDR outputs, not wall-clock timing
- The input is raw Zenodo data (e.g. v2/simulations.tgz) that has not been processed
  into data/processed/ HDF5 format
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: Runs the inference benchmark via src/neuralpdr/inference.py and computes/tabulates
    speedup factors
  repo: https://github.com/uclchem/neuralpdr
- name: 3D-PDR
  role: Original photodissociation-region chemical solver whose wall-clock runtime
    is the baseline of the speedup comparison
- name: HDF5
  role: File format of the processed datasets loaded for benchmark inference
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:d2dff3e970a7be8fb7b74ab22d518cd5cc48206c75c38a7aa71e796b0b6750e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - Python
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gpu-chemistry-emulator-speedup-quantification

## Summary

Benchmarks the wall-clock runtime of trained Latent Augmented Neural ODE surrogate chemistry inference on GPU against the original 3D-PDR chemical solver on identical input parameter sets, computing a per-case speedup_factor to substantiate the claim that surrogates effectively substitute the chemical code.

## When to use

Use when a trained neural-ODE chemistry surrogate (e.g. from the neuralpdr repo) must be shown to beat the original 3D-PDR solver in wall-clock time, and per-case speedup factors need to be tabulated.

## When NOT to use

- The surrogate model has not yet been trained or best_weights.pkl does not exist — validate surrogate accuracy (column density maps) before benchmarking speed
- The goal is emulator accuracy rather than runtime comparison — use inference outputs vs 3D-PDR outputs, not wall-clock timing
- The input is raw Zenodo data (e.g. v2/simulations.tgz) that has not been processed into data/processed/ HDF5 format

## Inputs

- Processed HDF5 dataset of PDR chemistry (e.g. data/processed/3dpdr_dataset_v2.h5)
- Trained LANODE surrogate model directory with best_weights.pkl (e.g. runs/model_name)
- Original 3D-PDR chemical solver executable, run on identical density, cosmic ray ionisation rate, visual extinction (Av), and radiation field parameter sets

## Outputs

- speedup_results.csv with per-case surrogate and 3D-PDR runtimes and a speedup_factor column
- Per-case wall-clock runtime measurements on GPU (surrogate) and CPU (3D-PDR)

## How to apply

Load the processed HDF5 dataset (e.g. data/processed/3dpdr_dataset_v2.h5) via the neuralpdr HDF5 interface and run GPU inference with the trained LANODE model, recording wall-clock time. Run the original 3D-PDR chemical solver on the identical input parameter sets (densities, cosmic ray ionisation rates, visual extinction, radiation field) and record its runtime. Compute speedup_factor as the ratio of 3D-PDR solver runtime to surrogate inference runtime for each benchmarked case. Tabulate per-case runtimes and the speedup_factor column into speedup_results.csv. A valid result requires speedup_factor > 1 while the surrogate reproduces the original observable column density maps of the dataset.

## Related tools

- **Python** (Runs the inference benchmark via src/neuralpdr/inference.py and computes/tabulates speedup factors) — https://github.com/uclchem/neuralpdr
- **3D-PDR** (Original photodissociation-region chemical solver whose wall-clock runtime is the baseline of the speedup comparison)
- **HDF5** (File format of the processed datasets loaded for benchmark inference)

## Examples

```
python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v2.h5 --model_dir runs/model_name --weights_file best_weights.pkl
```

## Evaluation signals

- The benchmark script runs to completion and speedup_results.csv exists with a 'speedup_factor' field
- Every speedup_factor value exceeds 1, consistent with the paper's claim that surrogate models provide speedup
- Surrogate inference reproduces the original observable column density maps of the dataset
- The same input parameter sets (densities, cosmic ray ionisation rates, Av, radiation field) are used for both surrogate and 3D-PDR runs
- Wall-clock timings are recorded for both the GPU inference and the original solver runs

## Limitations

- Speedup is demonstrated for emulating 3D-PDR on 1D uniform clouds (v1), varying-density clouds (v2) and a 3D giant molecular cloud (v3); the surrogate substitutes only the chemical solver, not full coupled hydrodynamics
- Surrogate inference requires a GPU; speedup claims do not transfer to CPU-only inference
- The Neptune logging callback is deprecated as of March 2026 and will stop functioning, which may affect benchmark logging setups

## Evidence

- [abstract] Surrogate models provide speedup while reproducing column density maps: "We show that these surrogate models can provide speedup and reproduce the original observable column density maps of the dataset."
- [abstract] Surrogates enable rapid GPU inference of chemistry: "This enables the rapid inference of the chemistry (on the GPU), allowing for the faster statistical inference of observations or increasing the resolution in hydrodynamical simulations"
- [readme] Inference CLI specifying dataset, model directory, and weights file: "python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v2.h5 --model_dir runs/model_name --weights_file best_weights.pkl"
- [intro] Surrogate input parameters and features: "Taking densities (constant), cosmic ray ionisations (constant), visual extinctions, radiation field as additional parameters and the chemical abundances and temperatures as normal features."
