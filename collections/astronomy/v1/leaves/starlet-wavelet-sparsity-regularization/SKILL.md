---
name: starlet-wavelet-sparsity-regularization
description: Use when fitting spectral parameters pixel-by-pixel on hyperspectral
  (IFU) data leaves noisy, ill-posed parameter maps that need spatial regularization
  to stay physically meaningful at fine scales.
when_to_use_negative:
- The parameter maps are not expected to be sparse in the starlet domain (e.g. highly
  structured extended targets where the patches are too large for the regularization
  to compensate, as happened for weak synchrotron emission in Cas A)
- The data reduction goal is a plain per-spectrum 1D fit with no spatial coupling,
  where the wavelet proximal step would bias genuinely pixel-local solutions
tools:
- name: SUSHI
  role: Implements the starlet-transform l1 proximal step on theta_c inside its PALM
    unmixing loop (SUSHI.py, Python 3.10)
  repo: https://github.com/jmlascar/SUSHI
- name: JAX
  role: Auto-differentiates the IAE decoder to provide the gradient on theta_c that
    precedes the wavelet soft-thresholding step
- name: IAE (Interpolatory Auto-Encoder)
  role: Supplies the learned spectral model whose latent parameter maps theta_c are
    the target of the starlet sparsity regularization
  repo: https://github.com/jbobin/IAE
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:89cfc165f3e812622e0eb20ae2356eea8a8b0b5cae138c8ee9fadbae30607335
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
yes_rate: 0.8
n_claims: 5
---

# starlet-wavelet-sparsity-regularization

## Summary

Spatial regularization of latent spectral parameter maps in the SUSHI non-stationary unmixing loop: the starlet (isotropic undecimated) wavelet transform of each parameter map theta_c is soft-thresholded with an MAD-based l1 proximal step, constraining the ill-posed per-pixel spectral fitting.

## When to use

Use when fitting spectral parameters pixel-by-pixel on hyperspectral (IFU) data leaves noisy, ill-posed parameter maps that need spatial regularization to stay physically meaningful at fine scales.

## When NOT to use

- The parameter maps are not expected to be sparse in the starlet domain (e.g. highly structured extended targets where the patches are too large for the regularization to compensate, as happened for weak synchrotron emission in Cas A)
- The data reduction goal is a plain per-spectrum 1D fit with no spatial coupling, where the wavelet proximal step would bias genuinely pixel-local solutions

## Inputs

- Latent spectral parameter maps theta_c (one per physical component, e.g. APEC thermal and synchrotron power-law IAE components), each holding per-pixel anchor weights of the IAE decoder
- Starlet wavelet transform with J scales
- Threshold factor k (k = 1 in the paper) and MAD-based noise estimate per fine-scale coefficient set

## Outputs

- Sparsified latent parameter maps theta_c with fine-scale starlet coefficients soft-thresholded, reconstructed in pixel space
- Downstream: physically interpretable parameter maps (e.g. crab_photon_index_map.fits, Cas A thermal/synchrotron maps) once decoded through the IAE

## How to apply

Within the SUSHI PALM loop, after each gradient-descent step on theta_c against the negative Poisson log-likelihood, apply an l1 proximal step on the wavelet transform of the parameter map: transform theta_c into J starlet scales (the isotropic undecimated wavelet transform of Starck & Murtagh 1994), soft-threshold each fine-scale coefficient set with rho_j = k x 1.4826 x MAD(w_j) using threshold factor k = 1 (MAD = 1.4826 x median(|X - median(X)|)), then inverse-transform to reconstruct theta_c close to its previous value. The amplitude maps A_c are NOT thresholded — they receive a separate non-negativity proximal step instead. Iterate over all components until the relative cost fluctuation over the past fifty iterations falls below epsilon ~ 1e-6 (max ~1e4 iterations). This sparsity constraint is what lets SUSHI recover reliable temperature, redshift, and photon-index maps at small scales without spatial rebinning, and it reduced temperature and photon-index residual standard deviations by an order of magnitude versus classic 1D fits on the Cas A toy model.

## Related tools

- **SUSHI** (Implements the starlet-transform l1 proximal step on theta_c inside its PALM unmixing loop (SUSHI.py, Python 3.10)) — https://github.com/jmlascar/SUSHI
- **JAX** (Auto-differentiates the IAE decoder to provide the gradient on theta_c that precedes the wavelet soft-thresholding step)
- **IAE (Interpolatory Auto-Encoder)** (Supplies the learned spectral model whose latent parameter maps theta_c are the target of the starlet sparsity regularization) — https://github.com/jbobin/IAE

## Evaluation signals

- Each fine-scale threshold equals rho_j = k x 1.4826 x MAD(w_j) with k = 1; verify MAD is computed as 1.4826 x median(|X - median(X)|) per wavelet scale
- Amplitude maps A_c remain non-negative and untouched by the thresholding (only theta_c is wavelet-regularized)
- The PALM loop still reaches the stopping criterion of Eq. 18 (average relative cost fluctuation over 50 iterations < epsilon = 1e-6) or the ~1e4 iteration cap
- On the Cas A toy model, regularized results should show temperature and photon-index residual standard deviations reduced by an order of magnitude versus classic pixel-by-pixel 1D fits (RMSE 0.19 thermal / 0.16 synchrotron vs 0.22 / 0.19)
- Reconstructed maps must preserve the spatial morphology of the reference component maps (e.g. Picquenot et al. 2019 Cas A maps)

## Limitations

- SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component because the patches were large enough that the spatial regularization could not compensate
- The regularization assumes sparsity of the IAE parameter maps in the starlet domain; a threshold factor k = 1 is the paper's setting and other values are not validated here

## Evidence

- [methods] we used the isotropic undecimated wavelet transform introduced in Starck & Murtagh (1994), dubbed "starlet transform": "we used the isotropic undecimated wavelet transform introduced in Starck & Murtagh (1994), dubbed "starlet transform""
- [methods] Then, a proximal step to minimize the norm of W(θc) while remaining close to the previously found value of θc.: [evidence span withheld — anchor status: not_found]
- [abstract] We achieve this by training a spectral model and applying a spatial regularization constraint on its parameters.: "We achieve this by training a spectral model and applying a spatial regularization constraint on its parameters."
- [discussion] since the patches were large enough that the spatial regularization could not go around this difficulty.: "since the patches were large
enough that the spatial regularization could not go around this
difficulty."
