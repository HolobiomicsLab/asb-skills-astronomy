---
name: soft-thresholding-proximal-operator-implementation
description: Use when a PALM or other proximal-splitting optimizer needs an l1 proximal
  step on wavelet coefficients of spatial parameter maps to regularize them while
  staying close to the previous iterate.
when_to_use_negative:
- The data/model parameters have no spatial structure to exploit (e.g. per-pixel 1D
  fits or non-imaging spectra), since the regularizer acts on spatial wavelet coefficients.
- Components are weak relative to overlapping components at large patch scales — the
  article reports SUSHI fails to reconstruct the synchrotron component where it is
  weak relative to the thermal component because patches were too large for the spatial
  regularization to compensate.
edam_operation: http://edamontology.org/operation_3720
edam_topics:
- http://edamontology.org/topic_3344
tools:
- name: SUSHI
  role: Contains Algorithm 1, which applies the sparsity proximal step (soft-thresholding
    of starlet wavelet coefficients of θ_c) inside the PALM loop
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Learned spectral model whose latent anchor-weight maps θ_c are the target
    of the soft-thresholding regularizer
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Autodifferentiation backend used for the SUSHI/IAE code in which the proximal
    step is embedded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:d6fbdcf2a45d33aff5b76e0156f4808ed6e92fe4798dfd719c90754b54ec77bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - SUSHI
  edam_operation: http://edamontology.org/operation_3720
  edam_topics:
  - http://edamontology.org/topic_3344
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# soft-thresholding-proximal-operator-implementation

## Summary

Implements the l1 sparsity proximal step used in SUSHI's PALM loop: starlet-transform a component's latent parameter maps θ_c into J wavelet scales, then soft-threshold each fine-scale coefficient set with thresholds ρ_j = k × mad(w_j) (mad = 1.4826 × median(|X − median(X)|)), using k = 1, to impose spatial regularization on spectral parameters.

## When to use

Use when a PALM or other proximal-splitting optimizer needs an l1 proximal step on wavelet coefficients of spatial parameter maps to regularize them while staying close to the previous iterate.

## When NOT to use

- The data/model parameters have no spatial structure to exploit (e.g. per-pixel 1D fits or non-imaging spectra), since the regularizer acts on spatial wavelet coefficients.
- Components are weak relative to overlapping components at large patch scales — the article reports SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component because patches were too large for the spatial regularization to compensate.

## Inputs

- Latent parameter maps θ_c for component c (per-pixel anchor weights of the IAE decoder), shape (N_A, image rows, image cols)
- Number of starlet wavelet scales J
- Threshold scaling factor k (k = 1 in the SUSHI notebook)

## Outputs

- Spatially regularized parameter maps θ_c after the proximal step, ready for the A_c gradient-descent update

## How to apply

After the gradient-descent step on θ_c against the negative Poisson log-likelihood, transform each component's parameter maps with the isotropic undecimated wavelet transform ('starlet transform', Starck & Murtagh 1994) into J scales. For each fine-scale coefficient set w_j, estimate the noise scale as mad(w_j) = 1.4826 × median(|X − median(X)|), set the threshold ρ_j = k × mad(w_j) with k = 1, and apply soft-thresholding (shrink coefficients toward zero by ρ_j) to minimize the norm of W(θ_c) while remaining close to the previously found θ_c. Then inverse-transform the thresholded coefficients back to the spatial domain before the subsequent amplitude update. In the SUSHI implementation this step appears in Algorithm 1 between the θ_c gradient step and the A_c gradient step, with the per-pixel step size α_A = 1/H_Ac taken from the explicit Poisson Hessian diagonal (Equation 16).

## Related tools

- **SUSHI** (Contains Algorithm 1, which applies the sparsity proximal step (soft-thresholding of starlet wavelet coefficients of θ_c) inside the PALM loop) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Learned spectral model whose latent anchor-weight maps θ_c are the target of the soft-thresholding regularizer) — https://github.com/jbobin/IAE
- **JAX** (Autodifferentiation backend used for the SUSHI/IAE code in which the proximal step is embedded)

## Examples

```
w = starlet_transform(theta_c, J); for j, wj in enumerate(w[:-1]): mad = 1.4826*np.median(np.abs(wj-np.median(wj))); w[j] = np.sign(wj)*np.maximum(np.abs(wj)-1.0*mad, 0)
```

## Evaluation signals

- The starlet transform of the parameter maps yields J scales with fine-scale coefficient sets that are each soft-thresholded by ρ_j = k × mad(w_j) with mad = 1.4826 × median(|X − median(X)|) and k = 1.
- After the proximal step, the parameter maps remain close to the pre-proximal value (the step minimizes the norm of W(θ_c) while remaining close to the previously found θ_c).
- The full SUSHI Test Notebook executes end-to-end and the PALM loop reaches the stopping criterion of Equation 18 (average cost fluctuation over the past fifty iterations < ε, e.g. 1e-6) or the ~1e4 iteration cap.
- Resulting unmixed cubes X̂_c exist for every component c and amplitude maps are non-negative after the non-negativity proximal step that follows the amplitude update.
- On the toy model, reconstructed physical-parameter maps should beat the classic per-pixel 1D fit baseline (SUSHI average RMSE per spectrum 0.19 thermal / 0.16 synchrotron vs 0.22 / 0.19 classic).

## Limitations

- Regularization fails when patches are too large: SUSHI could not reconstruct the synchrotron component where it was weak relative to the thermal component because patches were large enough that the spatial regularization could not compensate.
- The mad-based threshold assumes fine-scale wavelet coefficients are dominated by noise; choosing k other than the paper's k = 1 changes the sparsity/likelihood trade-off without guidance from the article.
- SUSHI assumes a fixed number of physical components, each with an available (possibly non-analytical) spectral model; the proximal step only makes sense as part of that semi-blind setup.

## Evidence

- [methods] Sparsity proximal step on θc (soft thresholding of starlet wavelet coefficients): [evidence span withheld — anchor status: not_found]
- [methods] Starlet transform used for regularization: "we used the isotropic undecimated wavelet transform introduced in Starck & Murtagh (1994), dubbed "starlet transform""
- [readme] Spatial regularization purpose: "In order to obtain more robust results on voxels with low signal to noise ratio, a spatial regularization is applied."
- [methods] PALM as the optimization framework: [evidence span withheld — anchor status: not_found]
