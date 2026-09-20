---
name: anchor-point-selection-for-latent-interpolation
description: Use when training a spectral surrogate (e.g. an IAE for SUSHI) from a
  physical emission model such as APEC thermal plasma or power-law synchrotron spectra,
  and the interpolatory scheme needs anchor points spanning the parameter boundaries.
when_to_use_negative:
- The input spectra are observational data to be unmixed directly rather than a physical
  model sampled over a known parameter range — anchors come from the forward model,
  not the data
- The spectral model is stationary (a single fixed spectrum per component) so no interpolatory
  model over parameters is needed
- There is no defined parameter range with identifiable boundaries, making boundary
  anchor selection ill-posed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: IAE (Interpolatory Auto-Encoder)
  role: Learning framework whose interpolatory scheme is built from the selected anchor
    points (Output['Psi'])
  repo: https://github.com/jbobin/IAE
- name: SUSHI
  role: Downstream source-separation algorithm that consumes the anchor-trained IAE
    spectral model
  repo: https://github.com/jmlascar/SUSHI
- name: JAX
  role: Backend used to implement and train the IAE encoder/decoder with the anchor
    points
  repo: https://github.com/jbobin/IAE
- name: APEC model (AtomDB)
  role: Physical emission model (collisionally-ionized thermal plasma) sampled over
    parameter ranges to generate candidate spectra and boundary anchors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:46a24a4d9996667036a8dae43377b72ecea5a40205010b4c937939d135a80608
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jbobin/IAE
  tools:
  - APEC model (AtomDB)
  - IAE (Interpolatory Auto-Encoder)
  - IAE training code (github.com/jbobin/IAE)
  - JAX
  - Python
  - SUSHI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# anchor-point-selection-for-latent-interpolation

## Summary

Selects boundary-parameter spectra (e.g. lowest/highest temperature and redshift) from a physical model as anchor points so a trained Interpolatory Auto-Encoder (IAE) can build a nonlinear interpolatory surrogate spanning the full physical-parameter space.

## When to use

Use when training a spectral surrogate (e.g. an IAE for SUSHI) from a physical emission model such as APEC thermal plasma or power-law synchrotron spectra, and the interpolatory scheme needs anchor points spanning the parameter boundaries.

## When NOT to use

- The input spectra are observational data to be unmixed directly rather than a physical model sampled over a known parameter range — anchors come from the forward model, not the data
- The spectral model is stationary (a single fixed spectrum per component) so no interpolatory model over parameters is needed
- There is no defined parameter range with identifiable boundaries, making boundary anchor selection ill-posed

## Inputs

- Pool of >=1e3 simulated spectra from the physical model (numpy arrays of shape (N, n_E), n_E = number of spectral channels)
- Physical-parameter values (temperature, redshift, etc.) for each simulated spectrum, used to identify boundary points
- Chandra instrumental response files (ARF and RMF) for forward-modeling spectra

## Outputs

- Output['Psi']: anchor-point spectra at boundary parameters, shape (N_anchor, n_E)
- Output['X_train'] and Output['X_valid']: shuffled 70/30 train/validation spectra, numpy arrays of shape (N, n_E)
- A trained IAE model saved under its fname, ready for use in SUSHI

## How to apply

Generate a large number of spectra (at least 1e3) from the physical model (APEC collisionally-ionized thermal plasma from AtomDB, or power-law synchrotron emission) over the energy and physical-parameter ranges of Appendix Table A.2, convolving with the Chandra instrumental response (ARF/RMF) where required. From this pool, select the spectra lying at the boundary parameters — e.g. the highest and lowest temperature and redshift — and store them as the anchor points in Output['Psi']. Shuffle the remaining spectra and split 70/30 into training and validation sets stored as numpy arrays of shape (N, n_E) in Output['X_train'] and Output['X_valid']. The anchors define the barycentric span from which the IAE encoder/decoder learn an interpolation scheme (Eq. 8 cost with weight mu), so they must cover the extremes of the parameter domain; train with Adagrad over three epochs per Table A.1 and verify the cost converged. Repeat for each physical component (one IAE per component).

## Related tools

- **IAE (Interpolatory Auto-Encoder)** (Learning framework whose interpolatory scheme is built from the selected anchor points (Output['Psi'])) — https://github.com/jbobin/IAE
- **SUSHI** (Downstream source-separation algorithm that consumes the anchor-trained IAE spectral model) — https://github.com/jmlascar/SUSHI
- **JAX** (Backend used to implement and train the IAE encoder/decoder with the anchor points) — https://github.com/jbobin/IAE
- **APEC model (AtomDB)** (Physical emission model (collisionally-ionized thermal plasma) sampled over parameter ranges to generate candidate spectra and boundary anchors)

## Examples

```
Output = {'Psi': boundary_spectra, 'X_train': X_train, 'X_valid': X_valid}; train_iae(Output, niters=..., opts=[3,3,3], steps=..., fname='thermal_iae')
```

## Evaluation signals

- All anchor points in Output['Psi'] sit at extreme (boundary) parameter values, e.g. lowest/highest temperature and redshift, within the Table A.2 ranges
- Output['Psi'], Output['X_train'], and Output['X_valid'] are numpy arrays of shape (N, n_E); training/validation split is 70/30 of the shuffled non-anchor spectra
- The IAE training cost (two-term interpolation cost, Eq. 8) successfully converges (plateaued cost curve) across all three epochs for every network
- The trained decoder reproduces held-out validation spectra generated by the physical model within acceptable error (SUSHI achieved average RMSE per spectrum of 0.19 thermal / 0.16 synchrotron on the toy model)
- One anchor set / IAE exists per physical component in the model

## Limitations

- Anchor coverage defines the barycentric span: parameters outside the anchor-defined boundaries are not guaranteed to be interpolated correctly
- Pondering (cost_weight) may be needed for very dynamic ranges; otherwise the anchors and training may under-represent low-amplitude regions
- Trained IAE networks/weights were not originally stated as shared in the paper (only code), so exact anchor sets may not be reproducible from the publication alone
- The toy model depended on a simulation (Orlando et al. 2016) provided by a third party rather than a publicly hosted dataset link

## Evidence

- [readme] Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"].: "Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"]."
- [readme] Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set.: "Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set."
- [methods] The trained spectral model used by SUSHI is an Interpolatory Auto-Encoder (IAE), a generative model chosen because it can be trained with a limited number of samples.: [evidence span withheld — anchor status: not_found]
- [methods] An IAE model may be trained for each physical component. The code to do so is openly accessible on Git Hub: "An IAE model may be trained for each physical component. The code to do so is openly accessible on Git Hub"
- [readme] The basic idea would be to build barycenters from known anchor points: [evidence span withheld — anchor status: not_found]
