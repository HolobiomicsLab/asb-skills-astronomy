---
name: autoencoder-testing-set-evaluation
description: Use when one or more trained IAE spectral models (e.g. the five SUSHI
  networks for thermal/synchrotron components) must be validated on held-out testing
  spectra, with reconstruction residuals compared to published reference values such
  as Table B.1.
when_to_use_negative:
- The spectra to evaluate come from raw observational Chandra data cubes rather than
  the simulated training/validation set the IAE was trained on — SUSHI's semi-blind
  unmixing, not held-out IAE evaluation, is the right tool there
- The IAE training cost has not converged (check the cost curve after the three training
  epochs defined by niters/opts/steps in 'Training an IAE.ipynb'), so residuals would
  not be interpretable
- Only one generic autoencoder is available with no barycentric anchor-point structure
  — the interpolatory projection step I cannot be applied
tools:
- name: IAE (Interpolatory Auto-Encoder)
  role: The spectral surrogate model being evaluated; its architecture (encoder E,
    linear interpolator I, decoder D) is exercised on testing spectra to compute residuals
  repo: https://github.com/jbobin/IAE
- name: SUSHI
  role: Provides the five readily trained IAE models and the training/validation data
    pipeline (Output dictionary with X_train, X_valid, Psi) used in this evaluation
  repo: https://github.com/jmlascar/SUSHI
- name: JAX
  role: Framework in which the IAE models were coded and are executed for the encoder/interpolator/decoder
    forward pass
- name: Python 3.10
  role: Implementation language for the evaluation script and CSV export
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:ce5ed771743d0772f5d54e58285cd78ae28f8ca0d2ec5be7868e66d1381cc918
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
yes_rate: 0.625
n_claims: 8
---

# autoencoder-testing-set-evaluation

## Summary

Evaluates a trained Interpolatory Auto-Encoder (IAE) spectral surrogate by forward-passing held-out validation spectra through encoder E, the barycentric linear interpolator I, and decoder D, then computing mean relative reconstruction residuals to benchmark against the paper's Table B.1 figures. This verifies the generative spectral model SUSHI relies on generalizes beyond its training samples.

## When to use

Use when one or more trained IAE spectral models (e.g. the five SUSHI networks for thermal/synchrotron components) must be validated on held-out testing spectra, with reconstruction residuals compared to published reference values such as Table B.1.

## When NOT to use

- The spectra to evaluate come from raw observational Chandra data cubes rather than the simulated training/validation set the IAE was trained on — SUSHI's semi-blind unmixing, not held-out IAE evaluation, is the right tool there
- The IAE training cost has not converged (check the cost curve after the three training epochs defined by niters/opts/steps in 'Training an IAE.ipynb'), so residuals would not be interpretable
- Only one generic autoencoder is available with no barycentric anchor-point structure — the interpolatory projection step I cannot be applied

## Inputs

- Five trained IAE model files (one per physical component) from the SUSHI repository's 'IAE models' directory
- Held-out testing spectra: Output['X_valid'], the 30% validation split of the shuffled training set, numpy array of shape (N, n_E)
- Anchor points Output['Psi'] (boundary-parameter spectra, e.g. highest and lowest temperature), numpy array of shape (N, n_E)
- Reference residual values from Table B.1 of the companion paper

## Outputs

- Five mean reconstruction residuals (one per IAE network), each expected within ~0.1% magnitude
- CSV file named iae_reconstruction_residuals.csv with the five average residuals

## How to apply

Load each readily trained IAE network (one per physical component, e.g. from the SUSHI repo's 'IAE models' directory) and its held-out testing set, which per the README is the 30% validation split Output['X_valid'] of the shuffled training spectra (each set a numpy array of shape (N, n_E), with at least 1e3 spectra). For each testing spectrum s_i, forward-pass through the encoder E, project onto the anchor points' barycentric span via the linear interpolator I (weights defined relative to the boundary anchor points Output['Psi'], e.g. highest/lowest temperature spectra), and decode with D to get s_hat_i. Compute the element-wise relative residual R = (s_hat_i - s_i)/s_i, then average residuals across that network's testing set to yield one mean reconstruction error per IAE. Compare the five mean residuals against the Table B.1 reference values -0.024%, 0.040%, 0.041%, -0.035%, -0.052%, expecting each to match within a small tolerance and to be well under 0.1% in magnitude. Save the five averages as iae_reconstruction_residuals.csv.

## Related tools

- **IAE (Interpolatory Auto-Encoder)** (The spectral surrogate model being evaluated; its architecture (encoder E, linear interpolator I, decoder D) is exercised on testing spectra to compute residuals) — https://github.com/jbobin/IAE
- **SUSHI** (Provides the five readily trained IAE models and the training/validation data pipeline (Output dictionary with X_train, X_valid, Psi) used in this evaluation) — https://github.com/jmlascar/SUSHI
- **JAX** (Framework in which the IAE models were coded and are executed for the encoder/interpolator/decoder forward pass)
- **Python 3.10** (Implementation language for the evaluation script and CSV export)

## Examples

```
s_hat = D(I(E(s_valid), w_bary)); R = (s_hat - s_valid) / s_valid; mean_R = R.mean(axis=0)  # expect |mean_R| < 0.1%, cf. Table B.1
```

## Evaluation signals

- Each network's computed average residual matches its Table B.1 reference value (-0.024%, 0.040%, 0.041%, -0.035%, -0.052%) within a small tolerance
- All residuals are well under 0.1% in magnitude, consistent with the paper's expectation for held-out testing-set reconstruction
- The testing set is exactly the 30% validation split (Output['X_valid']) of the shuffled training set, not the 70% training portion
- Residuals are element-wise relative values R = (s_hat_i - s_i)/s_i, averaged across each network's testing set (one scalar per IAE)
- Output CSV is named iae_reconstruction_residuals.csv and contains exactly five values

## Limitations

- No random seeds or training hyperparameter randomness handling is described for IAE training, so exact reproduction of residuals may vary slightly
- Trained IAE network weights are not explicitly stated as shared in the paper (only code availability is given), so the five readily trained models in the repo must be used or retrained
- The training sets and architectures per network are only documented in supplementary Tables A.1 and A.2 of the paper

## Evidence

- [readme] Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set.: "Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set. Output["X_train"] and Output["X_valid"]."
- [readme] Provide a large number of spectra (1e3 at least) from your physical model.: "Provide a large number of spectra (1e3 at least) from your physical model."
- [readme] Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"].: "Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"]."
- [intro] we present our trained spectral model, an Interpolatory Auto-Encoder (IAE): "we present our trained spectral model, an Interpolatory Auto-Encoder (IAE)"
- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [discussion] Details of the architecture of the five networks can be found in Table A.1, and details about the training sets of each can be found in A.2.: "Details of the architecture of the five networks can be found in Table A.1, and details about the training sets of each can be found in A.2."
- [other] Compute the element-wise residual R = (s_hat_i - s_i)/s_i for each reconstructed spectrum.: [evidence span withheld — anchor status: not_found]
