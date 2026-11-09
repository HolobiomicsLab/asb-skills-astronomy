---
name: palm-proximal-optimization
description: Use when unmixing a hyperspectral (IFU) data cube into a fixed number
  of physical components whose spectra are provided by a differentiable learned model
  (e.g. an IAE decoder), and both amplitude and spectral shape must vary spatially
  under Poisson noise.
when_to_use_negative:
- The data are not a multi-component hyperspectral cube — e.g. a single-component
  source like the Crab Nebula synchrotron emission needs no alternating unmixing loop
- No differentiable spectral model (e.g. trained IAE decoder) exists for the components;
  the gradient step on θ_c requires auto-differentiation of M(θ)
- The number of physical components cannot be fixed in advance, since SUSHI/PALM assumes
  a fixed component set
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: SUSHI
  role: Implements Algorithm 1 (PALM loop with gradient descent and proximal steps)
    for non-stationary unmixing of the data cube
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Differentiable spectral model M(θ) whose decoder is auto-differentiated to
    provide gradients ∂M/∂θ in the θ_c update
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Backend for auto-differentiating the IAE decoder during the gradient-descent
    step on θ_c
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:60241936a235916fae0d7e0c43d4cf15d7980ed7d9e0cc47707d1c4cbef28e89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - SUSHI
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# palm-proximal-optimization

## Summary

Implements the proximal alternating linearized minimization (PALM) loop from Bolte et al. (2014) as used in SUSHI to jointly estimate non-stationary amplitude maps A_c and spectral parameter maps θ_c from a Poisson-likelihood hyperspectral cube, alternating gradient-descent and proximal (sparsity, non-negativity) steps.

## When to use

Use when unmixing a hyperspectral (IFU) data cube into a fixed number of physical components whose spectra are provided by a differentiable learned model (e.g. an IAE decoder), and both amplitude and spectral shape must vary spatially under Poisson noise.

## When NOT to use

- The data are not a multi-component hyperspectral cube — e.g. a single-component source like the Crab Nebula synchrotron emission needs no alternating unmixing loop
- No differentiable spectral model (e.g. trained IAE decoder) exists for the components; the gradient step on θ_c requires auto-differentiation of M(θ)
- The number of physical components cannot be fixed in advance, since SUSHI/PALM assumes a fixed component set

## Inputs

- Hyperspectral (IFU) data cube with Poisson counts (e.g. Chandra ACIS-S merged cube of Cassiopeia A, 980 ks, 2004)
- Pre-trained IAE decoder model per physical component (differentiable, JAX), e.g. APEC-based thermal and power-law synchrotron decoders
- Number of components n_C, number of starlet wavelet scales J, threshold scaling k, step-size parameters, stopping tolerance ε

## Outputs

- Per-component fitted cubes X̂_c = A_c ⊙ M_c(θ_c) and their sum X̂ (total fit)
- Non-negative amplitude maps A_c and spectral parameter maps θ_c (e.g. temperature, synchrotron amplitude) per component

## How to apply

Initialize each component's amplitude map A_c as the per-pixel summed counts divided by the number of components n_C, and each latent parameter map θ_c uniformly as 1/N_A per anchor weight. In the PALM loop (Bolte et al. 2014), for each component: (1) take a gradient-descent step on θ_c minimizing the negative Poisson log-likelihood with fixed step size α_θc = 0.1/max(A0_c), computing ∂M/∂θ via auto-differentiation of the IAE decoder; (2) apply the sparsity proximal step by starlet-transforming (isotropic undecimated wavelet, J scales) the θ maps and soft-thresholding fine-scale coefficients with ρ_j = k × 1.4826 × MAD(w_j), k = 1; (3) take a gradient-descent step on A_c with inverse-diagonal-Hessian step size from the explicit Poisson Hessian (Eqs. 16–17), then zero negative amplitudes (non-negativity proximal step). Iterate over all components until the stopping criterion of Eq. 18 — average relative cost fluctuation over the past fifty iterations < ε (e.g. 10^-6) — or ~10^4 maximum iterations, then compute per-component fitted cubes X̂_c = A_c⊙M_c(θ_c).

## Related tools

- **SUSHI** (Implements Algorithm 1 (PALM loop with gradient descent and proximal steps) for non-stationary unmixing of the data cube) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Differentiable spectral model M(θ) whose decoder is auto-differentiated to provide gradients ∂M/∂θ in the θ_c update) — https://github.com/jbobin/IAE
- **JAX** (Backend for auto-differentiating the IAE decoder during the gradient-descent step on θ_c)

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env; jupyter notebook "SUSHI Test Notebook.ipynb"
```

## Evaluation signals

- Convergence: average relative cost-function fluctuation over the past fifty iterations falls below ε = 10^-6 (Eq. 18), or ~10^4 maximum iterations reached
- Output amplitude maps A_c are non-negative (negative values zeroed at the proximal step)
- Fitted per-component cubes X̂_c exist for every component c and their sum reproduces the total fit X̂
- Recovered component maps match reference morphology (e.g. SUSHI's Cas A thermal and synchrotron maps are consistent with the stationary unmixing of Picquenot et al. 2019)
- Accuracy benchmark: average RMSE per spectrum of ~0.19 (thermal) and ~0.16 (synchrotron), better than the classic per-pixel 1D fit (0.22 / 0.19)

## Limitations

- SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component because patches were too large for the spatial regularization to compensate
- Stopping criterion ε = 10^-6 and k = 1 MAD thresholding are defaults from the Cas A use case; other noise regimes may need retuning
- Trained IAE networks/weights are not stated as shared in the article (only code availability), so reproducing gradients requires retraining models

## Evidence

- [methods] a natural choice of optimization algorithm is the proximal alternating linearized minimization (PALM) algorithm Bolte et al. (2014): [evidence span withheld — anchor status: not_found]
- [methods] starlet transform soft thresholding: "we used the isotropic undecimated wavelet transform introduced in Starck & Murtagh (1994), dubbed "starlet transform""
- [methods] non-negativity proximal step: "Non-negativity proximal step"
- [abstract] sparsity constraint on wavelet transform of parameter maps: "We achieve this by training a spectral model and applying a spatial regularization constraint on its parameters."
- [readme] README environment setup: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
