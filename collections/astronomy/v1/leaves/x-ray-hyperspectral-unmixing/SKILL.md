---
name: x-ray-hyperspectral-unmixing
description: Use when analyzing Chandra/IFU hyperspectral data cubes where a fixed
  number of physical components can be assumed and each has a spectral model, and
  per-pixel 1D fits lack the spatial resolution or low-S/N robustness needed for parameter
  maps.
when_to_use_negative:
- When no fixed number of physical components can be assumed or no spectral model
  (learned or analytical) is available for each component
- When a component is weak relative to another and patch sizes prevent the spatial
  regularization from recovering it (SUSHI's documented failure mode on the weak synchrotron
  region)
- When the goal is a single stationary spectrum per component — classic source separation
  or per-pixel 1D fits suffice
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SUSHI
  role: 'Core algorithm: semi-blind unmixing with sparsity, coded in Python 3.10,
    minimizing Poisson negative log-likelihood via PALM'
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Learned surrogate spectral model M(θ) per physical component, used as the
    decoder differentiated inside the unmixing loop
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Auto-differentiation backend for the IAE decoder and SUSHI optimization steps
- name: Sherpa
  role: Classic per-pixel 1D spectral fit baseline (XSpec model in Sherpa) for comparison
- name: Python 3.10
  role: Runtime environment, dependencies installed from SUSHI_env.yml via conda
  repo: https://github.com/jmlascar/SUSHI
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:6bfed60e340d92f69cb85b1826aed609de93b2112998c4fc03e49e6722bd7148
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - Python 3.10
  - SUSHI
  - Sherpa
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# x-ray-hyperspectral-unmixing

## Summary

Non-stationary unmixing of X-ray hyperspectral (IFU) data cubes using SUSHI, which separates physical components (e.g. APEC thermal and synchrotron power-law) whose spectra vary spatially, with starlet-wavelet sparsity regularization of spectral parameters via a PALM optimization loop.

## When to use

Use when analyzing Chandra/IFU hyperspectral data cubes where a fixed number of physical components can be assumed and each has a spectral model, and per-pixel 1D fits lack the spatial resolution or low-S/N robustness needed for parameter maps.

## When NOT to use

- When no fixed number of physical components can be assumed or no spectral model (learned or analytical) is available for each component
- When a component is weak relative to another and patch sizes prevent the spatial regularization from recovering it (SUSHI's documented failure mode on the weak synchrotron region)
- When the goal is a single stationary spectrum per component — classic source separation or per-pixel 1D fits suffice

## Inputs

- Hyperspectral X-ray data cube (Chandra ACIS IFU cube, e.g. Cas A 2004 980 ks merged cube in 3.0–8.48 keV; Crab 2000–2001 ObsIDs 1994–2001)
- Pre-trained IAE decoder models, one per physical component (APEC-based thermal; synchrotron power-law)
- Number of components n_C (e.g. n_C = 2 for Cas A, n_C = 1 for the Crab Nebula)
- Optional background spectrum (e.g. Chandra ACIS calibration team blank-sky observations for the Crab)

## Outputs

- Per-component fitted cubes X̂c = Ac⊙Mc(θc) and their summed total fit
- Amplitude maps Ac and latent-parameter maps θc per component (e.g. temperature, redshift, photon index)
- Converted physical-parameter maps (e.g. crab_photon_index_map.fits with fitted synchrotron cube as auxiliary product)
- Comparison tables such as sushi_vs_classic_rmse.csv (per-spectrum RMSE against ground truth)

## How to apply

Load the hyperspectral cube (e.g. Cas A Chandra ACIS-S 2004 merged 980 ks cube, 3.0–8.48 keV) and a pre-trained Interpolatory Auto-Encoder (IAE) decoder per physical component (APEC thermal, synchrotron power-law). Initialize amplitude maps as per-pixel total counts divided by n_C components and latent parameters θ uniformly as 1/N_A. Run the PALM loop (Bolte et al. 2014): gradient descent on θc against the Poisson negative log-likelihood (step size α_θc = 0.1/max(A0_c), auto-differentiated through the IAE decoder), a starlet-wavelet l1 sparsity proximal step on θc with MAD-based soft threshold ρ = k × 1.4826 × MAD (k = 1), a gradient step on amplitudes Ac with inverse-diagonal-Hessian step size, and non-negativity projection. Iterate until the relative cost fluctuation over the past fifty iterations falls below ε = 1e-6 or ~1e4 max iterations, then compute fitted cubes X̂c = Ac⊙Mc(θc) and inspect parameter maps. On the Cas A toy model this reduces temperature and photon-index residual standard deviations by an order of magnitude versus a classic pixel-by-pixel XSpec/Sherpa fit, with per-spectrum RMSE of 0.19 (thermal) / 0.16 (synchrotron) versus 0.22 / 0.19 for the classic method.

## Related tools

- **SUSHI** (Core algorithm: semi-blind unmixing with sparsity, coded in Python 3.10, minimizing Poisson negative log-likelihood via PALM) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Learned surrogate spectral model M(θ) per physical component, used as the decoder differentiated inside the unmixing loop) — https://github.com/jbobin/IAE
- **JAX** (Auto-differentiation backend for the IAE decoder and SUSHI optimization steps)
- **Sherpa** (Classic per-pixel 1D spectral fit baseline (XSpec model in Sherpa) for comparison)
- **Python 3.10** (Runtime environment, dependencies installed from SUSHI_env.yml via conda) — https://github.com/jmlascar/SUSHI

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env; jupyter notebook SUSHI\ Test\ Notebook.ipynb
```

## Evaluation signals

- PALM loop terminates when the average relative cost fluctuation over the past fifty iterations drops below ε = 1e-6 (Eq. 18) or ~1e4 max iterations is reached
- On the toy model, per-spectrum RMSE for SUSHI should be 0.19 (thermal) and 0.16 (synchrotron), lower than the classic method's 0.22 and 0.19
- Reconstructed Cas A thermal and synchrotron maps must match the spatial morphology of the reference component maps used to build the validation data, and be consistent with the stationary unmixing of Picquenot et al. (2019)
- Crab photon-index map must be consistent with Mori et al. (2004) on the same dataset while resolving finer spatial detail than re-binned per-pixel 1D fits
- Non-negativity invariant: amplitude maps contain no negative values after the proximal projection step

## Limitations

- SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component because patches were too large for the spatial regularization to compensate
- SUSHI's final output is fitted unmixed spectra but does not directly provide physical parameters; conversion via the IAE's parameter encoding is needed
- Toy-model generation depends on a simulation (Orlando et al. 2016) provided by a third party rather than a publicly hosted dataset link
- Trained IAE networks/weights are not stated as shared (only code availability is given), and no random seeds or training hyperparameter randomness handling is described for IAE training

## Evidence

- [intro] Non-stationary unmixing aims to account not only for the varying amplitude of each physical component but also for the variation of their spectral shapes across the image: "Non-stationary unmixing aims to account not only for the varying amplitude of each physical component but also for the variation of their spectral shapes across the image"
- [methods] Proximal alternating linearized minimization (PALM) optimization: [evidence span withheld — anchor status: not_found]
- [methods] Starlet transform: "we used the isotropic undecimated wavelet transform introduced in Starck & Murtagh (1994), dubbed "starlet transform""
- [results] RMSE comparison: "the average root mean squared error for each spectra was found to be 0.19 for the thermal and 0.16 for the synchrotron, while for the classic method it was 0.22 for the thermal and 0.19 for the"
- [readme] README environment setup: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
