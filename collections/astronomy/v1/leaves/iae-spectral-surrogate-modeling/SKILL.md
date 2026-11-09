---
name: iae-spectral-surrogate-modeling
description: Use when a physical spectral model (e.g. APEC via XSpec look-up tables)
  is too slow or non-differentiable for per-pixel hyperspectral fitting, and you need
  a learned, differentiable spectral decoder for components like thermal and synchrotron
  emission.
when_to_use_negative:
- You only need a single fixed spectrum per component — classic per-pixel 1D fits
  or stationary unmixing suffice without training a surrogate
- Your component's spectrum has no available physical or simulation-based model to
  generate training spectra
- You need exact physical parameter outputs from the unmixing — SUSHI's fitted unmixed
  spectra do not directly provide physical parameters
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3399
tools:
- name: IAE (Interpolatory Auto-Encoder)
  role: Generative model trained per physical component to serve as the differentiable
    spectral decoder M(θ) in SUSHI
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Framework used to code and train the IAE networks, enabling auto-differentiation
    of the decoder with respect to spectral parameters θ
- name: Python 3.10
  role: Implementation language; environment specified via SUSHI_env.yml conda file
- name: XSpec
  role: Look-up-table-based physical model (in Sherpa) used to generate training spectra;
    a call to the trained IAE decoder D is over 60 times faster
- name: Sherpa
  role: Hosts the XSpec physical model baseline that the IAE surrogate replaces for
    speed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:6fe355b80459c348f0fff337a52e6737fe3eb2c32ddae04edd4c26f5d25c79e1
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
  - Sherpa
  - XSpec
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# iae-spectral-surrogate-modeling

## Summary

Trains an Interpolatory Auto-Encoder (IAE) as a fast, differentiable surrogate of a physical spectral model (e.g. APEC thermal or power-law synchrotron emission convolved with the Chandra ARF/RMF response), enabling gradient-based non-stationary unmixing in SUSHI. The learned decoder is over 60x faster to evaluate than look-up-table-based models in Sherpa/XSpec and supports auto-differentiation of spectral parameters.

## When to use

Use when a physical spectral model (e.g. APEC via XSpec look-up tables) is too slow or non-differentiable for per-pixel hyperspectral fitting, and you need a learned, differentiable spectral decoder for components like thermal and synchrotron emission.

## When NOT to use

- You only need a single fixed spectrum per component — classic per-pixel 1D fits or stationary unmixing suffice without training a surrogate
- Your component's spectrum has no available physical or simulation-based model to generate training spectra
- You need exact physical parameter outputs from the unmixing — SUSHI's fitted unmixed spectra do not directly provide physical parameters

## Inputs

- Large set (>=1e3) of spectra simulated from the physical model, as numpy arrays of shape (N, n_E) with n_E spectral channels
- Boundary/anchor spectra at extreme parameter values (e.g. highest and lowest temperature), Output['Psi']
- Training/validation split (70%/30%) of the remaining spectra (Output['X_train'], Output['X_valid'])
- Chandra instrumental response (ARF and RMF files) for convolving model spectra
- APEC model spectra (AtomDB, http://www.atomdb.org/physics.php) for the thermal component

## Outputs

- A trained IAE decoder per physical component (thermal APEC-based and power-law synchrotron), stored as named model files in the 'IAE models' directory
- Training cost convergence curves for checking that the cost converged

## How to apply

Build a training set of at least ~1e3 spectra from the physical model (e.g. APEC from AtomDB convolved with the Chandra instrumental response ARF and RMF files), stored as numpy arrays of shape (N, n_E). Select spectra at boundary parameter values (e.g. highest and lowest temperature) as anchor points (Output['Psi']), shuffle the rest and split 70/30 into training (Output['X_train']) and validation (Output['X_valid']) sets. Train one IAE per physical component using the 'Training an IAE.ipynb' notebook, optionally weighting the cost by the mean (cost_weight) for very dynamic ranges, and run three epochs (niters) with optimizers chosen from Optims = ['adam', 'momentum', 'rmsprop', 'adagrad', 'Nesterov', 'SGD'], per-epoch step sizes, and three-epoch restarts taking the previous training as first guess. After training, check that the cost successfully converged, then verify the model's projection behaves as a barycentric interpolation between anchor points before handing the decoder to SUSHI as the spectral model M(θ).

## Related tools

- **IAE (Interpolatory Auto-Encoder)** (Generative model trained per physical component to serve as the differentiable spectral decoder M(θ) in SUSHI) — https://github.com/jbobin/IAE
- **JAX** (Framework used to code and train the IAE networks, enabling auto-differentiation of the decoder with respect to spectral parameters θ)
- **Python 3.10** (Implementation language; environment specified via SUSHI_env.yml conda file)
- **XSpec** (Look-up-table-based physical model (in Sherpa) used to generate training spectra; a call to the trained IAE decoder D is over 60 times faster)
- **Sherpa** (Hosts the XSpec physical model baseline that the IAE surrogate replaces for speed)

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env  # then run 'Training an IAE.ipynb' with Output={'Psi':anchors,'X_train':train,'X_valid':valid}
```

## Evaluation signals

- Training cost function converges over the three epochs — 'check that the cost successfully converged'
- Training set size is at least 1e3 spectra, with anchor points covering the boundary parameter values
- Arrays for Output['Psi'], Output['X_train'], Output['X_valid'] all have shape (N, n_E) matching the number of spectral channels
- A call to the trained decoder D is roughly 60x faster than calling the XSpec look-up-table model in Sherpa
- Downstream SUSHI validation: average RMSE per spectrum of 0.19 (thermal) and 0.16 (synchrotron) on the toy model, better than the classic 1D fit (0.22/0.19)

## Limitations

- No software version specified for JAX and trained IAE weights are not explicitly shared (only code availability is stated); older_version directory contains IAE code for older JAX versions
- No random seeds or training hyperparameter randomness handling described for IAE training
- The IAE constrains solutions to the barycentric span of the anchor points, so anchor coverage of parameter boundaries determines the reachable spectral space
- Downstream, SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component because patches were too large for the spatial regularization to compensate

## Evidence

- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [readme] Provide a large number of spectra (1e3 at least) from your physical model.: "Provide a large number of spectra (1e3 at least) from your physical model. 
* Select the spectra that are at boundary parameters (e.g. highest and lowest temperature). These will be your anchor points"
- [readme] These three sets (anchor points, training, and validation) should be numpy arrays of shape (N, $n_E$), where N is the number of spectra, and $n_E$ is the number of spectral channels.: "These three sets (anchor points, training, and validation) should be numpy arrays of shape (N, $n_E$), where N is the number of spectra, and $n_E$ is the number of spectral channels."
- [readme] Once the training is complete (this may take a while), check that the cost successfully converged.: "Once the training is complete (this may take a while), check that the cost successfully converged."
- [methods] An IAE model may be trained for each physical component. The code to do so is openly accessible on Git Hub: "An IAE model may be trained for each physical component. The code to do so is openly accessible on Git Hub"
