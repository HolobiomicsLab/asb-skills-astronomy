---
name: auto-differentiation-of-neural-spectral-decoder
description: Use when fitting a learned (non-analytical) spectral model to hyperspectral/IFU
  data cubes and gradient descent on the model's latent parameters is needed inside
  a proximal optimization loop.
when_to_use_negative:
- The spectral model is analytical and its gradient is available in closed form (e.g.
  simple parametric fits), where a direct analytic derivative is simpler than a neural
  surrogate
- Only a single per-pixel 1D fit is needed without iterative spatial regularization,
  where the IAE/PALM machinery adds unnecessary overhead
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: JAX
  role: Backend providing auto-differentiation for computing ∂M/∂θ of the IAE decoder
    during the gradient-descent step on θ_c
- name: IAE (Interpolatory Auto-Encoder)
  role: Learned spectral decoder whose parameters are differentiated; code in IAE_JAX_v2_devl_2023.py
  repo: https://github.com/jbobin/IAE
- name: Sherpa
  role: XSpec look-up-table physical model baseline that the differentiable IAE decoder
    replaces (60x faster per call)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:25e4a6973142ef2b63a569f6cdfda4c310048f09d9bc3033f4d9d320fea9996d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jbobin/IAE
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - SUSHI
  - Sherpa
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Auto-differentiation of neural spectral decoder

## Summary

Uses JAX auto-differentiation to compute the Jacobian ∂M/∂θ of a trained Interpolatory Auto-Encoder (IAE) spectral decoder, enabling gradient-descent steps on the latent spectral parameter maps θ_c inside SUSHI's PALM unmixing loop against the Poisson negative log-likelihood.

## When to use

Use when fitting a learned (non-analytical) spectral model to hyperspectral/IFU data cubes and gradient descent on the model's latent parameters is needed inside a proximal optimization loop.

## When NOT to use

- The spectral model is analytical and its gradient is available in closed form (e.g. simple parametric fits), where a direct analytic derivative is simpler than a neural surrogate
- Only a single per-pixel 1D fit is needed without iterative spatial regularization, where the IAE/PALM machinery adds unnecessary overhead

## Inputs

- Trained IAE decoder D(θ) for one physical component (e.g. APEC thermal emission or synchrotron spectrum)
- Latent parameter maps θ_c (anchor weights, initialized uniformly as 1/N_A per anchor weight)
- Current amplitude maps A_c (initialized as per-pixel count sum divided by n_C)

## Outputs

- Gradient ∂M/∂θ evaluated per pixel for the θ_c update step
- Updated latent parameter maps θ_c after gradient descent

## How to apply

Load the pretrained IAE decoder for the physical component (e.g. APEC thermal model convolved with the Chandra instrumental response) whose latent parameter maps θ_c are being fitted. During each PALM iteration, apply a gradient-descent step on θ_c that minimizes the negative Poisson log-likelihood, computing the partial derivative ∂M/∂θ via auto-differentiation with a fixed step size α_θ = 0.1/max(A_0). The learned decoder is preferred over the look-up-table physical model (XSpec in Sherpa) because a call to the IAE decoder D is over 60 times faster and is differentiable, which look-up-table interpolation is not. Gradients feed the subsequent sparsity proximal step (starlet wavelet soft-thresholding) on θ_c. JAX supplies the automatic differentiation backend; the IAE code is provided in IAE_JAX_v2_devl_2023.py.

## Related tools

- **JAX** (Backend providing auto-differentiation for computing ∂M/∂θ of the IAE decoder during the gradient-descent step on θ_c)
- **IAE (Interpolatory Auto-Encoder)** (Learned spectral decoder whose parameters are differentiated; code in IAE_JAX_v2_devl_2023.py) — https://github.com/jbobin/IAE
- **Sherpa** (XSpec look-up-table physical model baseline that the differentiable IAE decoder replaces (60x faster per call))

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env; jupyter notebook 'SUSHI Test Notebook.ipynb'
```

## Evaluation signals

- The negative Poisson log-likelihood decreases monotonically across θ_c gradient-descent steps
- Total average cost fluctuation over the past fifty iterations falls below ε (e.g. 1e-6) triggering the stopping criterion of Equation 18
- Gradient evaluation runs end-to-end in the SUSHI Test Notebook without error in the SUSHI_env conda environment (Python 3.10)
- Resulting θ_c maps remain within the barycentric span defined by the IAE anchor weights

## Limitations

- The IAE surrogate approximates the physical model, so gradients are those of the surrogate rather than the exact physical spectrum
- No software version specified for JAX used to code and train the IAE models
- Trained IAE networks/weights are not stated as shared (only code availability is given)

## Evidence

- [methods] Gradient descent step on θc against the negative Poisson log-likelihood with fixed step size α_θ = 0.1/max(A_0), computing ∂M/∂θ via auto-differentiation: [evidence span withheld — anchor status: not_found]
- [methods] IAE decoder speed advantage over look-up tables: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [intro] Trained IAE spectral model presented in the paper: "we present our trained spectral model, an Interpolatory Auto-Encoder (IAE)"
- [readme] JAX used for IAE code: "IAE_JAX_v2_devl_2023.py: Code for the Interpolary Auto-Encoder (IAE), taken from: https://github.com/jbobin/IAE"
