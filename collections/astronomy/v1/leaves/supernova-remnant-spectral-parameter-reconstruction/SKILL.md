---
name: supernova-remnant-spectral-parameter-reconstruction
description: Use when a Poisson-noised IFU/hyperspectral cube (e.g. Chandra X-ray)
  with a known number of physical components must have temperature, redshift, and
  photon index maps reconstructed and benchmarked against per-pixel 1D fits.
when_to_use_negative:
- The data contain a variable or unknown number of physical components, or no spectral
  model (analytical or learned) is available for at least one component — the paper
  states SUSHI is applicable only where 'a fixed number of physical components can
  be assumed and a spectral model for each of those components is available'
- The input is already a fitted parameter map or feature table rather than a raw hyperspectral/IFU
  data cube
- A unique spectrum per component is a valid assumption and stationary unmixing or
  simple per-pixel fitting suffices — SUSHI's spatial regularization is designed for
  non-stationary spectral variation and low-SNR voxels
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SUSHI
  role: Core unmixing algorithm (Algorithm 1) that reconstructs the component cubes
    via PALM minimization of the Poisson negative log-likelihood with starlet sparsity
    on theta_c
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Learned spectral surrogate model per physical component; provides the fast
    decoder D (over 60x faster than the XSpec/Sherpa look-up-table model) used inside
    SUSHI's optimization
  repo: https://github.com/jbobin/IAE
- name: Sherpa
  role: Hosts the XSpec physical model for the classic pixel-by-pixel 1D spectral
    fit baseline (no spatial regularization)
- name: XSpec
  role: Look-up-table-based physical emission model (e.g. APEC) called through Sherpa
    for the baseline fits
- name: JAX
  role: Autodiff/JIT backend in which SUSHI and the IAE decoders are implemented for
    gradient-based PALM optimization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:2be407c3b10503cca2440614811f05d2151a2e7b1c3c6b7431498eea20cc191c
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
  - XSpec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# supernova-remnant-spectral-parameter-reconstruction

## Summary

Reproduces the SUSHI paper's toy-model benchmark on a simulated Cassiopeia A-like X-ray hyperspectral cube, comparing SUSHI's semi-blind non-stationary unmixing (IAE spectral models + PALM optimization with starlet sparsity) against a classic per-pixel 1D spectral fit to quantify per-spectrum RMSE of reconstructed physical parameters.

## When to use

Use when a Poisson-noised IFU/hyperspectral cube (e.g. Chandra X-ray) with a known number of physical components must have temperature, redshift, and photon index maps reconstructed and benchmarked against per-pixel 1D fits.

## When NOT to use

- The data contain a variable or unknown number of physical components, or no spectral model (analytical or learned) is available for at least one component — the paper states SUSHI is applicable only where 'a fixed number of physical components can be assumed and a spectral model for each of those components is available'
- The input is already a fitted parameter map or feature table rather than a raw hyperspectral/IFU data cube
- A unique spectrum per component is a valid assumption and stationary unmixing or simple per-pixel fitting suffices — SUSHI's spatial regularization is designed for non-stationary spectral variation and low-SNR voxels

## Inputs

