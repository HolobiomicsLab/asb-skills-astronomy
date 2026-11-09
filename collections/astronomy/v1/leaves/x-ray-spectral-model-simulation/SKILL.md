---
name: x-ray-spectral-model-simulation
description: 'Use when building a learned spectral surrogate: you need many synthetic
  spectra spanning a physical-parameter grid (e.g. temperature, redshift) in a defined
  energy band, convolved with an instrument response, to train a generative spectral
  model such as an IAE.'
when_to_use_negative:
- You only need a single spectrum or a classical forward model call — use the physical
  model directly (e.g. XSpec model in Sherpa) instead of training a surrogate
- Your physical component's parameter space or spectral channels differ from the Chandra
  band / Table A.2 ranges; the IAE must be retrained for your own model and energy
  grid
- No spectral model is available for a component (SUSHI applies only where a fixed
  number of components each have a spectral model, which need not be analytical but
  must exist)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3399
tools:
- name: IAE (Interpolatory Auto-Encoder)
  role: Generative model architecture trained on the simulated spectra to act as the
    learned physical spectral surrogate in SUSHI
  repo: https://github.com/jbobin/IAE
- name: SUSHI
  role: Downstream algorithm that consumes the trained IAE models as spectral constraints
    during non-stationary unmixing; repository also ships 'Training an IAE.ipynb'
    and IAE_JAX_v2_devl_2023.py
  repo: https://github.com/jmlascar/SUSHI
- name: JAX
  role: Package used in Python 3.10 to code and train the IAE models
- name: Sherpa
  role: Hosts the XSpec look-up-table physical model that the trained IAE decoder
    replaces as a >60x faster surrogate
- name: XSpec
  role: Provides the APEC look-up-table physical model used to generate the training
    spectra
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:f2aba403a7578342b645aeafef7ab48aa1af545c84840e80c19b02cfd4e305d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jbobin/IAE
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - IAE training code (github.com/jbobin/IAE)
  - JAX
  - Python
  - SUSHI
  - Sherpa
  - XSpec
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# x-ray-spectral-model-simulation

## Summary

Simulate large ensembles (≥1e3) of X-ray spectra from physical models (APEC collisionally-ionized thermal plasma from AtomDB, or power-law synchrotron emission) over the paper's parameter ranges, convolved with the Chandra instrumental response (ARF and RMF files), and format them as training/anchor/validation sets for training IAE spectral surrogate models used by SUSHI.

## When to use

Use when building a learned spectral surrogate: you need many synthetic spectra spanning a physical-parameter grid (e.g. temperature, redshift) in a defined energy band, convolved with an instrument response, to train a generative spectral model such as an IAE.

## When NOT to use

- You only need a single spectrum or a classical forward model call — use the physical model directly (e.g. XSpec model in Sherpa) instead of training a surrogate
- Your physical component's parameter space or spectral channels differ from the Chandra band / Table A.2 ranges; the IAE must be retrained for your own model and energy grid
- No spectral model is available for a component (SUSHI applies only where a fixed number of components each have a spectral model, which need not be analytical but must exist)

## Inputs

- Physical spectral model (APEC collisionally-ionized thermal plasma from AtomDB, http://www.atomdb.org/physics.php, or power-law synchrotron emission)
- Chandra instrumental response files (ARF and RMF)
- Energy band and physical-parameter ranges from Appendix Table A.2 (e.g. temperature, redshift)
- IAE architecture and training hyperparameters from Appendix Table A.1 (anchor counts, layer counts, LReLU activation, niters, step sizes, mu)

## Outputs

- Training/anchor/validation sets as numpy arrays of shape (N, n_E): Output['Psi'] (anchor points), Output['X_train'] (70%), Output['X_valid'] (30%)
- Five trained IAE spectral models (encoder E and decoder D), each saved under its fname

## How to apply

For each of the five spectral models, generate at least 1e3 spectra from the physical model (APEC collisionally-ionized thermal plasma from AtomDB, or power-law synchrotron emission) over the energy and physical-parameter ranges of Appendix Table A.2, convolving with the Chandra instrumental response (ARF and RMF files) where required. Select spectra at boundary parameters (lowest/highest temperature and redshift) as anchor points stored as Output['Psi'], and split the remaining shuffled spectra 70/30 into Output['X_train'] and Output['X_valid'], all as numpy arrays of shape (N, n_E) where n_E is the number of spectral channels. Using the 'Training an IAE.ipynb' tutorial and the jbobin/IAE code (Python 3.10/JAX), train the encoder E and decoder D multilayer perceptrons with the anchor-point counts, layer counts and LReLU activation of Table A.1, minimizing the two-term interpolation cost of Equation 8 with relative-weight factor mu. Run three epochs with the Adagrad optimizer using the per-epoch step sizes and iteration counts (niters) from Table A.1, optionally enabling cost_weight pondering for very dynamic ranges, then save each model under its fname. The rationale is that the trained IAE decoder is a fast surrogate — over 60x faster than calling the look-up-table-based XSpec model in Sherpa — enabling SUSHI's per-voxel optimization.

## Related tools

- **IAE (Interpolatory Auto-Encoder)** (Generative model architecture trained on the simulated spectra to act as the learned physical spectral surrogate in SUSHI) — https://github.com/jbobin/IAE
- **SUSHI** (Downstream algorithm that consumes the trained IAE models as spectral constraints during non-stationary unmixing; repository also ships 'Training an IAE.ipynb' and IAE_JAX_v2_devl_2023.py) — https://github.com/jmlascar/SUSHI
- **JAX** (Package used in Python 3.10 to code and train the IAE models)
- **Sherpa** (Hosts the XSpec look-up-table physical model that the trained IAE decoder replaces as a >60x faster surrogate)
- **XSpec** (Provides the APEC look-up-table physical model used to generate the training spectra)

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env; jupyter notebook 'Training an IAE.ipynb'  # build Output['Psi'/'X_train'/'X_valid'] from simulated spectra, then train with opts=['adagrad'], steps, niters, fname
```

## Evaluation signals

- Training/validation cost curves plateau (converge) for all five networks before the IAEs are used in SUSHI
- Output['Psi'], Output['X_train'], Output['X_valid'] are numpy arrays of shape (N, n_E) with the 70/30 train/validation split
- At least 1e3 spectra generated per physical model, with anchor points at the boundary parameters (lowest/highest temperature and redshift)
- Trained model files saved under their fname and loadable via the IAE_JAX_v2 code
- Decoder call speed is far faster than the look-up-table physical model (paper reports >60x), and simulated spectra reproduce APEC/synchrotron shapes after ARF/RMF convolution

## Limitations

- No random seeds or training-hyperparameter randomness handling is described for IAE training, so exact reproduction of the five trained networks may not be possible
- Trained IAE networks/weights are not stated as shared in the paper (only code availability); only the repository's readily trained IAE models serve testing purposes
- The IAE surrogate is only valid over the energy and physical-parameter ranges used in training (Appendix Table A.2); extrapolation outside these bounds is not supported

## Evidence

- [methods] Generate spectra from physical model over Table A.2 ranges: [evidence span withheld — anchor status: not_found]
- [readme] README anchor/train/valid construction: "Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points: Output["Psi"]. Shuffle the rest of the training set. Separate it into a"
- [methods] IAE as surrogate for physical model: "we sought to train a surrogate model M⋆(θ) that would have the following characteristics"
- [methods] Decoder faster than look-up table: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [readme] Convergence check requirement: "Once the training is complete (this may take a while), check that the cost successfully converged."
