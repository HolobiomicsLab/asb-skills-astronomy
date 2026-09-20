---
name: zenodo-dataset-ingestion
description: Use when a task requires retrieving raw training data from a Zenodo deposit
  (e.g. 10.5281/zenodo.7310832 or 10.5281/zenodo.15688233) and staging it in the repository
  layout before header-processing scripts run.
when_to_use_negative:
- Input is already a processed HDF5 file in data/processed/ (e.g. 3dpdr_dataset_v2.h5)
  — ingestion and header processing have already been done
- The goal is a quick smoke test of training/inference; use the bundled test datasets
  in data/test/ instead of downloading full Zenodo deposits
- The task targets a dataset version with no corresponding Zenodo DOI listed in the
  README (only v1, v2, v3 are documented)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3810
tools:
- name: Python
  role: Runs the v1 header-processing script (scripts/data/v1_add_headers.py) and
    downstream train/inference scripts that consume the ingested data
  repo: https://github.com/uclchem/neuralpdr
- name: HDF5
  role: File format of both the raw Zenodo deposits (e.g. 3dpdr_dataset_v3.h5) and
    the processed training-ready files in data/processed/
- name: 3D-PDR
  role: Source simulation code that generated the deposited PDR chemistry datasets
    being ingested
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1088/2632-2153/ade4ee
    title: 'NeuralPDR: neural differential equations as surrogate models for photodissociation
      regions'
  source_content_hash: sha256:2aab39983615da8a5364f88f21cfd449acc459cfff48b4fae74a9f4a0c1942fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/uclchem/neuralpdr
  tools:
  - 3D-PDR
  - HDF5
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3810
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# zenodo-dataset-ingestion

## Summary

Retrieves deposited simulation datasets from Zenodo DOIs and places them under data/zenodo/ with the version-specific filenames (3pdr_dataset_8192.h5, simulations.tgz, 3dpdr_dataset_v3.h5) expected by the NeuralPDR pipeline, so downstream header-processing scripts can produce training-ready HDF5 files.

## When to use

Use when a task requires retrieving raw training data from a Zenodo deposit (e.g. 10.5281/zenodo.7310832 or 10.5281/zenodo.15688233) and staging it in the repository layout before header-processing scripts run.

## When NOT to use

- Input is already a processed HDF5 file in data/processed/ (e.g. 3dpdr_dataset_v2.h5) — ingestion and header processing have already been done
- The goal is a quick smoke test of training/inference; use the bundled test datasets in data/test/ instead of downloading full Zenodo deposits
- The task targets a dataset version with no corresponding Zenodo DOI listed in the README (only v1, v2, v3 are documented)

## Inputs

- Zenodo deposit archive per version: v1 3pdr_dataset_8192.h5 (DOI 10.5281/zenodo.13711173), v2 simulations.tgz (DOI 10.5281/zenodo.7310832), v3 3dpdr_dataset_v3.h5 (DOI 10.5281/zenodo.15688233)
- Repository directory layout with data/zenodo/, scripts/data/, and data/processed/ paths

## Outputs

- data/zenodo/v1/3pdr_dataset_8192.h5, data/zenodo/v2/simulations.tgz, data/zenodo/v3/3dpdr_dataset_v3.h5 (raw staged files)
- data/processed/3dpdr_dataset_v1.h5, 3dpdr_dataset_v2.h5, 3dpdr_dataset_v3.h5 (training-ready HDF5 after header-processing scripts)

## How to apply

Resolve the Zenodo DOI for the target dataset version (v1: 10.5281/zenodo.13711173, v2: 10.5281/zenodo.7310832, v3: 10.5281/zenodo.15688233) and download the deposit archive. Place the file under data/zenodo/ following the exact expected structure: data/zenodo/v1/3pdr_dataset_8192.h5, data/zenodo/v2/simulations.tgz, or data/zenodo/v3/3dpdr_dataset_v3.h5. After placement, run the matching header-processing script — python scripts/data/v1_add_headers.py, ./scripts/data/workflow_v2.sh, or ./scripts/data/workflow_v3.sh — which converts the raw Zenodo files into processed HDF5 files in data/processed/ (3dpdr_dataset_v1.h5, 3dpdr_dataset_v2.h5, 3dpdr_dataset_v3.h5). For quick validation without large downloads, the repo's test datasets in data/test/ can substitute for the full Zenodo deposits. Correct ingestion is judged by the existence of the correctly named processed HDF5 file that the training configs (e.g. configs/v2/base.yaml) expect.

## Related tools

- **Python** (Runs the v1 header-processing script (scripts/data/v1_add_headers.py) and downstream train/inference scripts that consume the ingested data) — https://github.com/uclchem/neuralpdr
- **HDF5** (File format of both the raw Zenodo deposits (e.g. 3dpdr_dataset_v3.h5) and the processed training-ready files in data/processed/)
- **3D-PDR** (Source simulation code that generated the deposited PDR chemistry datasets being ingested)

## Examples

```
wget https://doi.org/10.5281/zenodo.7310832 -O data/zenodo/v2/simulations.tgz && ./scripts/data/workflow_v2.sh
```

## Evaluation signals

- File exists at the exact expected path after staging: data/zenodo/v2/simulations.tgz or data/zenodo/v3/3dpdr_dataset_v3.h5
- Header-processing script completes without error and produces data/processed/3dpdr_dataset_v{1,2,3}.h5 matching the README directory listing
- Subsequent training run (python src/neuralpdr/train.py configs/v2/base.yaml) can open the processed HDF5 file and write checkpoints to the configured run directory
- For a no-download sanity check, the same pipeline runs end-to-end on the bundled data/test/ datasets
- Inference on the ingested dataset (python src/neuralpdr/inference.py --dataset_path data/processed/3dpdr_dataset_v2.h5 ...) executes against the staged file

## Limitations

- Full Zenodo deposits can be large; README offers data/test/ datasets only as a quick-validation alternative, not a substitute for real training data
- The Neptune logging callback is deprecated as of March 2026 and will stop functioning, affecting any tracking configured during runs following ingestion
- Only three dataset versions (v1 uniform 1D clouds, v2 varying density clouds, v3 3D giant molecular cloud) have documented Zenodo DOIs and processing scripts

## Evidence

- [readme] README dataset layout: "Download the datasets from Zenodo (links below) and place them in `data/zenodo/` following this structure"
- [readme] README v2/v3 script mapping: "./scripts/data/workflow_v2.sh           # for v2
./scripts/data/workflow_v3.sh           # for v3 "
- [methods] article v2 DOI: "The second dataset can be retrieved from: https://doi.org/10.5281/zenodo.7310832"
- [methods] article processed output: "This creates processed HDF5 files in `data/processed/`"
- [full_text] task_002 validation workflow: [evidence span withheld — anchor status: not_found]
