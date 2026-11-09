---
name: poisson-likelihood-spectral-fitting
description: Use when the input is a photon-counting X-ray hyperspectral cube (e.g.
  Chandra ACIS-S, 375 channels from 3 keV to 8.48 keV) whose counts follow Poisson
  statistics, and you need per-pixel physical parameters (temperature, redshift, photon
  index) or unmixed component maps.
when_to_use_negative:
- When SUSHI's final output would be mistaken for physical parameters directly — it
  returns fitted unmixed spectra and does not directly provide physical parameters
- When the synchrotron (or any) component is weak relative to the thermal component
  and patches are large enough that spatial regularization cannot compensate — SUSHI
  failed to reconstruct the synchrotron component in this regime
- When no spectral model is available for the assumed physical components — SUSHI
  requires a fixed number of components, each with a (possibly non-analytical) spectral
  model
tools:
- name: SUSHI
  role: Implements the Poisson negative log-likelihood minimization with PALM, starlet-wavelet
    sparsity on θc, and non-negativity on Ac
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Provides the learned surrogate spectral model M(θ) whose calls are over 60×
    faster than XSpec look-up tables in Sherpa
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Framework in which SUSHI and the IAE models are coded and differentiated for
    gradient descent steps
  repo: https://github.com/jbobin/IAE
- name: Sherpa
  role: Hosts the classic per-pixel 1D spectral fit baseline (XSpec model) against
    which the Poisson-likelihood unmixing is compared
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:febf9166639eb693d80eb9c121677e1e100b340aecb91b148d09a88485b594c9
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
yes_rate: 0.8
n_claims: 10
---

# poisson-likelihood-spectral-fitting

## Summary

Fits X-ray hyperspectral (IFU) cubes by minimizing the Poisson negative log-likelihood over spectral-parameter and amplitude maps, as implemented in the SUSHI algorithm (Lascar, Bobin & Acero 2024), using PALM optimization with starlet-wavelet sparsity and non-negativity proximal steps.

## When to use

Use when the input is a photon-counting X-ray hyperspectral cube (e.g. Chandra ACIS-S, 375 channels from 3 keV to 8.48 keV) whose counts follow Poisson statistics, and you need per-pixel physical parameters (temperature, redshift, photon index) or unmixed component maps.

## When NOT to use

- When SUSHI's final output would be mistaken for physical parameters directly — it returns fitted unmixed spectra and does not directly provide physical parameters
- When the synchrotron (or any) component is weak relative to the thermal component and patches are large enough that spatial regularization cannot compensate — SUSHI failed to reconstruct the synchrotron component in this regime
- When no spectral model is available for the assumed physical components — SUSHI requires a fixed number of components, each with a (possibly non-analytical) spectral model

## Inputs

- Poisson-noised X-ray hyperspectral cube (e.g. 375 energy channels, 94×94 pixels, Chandra ACIS-S response-convolved)
- Pretrained IAE spectral models, one per physical component (e.g. APEC thermal, synchrotron power-law)
- Chandra instrumental response (ARF and RMF files) for building forward models

## Outputs

- Fitted unmixed spectral component cubes (per-component amplitude and parameter maps)
- Per-spectrum RMSE against ground truth or reference (e.g. sushi_vs_classic_rmse.csv comparison table)
- Physical-parameter maps (temperature, redshift, photon index)

## How to apply

Load the hyperspectral cube together with pretrained IAE (Interpolatory Auto-Encoder) spectral models — one per physical component (e.g. APEC thermal and synchrotron power-law) — since the trained IAE decoder is over 60 times faster than calling look-up-table models (XSpec in Sherpa). Run SUSHI (Algorithm 1) with PALM: (1) gradient descent on the parameter maps θc and amplitudes Ac to minimize the Poisson negative log-likelihood; (2) a sparsity proximal step on θc via soft thresholding of starlet (isotropic undecimated wavelet) coefficients with threshold factor k=1, ρ = k × 1.4826 × mad; (3) a non-negativity proximal step on Ac. Iterate until the relative cost fluctuation over the past fifty iterations falls below ε ≈ 1e-6 (max ~1e4 iterations). Judge success by computing per-spectrum RMSE against ground truth when available: on the Cassiopeia A-like toy model, SUSHI achieved 0.19 (thermal) and 0.16 (synchrotron) versus 0.22 and 0.19 for the classic pixel-by-pixel 1D fit, with temperature and photon-index residual standard deviations reduced by an order of magnitude.

## Related tools

- **SUSHI** (Implements the Poisson negative log-likelihood minimization with PALM, starlet-wavelet sparsity on θc, and non-negativity on Ac) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Provides the learned surrogate spectral model M(θ) whose calls are over 60× faster than XSpec look-up tables in Sherpa) — https://github.com/jbobin/IAE
- **JAX** (Framework in which SUSHI and the IAE models are coded and differentiated for gradient descent steps) — https://github.com/jbobin/IAE
- **Sherpa** (Hosts the classic per-pixel 1D spectral fit baseline (XSpec model) against which the Poisson-likelihood unmixing is compared)

## Examples

```
conda env create -f SUSHI_env.yml && conda activate SUSHI_env && jupyter notebook "SUSHI Test Notebook.ipynb"
```

## Evaluation signals

- Average RMSE per spectrum should reach ~0.19 (thermal) and ~0.16 (synchrotron) on the Cassiopeia A-like toy model, beating the classic 1D fit's 0.22 and 0.19
- Optimization stops only when the relative cost fluctuation over the past fifty iterations falls below ε ≈ 1e-6 (cap ~1e4 iterations)
- Amplitude maps Ac must remain non-negative after the proximal step at every iteration
- On the toy model, residual standard deviations for temperature and photon index should be an order of magnitude lower than the classic pixel-by-pixel method
- Thermal-amplitude accuracy should be comparable (not worse) than the classic method, since the two methods tie on that parameter

## Limitations

- SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component, because patches were too large for the spatial regularization to compensate
- The IAE decoder is a surrogate: SUSHI's output is fitted unmixed spectra and does not directly provide physical parameters
- No license stated for the SUSHI repository, and trained IAE weights are not explicitly stated as shared (only code availability is given)

## Evidence

- [methods] PALM optimization structure: [evidence span withheld — anchor status: not_found]
- [methods] Gradient step on parameters: [evidence span withheld — anchor status: not_found]
- [methods] Sparsity proximal step: [evidence span withheld — anchor status: not_found]
- [results] RMSE benchmark values: "the average root mean squared error for each spectra was found to be 0.19 for the thermal and 0.16 for the synchrotron, while for the classic method it was 0.22 for the thermal and 0.19 for the"
- [readme] Environment setup: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