- Toy-model hyperspectral cube: 375 energy channels (3-8.48 keV), 94x94 pixels, Poisson-noised (based on the Orlando et al. 2016 SNR simulation convolved with Chandra ARF/RMF responses)
- Pretrained IAE spectral models for the APEC thermal component and the synchrotron power-law component (from the SUSHI repo's 'IAE models' directory)
- Ground-truth component/parameter cubes for the toy model

## Outputs

- Reconstructed thermal and synchrotron component cubes from SUSHI
- Reconstructed component cubes from the classic pixel-by-pixel 1D fit
- Comparison table sushi_vs_classic_rmse.csv with per-spectrum RMSE for each component and method

## How to apply

Create the conda environment from SUSHI_env.yml (Python 3.10 with JAX) per the README. Load the toy-model cube (375 energy channels between 3 keV and 8.48 keV, 94x94 pixels, Poisson-noised at a regime similar to the 2004 980 ks Cas A Chandra observation) and the pretrained IAE models for the APEC thermal component and the synchrotron power-law. Run SUSHI (Algorithm 1) minimizing the Poisson negative log-likelihood with PALM: gradient descent on theta_c and A_c, a starlet-wavelet l1 sparsity proximal step on theta_c with threshold factor k=1 (rho = k x 1.4826 x mad), and a non-negativity proximal step on A_c, stopping when the relative cost fluctuation over the past fifty iterations falls below epsilon ~ 1e-6 (max ~1e4 iterations). In parallel, run the classic pixel-by-pixel 1D spectral fit (XSpec model in Sherpa) with no spatial regularization on the same cube. Compute the average root mean squared error per spectrum against ground truth for each component and method, save reconstructed cubes and the comparison table to sushi_vs_classic_rmse.csv.

## Related tools

- **SUSHI** (Core unmixing algorithm (Algorithm 1) that reconstructs the component cubes via PALM minimization of the Poisson negative log-likelihood with starlet sparsity on theta_c) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Learned spectral surrogate model per physical component; provides the fast decoder D (over 60x faster than the XSpec/Sherpa look-up-table model) used inside SUSHI's optimization) — https://github.com/jbobin/IAE
- **Sherpa** (Hosts the XSpec physical model for the classic pixel-by-pixel 1D spectral fit baseline (no spatial regularization))
- **XSpec** (Look-up-table-based physical emission model (e.g. APEC) called through Sherpa for the baseline fits)
- **JAX** (Autodiff/JIT backend in which SUSHI and the IAE decoders are implemented for gradient-based PALM optimization)

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env
```

## Evaluation signals

- Average per-spectrum RMSE for SUSHI must be lower than the classic method and match the paper's reported values: 0.19 thermal / 0.16 synchrotron (SUSHI) vs 0.22 / 0.19 (classic)
- Temperature and photon index residual standard deviations should be reduced by roughly an order of magnitude relative to the classic per-pixel fit
- Thermal amplitude RMSE should be at worst equal between SUSHI and the classic method (the one parameter with no improvement)
- PALM convergence check: relative cost fluctuation over the past fifty iterations falls below epsilon ~ 1e-6 within ~1e4 iterations
- Amplitude maps A_c remain non-negative throughout; sparsity threshold on starlet coefficients of theta_c uses rho = k x 1.4826 x mad with k = 1

## Limitations

- SUSHI's final output is fitted unmixed spectra and does not directly provide physical parameters (a follow-up conversion step is needed)
- SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component, because the patches were large enough that the spatial regularization could not compensate
- The toy model depends on the Orlando et al. (2016) simulation kindly provided by S. Orlando rather than a publicly hosted dataset link
- Trained IAE network weights are not explicitly stated as shared in the paper (only code availability); random seeds / training hyperparameter handling for IAE training is not described

## Evidence

- [results] the average root mean squared error for each spectra was found to be 0.19 for the thermal and 0.16 for the synchrotron, while for the classic method it was 0.22 for the thermal and 0.19 for the: "the average root mean squared error for each spectra was found to be 0.19 for the thermal and 0.16 for the synchrotron, while for the classic method it was 0.22 for the thermal and 0.19 for the"
- [methods] noise level regimes similar to the real data of Cassiopeia A observed in 2004 for 980 ks in the same energy band: [evidence span withheld — anchor status: not_found]
- [methods] a natural choice of optimization algorithm is the proximal alternating linearized minimization (PALM) algorithm Bolte et al. (2014): [evidence span withheld — anchor status: not_found]
- [readme] SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml.: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
- [discussion] SUSHI's final output is a fitted unmixed spectra, but it does not provide physical parameters: "SUSHI’s final output is a fitted unmixed spectra, but it does not provide physical parameters"
- [discussion] since the patches were large enough that the spatial regularization could not go around this difficulty: "since the patches were large
enough that the spatial regularization could not go around this
di ffi culty."
