---
name: appendix-table-reproduction
description: Use when a paper publishes an appendix results table (e.g. Table B.1
  of the SUSHI paper) and the associated artifact (trained models, validation splits)
  is released in the companion repository, so the numbers can be regenerated and checked.
when_to_use_negative:
- The IAE network weights for a component are not shared in the repository — only
  the IAE code and training notebooks are available, so reproducing the table would
  require retraining first.
- The goal is to validate SUSHI's unmixing output (e.g. RMSE of 0.19 thermal / 0.16
  synchrotron vs the classic per-pixel fit) rather than the surrogate spectral model
  itself.
- Only the paper's headline results on Cassiopeia A or the Crab Nebula Chandra data
  are of interest and no appendix table with a reproducible artifact exists.
tools:
- name: SUSHI
  role: Companion repository providing the five readily trained IAE spectral models
    and the training/validation split conventions (Output['X_valid']) needed to recompute
    the residuals
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: 'The spectral surrogate being evaluated: encoder E, linear interpolator I
    onto anchor points'' barycentric span, and decoder D are composed to reconstruct
    each held-out spectrum'
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Package used to code and run the IAE models for the forward passes through
    E, I, and D
  repo: https://github.com/jmlascar/SUSHI
- name: Python 3.10
  role: Language in which SUSHI and the IAE models were coded; environment specified
    via SUSHI_env.yml
  repo: https://github.com/jmlascar/SUSHI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:65c2f7b74a93f74aa03e88d5618faef5308f62783623382dcbeb4e8540d034e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
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
yes_rate: 0.8
n_claims: 5
---

# appendix-table-reproduction

## Summary

Reproduce a paper's appendix benchmark table — here SUSHI's Table B.1 of per-component IAE held-out reconstruction residuals — by re-running the described computation on the released trained models and comparing the five resulting values to the published reference numbers.

## When to use

Use when a paper publishes an appendix results table (e.g. Table B.1 of the SUSHI paper) and the associated artifact (trained models, validation splits) is released in the companion repository, so the numbers can be regenerated and checked.

## When NOT to use

- The IAE network weights for a component are not shared in the repository — only the IAE code and training notebooks are available, so reproducing the table would require retraining first.
- The goal is to validate SUSHI's unmixing output (e.g. RMSE of 0.19 thermal / 0.16 synchrotron vs the classic per-pixel fit) rather than the surrogate spectral model itself.
- Only the paper's headline results on Cassiopeia A or the Crab Nebula Chandra data are of interest and no appendix table with a reproducible artifact exists.

## Inputs

- Five readily trained IAE models (one per physical component) from the 'IAE models' directory of the JMLascar/SUSHI repository
- Each network's held-out testing spectra: the 30% validation split of the shuffled training set, Output['X_valid'], numpy arrays of shape (N, n_E)
- Anchor points Output['Psi'] defining each IAE's barycentric span
- Reference values from Table B.1: -0.024%, 0.040%, 0.041%, -0.035%, -0.052%

## Outputs

- iae_reconstruction_residuals.csv containing one mean reconstruction residual per IAE network
- Comparison of each computed average residual to its Table B.1 reference value

## How to apply

Load the five readily trained IAE models (one per physical component) from the SUSHI repository's 'IAE models' folder and each network's held-out testing spectra, which the SUSHI README defines as the 30% validation split of the shuffled training set, stored as Output['X_valid'] with shape (N, n_E). For each network, forward-pass every testing spectrum s_i through the encoder E, project onto the anchor points' barycentric span via the linear interpolator I (as in the jbobin/IAE interpolatory scheme), and decode with D to get the reconstructed spectrum s_hat_i. Compute the element-wise relative residual R = (s_hat_i - s_i)/s_i for every reconstructed spectrum, then average the residuals across each network's testing set to obtain one mean reconstruction error per IAE. Compare the five average residuals against the Table B.1 reference values -0.024%, 0.040%, 0.041%, -0.035%, -0.052%; acceptance is a match within a small tolerance since the residuals are expected to be well under 0.1% in magnitude. Save the five values as iae_reconstruction_residuals.csv.

## Related tools

- **SUSHI** (Companion repository providing the five readily trained IAE spectral models and the training/validation split conventions (Output['X_valid']) needed to recompute the residuals) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (The spectral surrogate being evaluated: encoder E, linear interpolator I onto anchor points' barycentric span, and decoder D are composed to reconstruct each held-out spectrum) — https://github.com/jbobin/IAE
- **JAX** (Package used to code and run the IAE models for the forward passes through E, I, and D) — https://github.com/jmlascar/SUSHI
- **Python 3.10** (Language in which SUSHI and the IAE models were coded; environment specified via SUSHI_env.yml) — https://github.com/jmlascar/SUSHI

## Examples

```
R = (D(I(E(X_valid), anchors)) - X_valid) / X_valid; np.mean(R, axis=0).to_csv('iae_reconstruction_residuals.csv')
```

## Evaluation signals

- Each network's computed average residual matches its Table B.1 reference value (-0.024%, 0.040%, 0.041%, -0.035%, -0.052%) within a small tolerance
- All five mean residuals are well under 0.1% in magnitude
- The validation set used is exactly the 30% split of the shuffled training set (Output['X_valid']), not the training split, with arrays of shape (N, n_E)
- The output CSV iae_reconstruction_residuals.csv contains exactly five rows, one per IAE network

## Limitations

- Only code availability is stated for SUSHI; the trained IAE networks' weights being shipped in the repository is what makes direct table reproduction possible — without them the table must be retrained from scratch.
- No random seeds or handling of training-time randomness are documented for IAE training, so retraining from scratch will not exactly reproduce Table B.1.
- No JAX or Python version is specified for the IAE training in the paper (only Python 3.10 / SUSHI_env.yml for SUSHI); older JAX versions require the code in the older_version folder.
- The IAE is a surrogate for physical spectra: reconstruction residuals only validate the learned model, not SUSHI's downstream unmixing or its physical-parameter recovery (SUSHI's output is unmixed spectra, not physical parameters).

## Evidence

- [readme] Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set. Output["X_train"] and Output["X_valid"].: "Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set. Output["X_train"] and Output["X_valid"]."
- [readme] IAE models: readily trained IAE spectral models for testing SUSHI.: "IAE models: readily trained IAE spectral models for testing SUSHI."
- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [abstract] For the spectral model, we used an Interpolatory Auto-Encoder, a generative model that: "For the spectral model, we used an Interpolatory Auto-Encoder, a generative model that"
- [discussion] Details of the architecture of the five networks can be found in Table A.1, and details about the training sets of each can be found in A.2.: "Details of the architecture
of the five networks can be found in Table A.1, and details about
the training sets of each can be found in A.2."
