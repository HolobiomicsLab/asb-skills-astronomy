---
name: x-ray-spectral-model-quality-assessment
description: Use when one or more IAE spectral models have been trained (one per physical
  component) and their generalization must be checked on the 30% held-out validation
  split before using them as surrogates in SUSHI unmixing.
when_to_use_negative:
- The IAE models are still being trained — use the training tutorial ('Training an
  IAE.ipynb') and check training-cost convergence first, not this residual benchmark
- The goal is to assess SUSHI's unmixed output or fitted physical-parameter maps (RMSE
  0.19/0.16 comparisons) rather than the spectral surrogate's reconstruction fidelity
  on held-out spectra
- Input spectra are raw Chandra event data rather than the simulated model spectra
  used to train the IAE networks
tools:
- name: IAE (Interpolatory Auto-Encoder)
  role: Spectral surrogate model whose encoder E, linear interpolator I, and decoder
    D are exercised to compute held-out reconstruction residuals
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Python package in which the IAE models are coded and evaluated (encoder/decoder
    forward passes)
- name: SUSHI
  role: Provides the trained IAE models, the training/validation split conventions
    (Output['X_valid'], Output['Psi']), and the tutorial notebooks used to load and
    test the networks
  repo: https://github.com/jmlascar/SUSHI
- name: Python 3.10
  role: Execution environment; dependencies listed in SUSHI_env.yml (conda env SUSHI_env)
  repo: https://github.com/jmlascar/SUSHI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:bd1e133e81ed9adfeae1050f1bcf12c44f416f72fa391cd6fb07f09f36736a72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jbobin/IAE
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - Python 3.10
  - SUSHI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# x-ray-spectral-model-quality-assessment

## Summary

Quality-assessment procedure for trained Interpolatory Auto-Encoder (IAE) spectral surrogate models: quantify held-out reconstruction residuals element-wise as (s_hat - s)/s and benchmark them against the Table B.1 reference values from the SUSHI paper.

## When to use

Use when one or more IAE spectral models have been trained (one per physical component) and their generalization must be checked on the 30% held-out validation split before using them as surrogates in SUSHI unmixing.

## When NOT to use

- The IAE models are still being trained — use the training tutorial ('Training an IAE.ipynb') and check training-cost convergence first, not this residual benchmark
- The goal is to assess SUSHI's unmixed output or fitted physical-parameter maps (RMSE 0.19/0.16 comparisons) rather than the spectral surrogate's reconstruction fidelity on held-out spectra
- Input spectra are raw Chandra event data rather than the simulated model spectra used to train the IAE networks

## Inputs

- Trained IAE model files (one per physical component) from the SUSHI repository's 'IAE models' directory
- Held-out testing spectra: Output['X_valid'], the 30% validation split of the shuffled training set, as numpy arrays of shape (N, n_E)
- Anchor points Output['Psi'] (boundary-parameter spectra) defining each IAE's barycentric span
- Reference Table B.1 values: -0.024%, 0.040%, 0.041%, -0.035%, -0.052%

## Outputs

- Per-network mean reconstruction residual (element-wise R = (s_hat - s)/s averaged over the testing set)
- CSV table named iae_reconstruction_residuals.csv with one average residual per IAE

## How to apply

Load the five readily trained IAE models from the SUSHI repository's 'IAE models' directory, plus each network's held-out testing spectra — the 30% validation split of the shuffled training set stored as Output['X_valid']. For each network, forward-pass every testing spectrum through the encoder E, project onto the anchor points' barycentric span via the linear interpolator I, and decode with D to obtain the reconstruction s_hat_i. Compute the element-wise residual R = (s_hat_i - s_i)/s_i, average across the testing set to get one mean reconstruction error per IAE, and compare against the Table B.1 references: -0.024%, 0.040%, 0.041%, -0.035%, -0.052%. Residuals are expected to be well under 0.1% in magnitude; the IAE decoder D is the same call that is over 60 times faster than the XSpec look-up-table physical model in Sherpa, so quality checks must confirm accuracy before exploiting that speed.

## Related tools

- **IAE (Interpolatory Auto-Encoder)** (Spectral surrogate model whose encoder E, linear interpolator I, and decoder D are exercised to compute held-out reconstruction residuals) — https://github.com/jbobin/IAE
- **JAX** (Python package in which the IAE models are coded and evaluated (encoder/decoder forward passes))
- **SUSHI** (Provides the trained IAE models, the training/validation split conventions (Output['X_valid'], Output['Psi']), and the tutorial notebooks used to load and test the networks) — https://github.com/jmlascar/SUSHI
- **Python 3.10** (Execution environment; dependencies listed in SUSHI_env.yml (conda env SUSHI_env)) — https://github.com/jmlascar/SUSHI

## Examples

```
import numpy as np; s_hat = D(I(E(Output['X_valid']))); resid = np.mean((s_hat - Output['X_valid'])/Output['X_valid'])
```

## Evaluation signals

- Each network's computed average residual matches its Table B.1 reference value (-0.024%, 0.040%, 0.041%, -0.035%, -0.052%) within a small tolerance
- All five average residuals are well under 0.1% in magnitude
- The evaluation set is exactly the 30% validation split (Output['X_valid']) of the shuffled training set, not the training spectra
- Output CSV iae_reconstruction_residuals.csv contains exactly five rows, one per physical component's IAE
- Training cost convergence was checked before evaluation (README instructs: 'check that the cost successfully converged')

## Limitations

- Trained IAE network weights are not stated as shared in the paper (only code availability) — reproduction may require retraining, and no random seeds or training hyperparameter randomness handling is described
- The JAX and Python versions used to code and train the IAE models are not specified in the paper; 'IAE_JAX_v2_devl_2023.py' targets newer JAX, with older versions in the 'older_version' directory
- Architecture details of the five networks are only in supplementary Table A.1, and training-set details in A.2

## Evidence

- [discussion] The IAE models trained in this work were coded in Python using the package JAX.: "The IAE models trained in this work were coded in Python using the package JAX."
- [readme] Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set. Output["X_train"] and Output["X_valid"].: "Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set. Output["X_train"] and Output["X_valid"]."
- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [methods] An IAE model may be trained for each physical component. The code to do so is openly accessible on Git Hub: "An IAE model may be trained for each physical component. The code to do so is openly accessible on Git Hub"
- [intro] we present our trained spectral model, an Interpolatory Auto-Encoder (IAE): "we present our trained spectral model, an Interpolatory Auto-Encoder (IAE)"
