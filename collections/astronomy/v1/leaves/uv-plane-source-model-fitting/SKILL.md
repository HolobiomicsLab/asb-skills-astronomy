---
name: uv-plane-source-model-fitting
description: Use when image-based flux densities from chimenea's iteratively re-Cleaned
  images need refinement for candidate sources and the underlying CASA measurement
  set of visibilities (with known ground-truth fluxes for simulations) is available.
when_to_use_negative:
- Fields containing extended emission or complex blended sources — the chimenea pipeline
  does not automate handling of such sources, which 'still need to be manually reduced'
- Before candidate sources have been identified — the paper positions uv-plane fitting
  strictly as a post-detection extension to improve flux-density accuracy of already-identified
  sources
- When the goal is the imaging/source-detection step itself — this skill supplements,
  not replaces, the iterative re-Cleaning and source-detection logic in chimenea's
  pipeline.py
tools:
- name: chimenea
  role: Runs the upstream image-based pipeline (iterative RMS re-estimation with repeated
    Clean) whose detected candidate sources are the subjects of uv-plane flux refinement
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer through which the uv-plane model fitting of CASA subroutines
    is scripted from a generic Python process
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the underlying image-synthesis and visibility-handling subroutines
    on which both chimenea imaging and uv-plane fitting are built
- name: UVMULTIFIT
  role: Reference approach (Martí-Vidal et al. 2014) whose uv-plane model-fitting
    manner the extension follows
- name: Python
  role: Language from which drive-casa automates interaction with CASA subroutines
    and in which the flux comparison table is generated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:392bc1073107bfe8928d2ed2f6c28302b6551fcd7ff438cd836084a569c84b73
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/chimenea
  tools:
  - CASA
  - Python
  - UVMULTIFIT
  - chimenea
  - drive-casa
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# uv-plane-source-model-fitting

## Summary

Refines flux-density measurements of candidate sources already identified from chimenea cleaned images by fitting source models directly to the visibilities of a CASA measurement set, in the manner of UVMULTIFIT (Martí-Vidal et al. 2014), scripted from Python via drive-casa. This is the paper's sanctioned extension for improving flux-density accuracy over image-plane measurements.

## When to use

Use when image-based flux densities from chimenea's iteratively re-Cleaned images need refinement for candidate sources and the underlying CASA measurement set of visibilities (with known ground-truth fluxes for simulations) is available.

## When NOT to use

- Fields containing extended emission or complex blended sources — the chimenea pipeline does not automate handling of such sources, which 'still need to be manually reduced'
- Before candidate sources have been identified — the paper positions uv-plane fitting strictly as a post-detection extension to improve flux-density accuracy of already-identified sources
- When the goal is the imaging/source-detection step itself — this skill supplements, not replaces, the iterative re-Cleaning and source-detection logic in chimenea's pipeline.py

## Inputs

- CASA measurement set containing visibilities (for validation, a simulated dataset with known ground-truth source fluxes)
- chimenea image-based pipeline output: cleaned images and image-based flux densities of candidate sources, produced via iterative RMS-estimation and automated source-detection

## Outputs

- flux_comparison.csv — one row per simulated source with uv-plane-fitted flux density, image-based flux density, and ground-truth flux columns populated

## How to apply

First run the chimenea image-based pipeline — iterative RMS-estimation with repeated Clean operations driven via drive-casa/CASA subroutines — to detect candidate sources and obtain image-based flux densities. Then, rather than relying on the image plane, fit source models directly in the uv-plane to the underlying visibilities, following the UVMULTIFIT approach (Martí-Vidal et al. 2014), scripted from a generic Python process through drive-casa's interface to CASA subroutines. Tabulate, for each simulated source, the uv-plane-fitted flux density, the image-based flux density, and the ground-truth flux, computing the divergence between the two measurement routes. Write the comparison to flux_comparison.csv and accept the result qualitatively, since the paper reports no quantitative threshold for this extension.

## Related tools

- **chimenea** (Runs the upstream image-based pipeline (iterative RMS re-estimation with repeated Clean) whose detected candidate sources are the subjects of uv-plane flux refinement) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer through which the uv-plane model fitting of CASA subroutines is scripted from a generic Python process) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the underlying image-synthesis and visibility-handling subroutines on which both chimenea imaging and uv-plane fitting are built)
- **UVMULTIFIT** (Reference approach (Martí-Vidal et al. 2014) whose uv-plane model-fitting manner the extension follows)
- **Python** (Language from which drive-casa automates interaction with CASA subroutines and in which the flux comparison table is generated)

## Evaluation signals

- flux_comparison.csv exists and contains one row per simulated source
- All three flux columns (uv-plane-fitted, image-based, ground-truth) are populated for every row
- The reported agreement/divergence between uv-plane-fitted and image-based flux densities is computed against the known ground-truth fluxes (qualitative acceptance check only, as the paper reports no quantitative threshold)
- Uv-plane fitting is applied only to candidate sources already identified by the chimenea imaging pipeline, not to blind detections
- Fitting is performed against the visibilities of the measurement set via drive-casa/CASA subroutines, not by re-measuring the cleaned image

## Limitations

- The paper reports no quantitative acceptance threshold for this extension — validation of agreement/divergence is qualitative against ground truth
- Extended emission and complex blended sources are not handled automatically by the pipeline and still require manual reduction
- Computational overhead of the iterative re-Cleaning (and by extension added fitting stages) is unquantified for large datasets and 'may become a significant contribution to the overall processing time'
- Only in the manner of UVMULTIFIT (Martí-Vidal et al. 2014) is the approach specified; no version numbers are given for the software packages described

## Evidence

- [discussion] uv-plane extension rationale: [evidence span withheld — anchor status: not_found]
- [discussion] iterative re-Clean context: [evidence span withheld — anchor status: not_found]
