---
name: poisson-likelihood-gradient-and-hessian-computation
description: Use when fitting physical component models to X-ray/IFU data cubes whose
  noise is Poisson-distributed (photon counts), and you need differentiable likelihood
  terms for a PALM-style optimization with explicit step sizes.
when_to_use_negative:
- Data whose noise is not Poisson-distributed (e.g. Gaussian-noise calibration or
  non-count data) — the likelihood, gradient, and Hessian diagonal forms would be
  wrong
- The spectral model is not differentiable with respect to its parameters (no ∂M/∂θ
  available via auto-differentiation)
- A per-pixel 1D fit suffices and no spatial coupling of parameters is desired — the
  classic per-pixel approach is the appropriate baseline
tools:
- name: SUSHI
  role: Hosts the PALM loop (Algorithm 1) in which the Poisson gradient-descent and
    Hessian-based steps are embedded
  repo: https://github.com/jmlascar/SUSHI
- name: JAX
  role: Auto-differentiation backend used to compute ∂M/∂θ through the IAE decoder
    for the likelihood gradient
  repo: https://github.com/jbobin/IAE
- name: IAE (Interpolatory Auto-Encoder)
  role: Differentiable learned spectral model M(θ) whose parameters enter the Poisson
    likelihood
  repo: https://github.com/jbobin/IAE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:9c6c94da9d95565d0f6d29f4c80124b810fcd80032413c2a352164c6e762eb7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# poisson-likelihood-gradient-and-hessian-computation

## Summary

Computes the gradients and per-pixel Hessian diagonal of the negative Poisson log-likelihood used inside SUSHI's PALM loop, driving the gradient-descent updates of the IAE spectral parameter maps θ_c and the amplitude maps A_c for hyperspectral (IFU) data cubes with photon-counting noise.

## When to use

Use when fitting physical component models to X-ray/IFU data cubes whose noise is Poisson-distributed (photon counts), and you need differentiable likelihood terms for a PALM-style optimization with explicit step sizes.

## When NOT to use

- Data whose noise is not Poisson-distributed (e.g. Gaussian-noise calibration or non-count data) — the likelihood, gradient, and Hessian diagonal forms would be wrong
- The spectral model is not differentiable with respect to its parameters (no ∂M/∂θ available via auto-differentiation)
- A per-pixel 1D fit suffices and no spatial coupling of parameters is desired — the classic per-pixel approach is the appropriate baseline

## Inputs

- Hyperspectral (IFU) data cube X with photon-count (Poisson) noise, e.g. Chandra ACIS-S observations
- Trained IAE spectral model decoder per physical component (differentiable, JAX-based)
- Initial amplitude map A_c (per-pixel count sum divided by n_C) and initial uniform latent parameter map θ_c (1/N_A per anchor weight)
- Predefined step-size hyperparameters: α_θ = 0.1/max(A_0), α_A = 1/H_Ac from Equation 16

## Outputs

- Updated spectral parameter maps θ_c (per component, per pixel)
- Updated non-negative amplitude maps A_c (per component, per pixel)
- Scalar negative-Poisson-log-likelihood cost value per iteration for convergence monitoring

## How to apply

For each component c, form the negative Poisson log-likelihood between the data cube X and the model prediction M(θ_c, A_c), where M is evaluated through the trained IAE decoder with auto-differentiation (JAX) supplying ∂M/∂θ. Take a gradient-descent step on θ_c with the fixed step size α_θ = 0.1/max(A_0) (scaled to the initial amplitude map A_0), while A_c is updated using a per-pixel step α_A = 1/H_Ac taken from the explicit Hessian diagonal of the Poisson likelihood (Equation 16), which makes the amplitude step size adapt to the local count level. The Poisson Hessian diagonal for amplitude pixels has the form H_Ac = Σ_E M_c(E)/σ_E² terms consistent with count-based variance, and the non-negativity proximal step (setting negative amplitudes to zero) must follow each A_c update. Iterate these steps per component until the stopping criterion of Equation 18 (average cost fluctuation over the last 50 iterations < ε, e.g. 1e-6) or ~1e4 maximum iterations. The rationale is that photon-count data demand Poisson statistics, and the explicit Hessian diagonal avoids line searches while the IAE surrogate keeps the model differentiable and over 60x faster than look-up-table (XSpec/Sherpa) evaluation.

## Related tools

- **SUSHI** (Hosts the PALM loop (Algorithm 1) in which the Poisson gradient-descent and Hessian-based steps are embedded) — https://github.com/jmlascar/SUSHI
- **JAX** (Auto-differentiation backend used to compute ∂M/∂θ through the IAE decoder for the likelihood gradient) — https://github.com/jbobin/IAE
- **IAE (Interpolatory Auto-Encoder)** (Differentiable learned spectral model M(θ) whose parameters enter the Poisson likelihood) — https://github.com/jbobin/IAE

## Evaluation signals

- Step sizes match the article: gradient step on θ_c uses fixed α_θ = 0.1/max(A_0); the A_c step uses the per-pixel Hessian diagonal α_A = 1/H_Ac from Equation 16
- The negative Poisson log-likelihood cost decreases monotonically enough that the Equation 18 criterion (average cost fluctuation over 50 iterations < ε, e.g. 1e-6) is reached within ~1e4 iterations
- After each A_c update, the non-negativity proximal step guarantees all amplitude-map pixels are ≥ 0
- Auto-differentiated ∂M/∂θ through the IAE decoder is consistent (e.g. spot-checkable against finite differences) and the model call is fast (the IAE decoder was over 60x faster than the XSpec/Sherpa look-up-table model)
- Fitted per-component cubes X̂_c sum to a total fit X̂ reproducing the input cube's count structure

## Limitations

- The θ_c step uses a fixed step size α_θ = 0.1/max(A_0), which may be suboptimal for components with very different amplitude scales
- The likelihood assumes Poisson photon statistics; it is not appropriate for Gaussian-noise regimes
- SUSHI's overall fit can fail to reconstruct weak components (e.g. synchrotron where weak relative to thermal) when spatial-regularization patches are too large — the likelihood step alone cannot compensate
- Computation of the explicit Hessian diagonal (Equation 16) is specific to the Poisson form and must be re-derived for other noise models

## Evidence

- [methods] PALM embedding of the likelihood steps: [evidence span withheld — anchor status: not_found]
- [methods] Gradient step on spectral parameters: [evidence span withheld — anchor status: not_found]
- [methods] Initialization of amplitude and parameter maps: [evidence span withheld — anchor status: not_found]
- [methods] Step-size definitions in the PALM loop: [evidence span withheld — anchor status: not_found]
- [methods] Hessian-based amplitude step and non-negativity: [evidence span withheld — anchor status: not_found]
- [methods] Speed advantage of the differentiable IAE surrogate: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [readme] Environment for running the algorithm: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
