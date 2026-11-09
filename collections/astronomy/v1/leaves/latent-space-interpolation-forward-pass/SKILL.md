---
name: latent-space-interpolation-forward-pass
description: Use when you have a trained IAE spectral model (one per physical component)
  and need to reconstruct held-out spectra via encoder/interpolator/decoder, e.g.
  to reproduce Table B.1 reconstruction residuals.
when_to_use_negative:
- When you need to train a new IAE from simulated spectra (use the 'Training an IAE.ipynb'
  workflow instead — this skill evaluates an already trained network)
- When the physical spectral model can be evaluated directly (e.g. the APEC/XSpec
  look-up tables in Sherpa) and the 60x speed advantage of the decoder is irrelevant
- When spectra lie outside the parameter range spanned by the anchor points, since
  the IAE constrains solutions to the barycentric span of the anchors
tools:
- name: IAE (Interpolatory Auto-Encoder)
  role: The spectral surrogate model whose E/I/D forward pass reconstructs spectra;
    code from jbobin/IAE is bundled as IAE_JAX_v2_devl_2023.py in the SUSHI repository
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Package used to implement and run the IAE networks (f, g dense layers and
    linear interpolator I)
- name: SUSHI
  role: Hosts the readily trained IAE models and the testing notebook in which the
    forward pass over Output['X_valid'] is evaluated
  repo: https://github.com/jmlascar/SUSHI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:548032b8e34604087115fe3cc7a9260efa58cab2a3bd480f5e350ed105b315ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jbobin/IAE
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - SUSHI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# latent-space-interpolation-forward-pass

## Summary

Forward-pass through a trained Interpolatory Auto-Encoder (IAE) — encoder E, linear interpolator I over anchor points, decoder D — to reconstruct spectra and evaluate the barycentric-span surrogate used by SUSHI for non-stationary unmixing of hyperspectral images.

## When to use

Use when you have a trained IAE spectral model (one per physical component) and need to reconstruct held-out spectra via encoder/interpolator/decoder, e.g. to reproduce Table B.1 reconstruction residuals.

## When NOT to use

- When you need to train a new IAE from simulated spectra (use the 'Training an IAE.ipynb' workflow instead — this skill evaluates an already trained network)
- When the physical spectral model can be evaluated directly (e.g. the APEC/XSpec look-up tables in Sherpa) and the 60x speed advantage of the decoder is irrelevant
- When spectra lie outside the parameter range spanned by the anchor points, since the IAE constrains solutions to the barycentric span of the anchors

## Inputs

- Trained IAE model files (encoder E, decoder D, anchor points Psi) from the 'IAE models' directory of JMLascar/SUSHI, one per physical component
- Held-out testing spectra: 30% validation split Output['X_valid'] of the shuffled training set, numpy arrays of shape (N, n_E)
- Anchor-point spectra Output['Psi'] at boundary parameter values

## Outputs

- Reconstructed spectra s_hat_i for each held-out testing spectrum
- Element-wise relative residuals R = (s_hat_i - s_i)/s_i per spectrum
- Mean reconstruction residual per IAE network, comparable to Table B.1 values
- CSV table iae_reconstruction_residuals.csv with the five average residuals

## How to apply

Load each readily trained IAE model from the 'IAE models' directory of the JMLascar/SUSHI repository together with its held-out validation spectra, i.e. the 30% validation split (Output['X_valid']) of the shuffled training set. For each testing spectrum, run the forward pass s_hat = D(I(E(s))): the encoder E (dense layers f, g with non-linear activations) maps the spectrum to the code domain, the linear interpolator I builds barycentric combinations of the known anchor points (Output['Psi'], the spectra at boundary parameters such as highest and lowest temperature), and the decoder D maps the interpolated code back to a spectrum. Compute the element-wise residual R = (s_hat - s)/s for every reconstructed spectrum, average across each network's testing set, and compare the five per-network mean residuals to the Table B.1 reference values (-0.024%, 0.040%, 0.041%, -0.035%, -0.052%). The IAE serves as a surrogate for the physical spectral model because a call to D is over 60 times faster than calling the look-up-table-based physical model (XSpec model in Sherpa).

## Related tools

- **IAE (Interpolatory Auto-Encoder)** (The spectral surrogate model whose E/I/D forward pass reconstructs spectra; code from jbobin/IAE is bundled as IAE_JAX_v2_devl_2023.py in the SUSHI repository) — https://github.com/jbobin/IAE
- **JAX** (Package used to implement and run the IAE networks (f, g dense layers and linear interpolator I))
- **SUSHI** (Hosts the readily trained IAE models and the testing notebook in which the forward pass over Output['X_valid'] is evaluated) — https://github.com/jmlascar/SUSHI

## Examples

```
import IAE_JAX_v2_devl_2023 as iae; s_hat = iae.D(iae.I(iae.E(s_valid)))
```

## Evaluation signals

- Each network's mean testing-set residual matches its Table B.1 reference value (-0.024%, 0.040%, 0.041%, -0.035%, -0.052%) within a small tolerance
- All mean residuals are well under 0.1% in magnitude
- Barycentric weights produced by the linear interpolator I define valid convex combinations of the anchor points
- Reconstructed spectra have the same shape (N, n_E) as the input numpy arrays, with n_E the number of spectral channels
- Decoder D runs orders of magnitude faster than the physical look-up-table model (reported as over 60x faster than the XSpec model in Sherpa)

## Limitations

- The IAE constrains solutions to the barycentric span of a limited set of anchor points, so extrapolation beyond the boundary-parameter spectra is unsupported
- Residuals should be evaluated only on the held-out 30% validation split of the shuffled training set, not the training data, to avoid optimistic estimates
- Trained IAE networks/weights are not stated as shared by the paper (only code availability is given), so reproducing Table B.1 may depend on the repositories' readily trained models

## Evidence

- [readme] where the f and g functions are standard dense layers equipped with a non-linear activation functions. I is a linear interpolator in the code domain.: [evidence span withheld — anchor status: external_doc]
- [readme] Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"].: "Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"]."
- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [intro] we present our trained spectral model, an Interpolatory Auto-Encoder (IAE): "we present our trained spectral model, an Interpolatory Auto-Encoder (IAE)"
- [readme] Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set. Output["X_train"] and Output["X_valid"].: "Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set. Output["X_train"] and Output["X_valid"]."
