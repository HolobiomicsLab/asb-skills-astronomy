---
name: poisson-likelihood-optimization
description: Use when fitting photon-counting (Poisson-noise) hyperspectral X-ray
  data cubes with fixed, learned per-component spectral models (e.g. IAE decoders)
  and you need pixel-wise amplitude maps and spatially regularized spectral parameters.
when_to_use_negative:
- Data with Gaussian (non-counting) noise statistics, where a Poisson log-likelihood
  is mismatched to the measurement process
- Problems where a fixed number of physical components cannot be assumed or no spectral
  model (analytical or learned IAE) is available for a component
- Cases where a component is very weak relative to another (e.g. synchrotron under
  bright thermal emission) and spatial regularization patches were too large to recover
  it — SUSHI's known failure mode
tools:
- name: SUSHI
  role: Implements the PALM loop with Poisson likelihood, starlet sparsity proximal,
    and amplitude Newton steps over a variable number of components
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Learned differentiable spectral decoder M(theta) whose auto-differentiated
    gradient drives the theta_c descent step
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Auto-differentiation backend for computing the negative Poisson log-likelihood
    gradient through the IAE decoder
- name: Sherpa
  role: Reference framework hosting the look-up-table XSpec physical model that the
    IAE surrogate replaces for speed
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:ac7e15f85b37d579ade15e9e7e742bea236df73861b39b354b69fc783fbee113
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - SUSHI
  - Sherpa
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# poisson-likelihood-optimization

## Summary

Alternating proximal optimization (PALM, Bolte et al. 2014) of per-component amplitudes and latent spectral parameters under a negative Poisson log-likelihood, as implemented in SUSHI's non-stationary unmixing of X-ray hyperspectral cubes with IAE spectral models.

## When to use

Use when fitting photon-counting (Poisson-noise) hyperspectral X-ray data cubes with fixed, learned per-component spectral models (e.g. IAE decoders) and you need pixel-wise amplitude maps and spatially regularized spectral parameters.

## When NOT to use

- Data with Gaussian (non-counting) noise statistics, where a Poisson log-likelihood is mismatched to the measurement process
- Problems where a fixed number of physical components cannot be assumed or no spectral model (analytical or learned IAE) is available for a component
- Cases where a component is very weak relative to another (e.g. synchrotron under bright thermal emission) and spatial regularization patches were too large to recover it — SUSHI's known failure mode

## Inputs

- Chandra ACIS hyperspectral counts cube (e.g. Cassiopeia A 2004 980 ks merged cube, 4.21-7.48 keV or 3.0-8.48 keV band)
- Pre-trained IAE (Interpolatory Auto-Encoder) decoder spectral models, one per physical component (APEC-based thermal, power-law synchrotron)
- Initial amplitude maps A0_c (per-pixel summed counts divided by number of components n_C)
- Initial latent parameter maps theta (uniform weights theta_i = 1/N_A per pixel)

## Outputs

- Per-component fitted cubes X̂_c = A_c ⊙ M_c(theta_c) and their total fitted sum
- Per-component amplitude maps A_c (non-negative)
- Per-component latent spectral parameter maps theta_c (convertible to photon-index maps via the IAE encoding)
- Converged cost-function trajectory for the stopping-criterion check

## How to apply

Use the proximal alternating linearized minimization (PALM) scheme: for each component c, take a gradient descent step on the latent parameters theta_c minimizing the negative Poisson log-likelihood, computing gradients by auto-differentiation of the IAE decoder with step size alpha_theta_c = 0.1/max(A0_c). Follow with a sparsity proximal step on theta_c: starlet (isotropic undecimated wavelet) transform with J scales, soft-threshold fine-scale coefficients at rho_j = k * 1.4826 * MAD(w_j) with k = 1, then reconstruct. Next, take a gradient step on the amplitude A_c with an inverse-diagonal-Hessian step size (Eqs. 16-17) and enforce non-negativity by zeroing negative amplitudes. Alternate over all components until the average relative cost fluctuation over the past fifty iterations falls below epsilon = 1e-6 (Eq. 18) or ~1e4 maximum iterations is reached. Auto-differentiation through the IAE decoder is what makes the theta gradient tractable; a call to the trained decoder D is over 60 times faster than the look-up-table physical model (XSpec model in Sherpa), enabling the per-iteration gradient evaluations.

## Related tools

- **SUSHI** (Implements the PALM loop with Poisson likelihood, starlet sparsity proximal, and amplitude Newton steps over a variable number of components) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Learned differentiable spectral decoder M(theta) whose auto-differentiated gradient drives the theta_c descent step) — https://github.com/jbobin/IAE
- **JAX** (Auto-differentiation backend for computing the negative Poisson log-likelihood gradient through the IAE decoder)
- **Sherpa** (Reference framework hosting the look-up-table XSpec physical model that the IAE surrogate replaces for speed)

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env; jupyter notebook SUSHI\ Test\ Notebook.ipynb
```

## Evaluation signals

- Stopping criterion satisfied: average relative cost-function fluctuation over the past fifty iterations < epsilon = 1e-6 (Eq. 18), typically within ~1e4 iterations
- Non-negativity invariant: all amplitude maps A_c contain no negative values after the projection step
- Reconstruction consistency: the sum of per-component fitted cubes X̂_c = A_c ⊙ M_c(theta_c) matches the input counts cube
- Recovered parameter maps (e.g. synchrotron photon index for the Crab) are consistent with prior analyses (Mori et al. 2004; Picquenot et al. 2019) while resolving finer spatial detail than per-pixel 1D fits
- Accuracy benchmark: average RMSE per spectrum ~0.19 (thermal) and ~0.16 (synchrotron), better than the classic 1D per-pixel fit (0.22 and 0.19)

## Limitations

- SUSHI fails to reconstruct a weak component (e.g. synchrotron) where it is weak relative to the thermal component because the regularization patches were too large for the spatial regularization to compensate
- The likelihood uses Poisson statistics for raw counts; background handling relies on an external background spectrum (e.g. Chandra blank-sky observations from the ACIS calibration team)
- No random seeds or handling of training-time randomness for the IAE models is described, which can affect reproducibility of the decoder gradients

## Evidence

- [methods] PALM algorithm with Poisson likelihood: [evidence span withheld — anchor status: not_found]
- [methods] theta gradient step: [evidence span withheld — anchor status: not_found]
- [methods] sparsity proximal step: [evidence span withheld — anchor status: not_found]
- [methods] starlet wavelet regularization: "we used the isotropic undecimated wavelet transform introduced in Starck & Murtagh (1994), dubbed "starlet transform""
- [methods] IAE decoder speed rationale: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [readme] low-SNR motivation for regularization: "In order to obtain more robust results on voxels with low signal to noise ratio, a spatial regularization is applied."
