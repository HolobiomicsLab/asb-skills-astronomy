---
name: interpolatory-autoencoder-decoding
description: Use when a hyperspectral (IFU) unmixing problem needs a differentiable
  spectral model per physical component — e.g. APEC thermal plasma or synchrotron
  power-law spectra convolved with Chandra ARF/RMF responses — and the physical model
  itself is too slow or non-differentiable for optimization.
when_to_use_negative:
- No spectral model (analytical or otherwise) is available for the physical components
  — SUSHI/IAE requires a fixed number of components each with a trainable spectral
  model
- A single call to the physical model is fast enough and gradients are available,
  so no learned surrogate is needed
- Target physical parameters fall outside the anchor-point parameter range, since
  the IAE only interpolates within the barycentric span of the anchor spectra
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: IAE (Interpolatory Auto-Encoder)
  role: The learned interpolatory network being trained and decoded; encoder E and
    decoder D are multilayer perceptrons with residual shortcuts and a linear interpolator
    in the code domain
  repo: https://github.com/jbobin/IAE
- name: SUSHI
  role: 'Downstream consumer: calls the trained decoder D during PALM optimization
    of spectral parameter maps theta_c'
  repo: https://github.com/jmlascar/SUSHI
- name: Python 3.10 / JAX
  role: Implementation environment for the IAE training code and SUSHI
  repo: https://github.com/jmlascar/SUSHI
- name: XSpec (in Sherpa)
  role: Baseline look-up-table-based physical model that the trained IAE decoder replaces
    as a >60x faster surrogate
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:13971d6874df157d596dbd27aa5f1c747ea30069b93c8b14e825ecf7ba2255c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jbobin/IAE
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - IAE training code (github.com/jbobin/IAE)
  - JAX
  - Python
  - Python 3.10 / JAX
  - SUSHI
  - XSpec (in Sherpa)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# interpolatory-autoencoder-decoding

## Summary

Trains and applies an Interpolatory Auto-Encoder (IAE) — a generative, non-linear interpolatory model built from anchor-point spectra — as the learned spectral surrogate that SUSHI calls in place of slow physical models (e.g. APEC lookup tables via XSpec in Sherpa). The decoder D maps physical parameters (temperature, redshift, photon index) to spectra over 60x faster than the look-up-table-based physical model, enabling gradient-based unmixing.

## When to use

Use when a hyperspectral (IFU) unmixing problem needs a differentiable spectral model per physical component — e.g. APEC thermal plasma or synchrotron power-law spectra convolved with Chandra ARF/RMF responses — and the physical model itself is too slow or non-differentiable for optimization.

## When NOT to use

- No spectral model (analytical or otherwise) is available for the physical components — SUSHI/IAE requires a fixed number of components each with a trainable spectral model
- A single call to the physical model is fast enough and gradients are available, so no learned surrogate is needed
- Target physical parameters fall outside the anchor-point parameter range, since the IAE only interpolates within the barycentric span of the anchor spectra

## Inputs

- At least 1e3 simulated spectra per physical component (APEC thermal from AtomDB or power-law synchrotron), optionally convolved with Chandra ARF/RMF instrumental responses
- Anchor-point spectra at boundary physical parameters, as numpy array Output['Psi'] of shape (N, n_E)
- Training/validation spectra, numpy arrays Output['X_train'] (70%) and Output['X_valid'] (30%) of shape (N, n_E)
- Architecture and training hyperparameters from Appendix Tables A.1/A.2 (anchor counts, layer counts, LReLU, optimizer index, per-epoch steps, niters, fname)

## Outputs

- Trained IAE encoder/decoder network files saved under fname (e.g. the pretrained APEC thermal and synchrotron models in 'IAE models/')
- Converged training/validation cost curves
- A fast callable decoder D(theta) used by SUSHI as the surrogate spectral model M*(theta)

## How to apply

For each physical component, generate at least 1e3 spectra from the physical model (APEC from AtomDB, or a power-law) over the parameter ranges of Appendix Table A.2, convolving with the Chandra instrumental response (ARF and RMF files) where required. Select boundary-parameter spectra (e.g. lowest/highest temperature and redshift) as anchor points stored in Output['Psi'], and split the remaining shuffled spectra 70/30 into Output['X_train'] and Output['X_valid'], all numpy arrays of shape (N, n_E). Using the 'Training an IAE.ipynb' tutorial and the jbobin/IAE code (Python 3.10/JAX), train encoder E and decoder D with the anchor counts, layer counts and LReLU activations of Table A.1, minimizing the two-term interpolation cost with relative-weight factor mu. Run three epochs with the Adagrad optimizer using the per-epoch step sizes and niters from Table A.1 (optionally setting cost_weight pondering for very dynamic ranges) and save each trained model under its fname; only use networks whose training/validation cost curve has plateaued. The trained decoder is then embedded in SUSHI as the surrogate M*(theta), so that the PALM algorithm can do gradient descent on the spectral parameter maps theta_c.

## Related tools

- **IAE (Interpolatory Auto-Encoder)** (The learned interpolatory network being trained and decoded; encoder E and decoder D are multilayer perceptrons with residual shortcuts and a linear interpolator in the code domain) — https://github.com/jbobin/IAE
- **SUSHI** (Downstream consumer: calls the trained decoder D during PALM optimization of spectral parameter maps theta_c) — https://github.com/jmlascar/SUSHI
- **Python 3.10 / JAX** (Implementation environment for the IAE training code and SUSHI) — https://github.com/jmlascar/SUSHI
- **XSpec (in Sherpa)** (Baseline look-up-table-based physical model that the trained IAE decoder replaces as a >60x faster surrogate)

## Examples

```
conda env create -f SUSHI_env.yml && conda activate SUSHI_env
```

## Evaluation signals

- Training and validation cost curves must successfully converge (plateau) before the IAE can be used in SUSHI — a non-converged network is rejected
- A call to decoder D should be over 60 times faster than calling the physical model via look-up tables (XSpec model in Sherpa)
- Embedding the decoder in SUSHI on the toy model should yield average per-spectrum RMSE of 0.19 (thermal) and 0.16 (synchrotron), beating the classic pixel-by-pixel fit's 0.22 / 0.19
- Reconstructed spectra should stay within the barycentric span of the anchor points Output['Psi'] — parameters outside the anchor range are not representable
- Shapes of all input sets must be (N, n_E) with n_E matching the spectral channel count (e.g. 375 channels over 3–8.48 keV)

## Limitations

- No random seeds or handling of training hyperparameter randomness is described for IAE training, so training may not be bit-reproducible
- No software version is specified for the JAX package used to code and train the IAE models, and the trained IAE network weights are not stated as shared (only the code is available)
- The IAE decoder returns fitted spectra; SUSHI's final output does not directly provide physical parameters, which must be recovered from the parameter maps theta_c
- No license is stated for the SUSHI repository (the jbobin/IAE repo is GPL-3.0)

## Evidence

- [abstract] For the spectral model, we used an Interpolatory Auto-Encoder, a generative model that: "For the spectral model, we used an Interpolatory Auto-Encoder, a generative model that"
- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [readme] Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"].: "Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"]."
- [readme] Once the training is complete (this may take a while), check that the cost successfully converged.: "Once the training is complete (this may take a while), check that the cost successfully converged."
- [readme] Provide a large number of spectra (1e3 at least) from your physical model.: "Provide a large number of spectra (1e3 at least) from your physical model."
