---
name: neural-ode-surrogate-training
description: Use when you have a raw 3D-PDR dataset (HDF5 or simulations.tgz) from
  one of the v1/v2/v3 Zenodo deposits and need to train an emulator that reproduces
  chemical abundances, temperatures, and column density maps faster than the chemical
  solver.
when_to_use_negative:
- You only need inference or evaluation with an already-trained surrogate — use the
  inference workflow (inference.py with --dataset_path, --model_dir, --weights_file)
  instead of retraining
- Your chemistry model is not 3D-PDR photodissociation-region chemistry (densities,
  cosmic ray ionisation rates, Av, radiation field as parameters) — the LANODE was
  trained specifically on 3D-PDR output
- You want full end-to-end dataset preparation of all versions at once — scripts/data/workflow_setup_all.sh
  covers all three datasets rather than a per-version training run
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Runtime for the header-processing scripts and the train.py training entry
    point invoked with a configs/<version>/<config>.yaml file
  repo: https://github.com/uclchem/neuralpdr
- name: HDF5
  role: Storage format for both the raw Zenodo datasets and the training-ready processed
    files under data/processed/
- name: 3D-PDR
  role: 'Origin of the training data: the photodissociation-region code whose cooling,
    heating and chemistry along lines of sight (changing Av) the LANODE surrogate
    emulates'
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:57d3e3512cb7d92696013cc3d700824b6799abd2771180018d23895de53f5750
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# neural-ode-surrogate-training

## Summary

Trains Latent Augmented Neural ODE surrogate models to emulate the 3D-PDR photodissociation-region chemistry on three datasets of increasing complexity (v1 uniform 1D clouds, v2 varying-density clouds, v3 3D giant molecular cloud), producing checkpoints that enable fast GPU inference of chemical abundances and temperatures.

## When to use

Use when you have a raw 3D-PDR dataset (HDF5 or simulations.tgz) from one of the v1/v2/v3 Zenodo deposits and need to train an emulator that reproduces chemical abundances, temperatures, and column density maps faster than the chemical solver.

## When NOT to use

- You only need inference or evaluation with an already-trained surrogate — use the inference workflow (inference.py with --dataset_path, --model_dir, --weights_file) instead of retraining
- Your chemistry model is not 3D-PDR photodissociation-region chemistry (densities, cosmic ray ionisation rates, Av, radiation field as parameters) — the LANODE was trained specifically on 3D-PDR output
- You want full end-to-end dataset preparation of all versions at once — scripts/data/workflow_setup_all.sh covers all three datasets rather than a per-version training run

## Inputs

- data/zenodo/v1/3pdr_dataset_8192.h5 (v1 uniform 1D cloud dataset, Zenodo 10.5281/zenodo.13711173)
- data/zenodo/v2/simulations.tgz (v2 varying-density cloud dataset, Zenodo 10.5281/zenodo.7310832)
- data/zenodo/v3/3dpdr_dataset_v3.h5 (v3 3D giant molecular cloud dataset, Zenodo 10.5281/zenodo.15688233)
- Configuration YAML from configs/v1/, configs/v2/, or configs/v3/ specifying run output directory
- Optionally, test datasets in data/test/ for quick validation without downloading the full Zenodo datasets

## Outputs

- Processed training-ready HDF5 file in data/processed/ (3dpdr_dataset_v1.h5, 3dpdr_dataset_v2.h5, or 3dpdr_dataset_v3.h5)
- Model checkpoints and logs written to the directory specified in the configuration file
- Best checkpoint weights file best_weights.pkl in the run directory

## How to apply

Download the appropriate Zenodo dataset (v1: 3pdr_dataset_8192.h5 at doi 10.5281/zenodo.13711173; v2: simulations.tgz at doi 10.5281/zenodo.7310832; v3: 3dpdr_dataset_v3.h5 at doi 10.5281/zenodo.15688233) and place it under data/zenodo/v1|v2|v3/. Run the version-specific header processing script (v1_add_headers.py, workflow_v2.sh, or workflow_v3.sh) to produce data/processed/3dpdr_dataset_v1|v2|v3.h5. Select a YAML configuration from configs/v1/, configs/v2/, or configs/v3/ and set the run output directory inside it, then launch training via train.py; the LANODE takes constant densities and cosmic ray ionisation rates plus visual extinction and radiation field as parameters, and chemical abundances and temperatures as features. Let training run to completion and verify the best checkpoint best_weights.pkl is written to the configured run directory; the learned surrogate should substitute the 3D-PDR chemical solver with speedup and reproduce the original observable column density maps.

## Related tools

- **Python** (Runtime for the header-processing scripts and the train.py training entry point invoked with a configs/<version>/<config>.yaml file) — https://github.com/uclchem/neuralpdr
- **HDF5** (Storage format for both the raw Zenodo datasets and the training-ready processed files under data/processed/)
- **3D-PDR** (Origin of the training data: the photodissociation-region code whose cooling, heating and chemistry along lines of sight (changing Av) the LANODE surrogate emulates)

## Examples

```
python src/neuralpdr/train.py configs/v2/base.yaml
```

## Evaluation signals

- Training script runs to completion on the deposited dataset without error
- best_weights.pkl exists in the configured run directory and is loadable as a serialized weights file
- Model checkpoints and logs are written to the directory specified in the configuration YAML
- Inference with the trained checkpoint reproduces the original observable column density maps with a mean_relative_error within the tolerance reported in the paper
- Surrogate inference delivers a speedup_factor > 1 over the original 3D-PDR chemical solver on identical input parameter sets

## Limitations

- Neptune callback/logging integration is deprecated as of March 2026; its call will stop functioning and is slated for removal from the codebase
- The discussion/changelog contains no additional content beyond a changelog-absence notice, so post-publication fixes or caveats are not documented here
- Only example configurations per dataset version (configs/v1/, configs/v2/, configs/v3/) are provided; the output run directory must be set manually inside the chosen YAML

## Evidence

- [abstract] Surrogate training on three datasets of increasing complexity: "We train these surrogate architectures on three datasets of increasing physical complexity, with the last dataset derived directly from a three-dimensional simulation of a molecular cloud"
- [intro] LANODE input parameterization: "Taking densities (constant), cosmic ray ionisations (constant), visual extinctions, radiation field as additional parameters and the chemical abundances and temperatures as normal features."
- [readme] Training command and config layout: "Train the model using a configuration file from `configs/`. Example configurations are provided for each dataset version in `configs/v1/`, `configs/v2/`, and `configs/v3/`."
- [readme] Dataset placement and processing scripts: "Download the datasets from Zenodo (links below) and place them in `data/zenodo/` following this structure"
- [methods] v1 per-version processing entry point: "python scripts/data/v1_add_headers.py   # for v1"
