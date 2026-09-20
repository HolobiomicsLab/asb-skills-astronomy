---
name: synchrotron-power-law-spectral-fitting
description: Use when the data cube contains a single synchrotron (non-thermal power-law)
  component with no unmixing needed, and reliable spectral parameters are required
  at spatial scales finer than what re-binned per-pixel 1D spectral fits achieve.
when_to_use_negative:
- The data contain multiple mixed physical components (e.g. thermal plus non-thermal
  emission) that require unmixing rather than single-component power-law fitting
- The spatial scales of interest are coarse enough that re-binned per-pixel 1D spectral
  fits already provide reliable synchrotron spectral parameters
- No trained IAE spectral model exists for the assumed power-law component and no
  training spectra (at least 1e3) from the physical model are available
tools:
- name: SUSHI
  role: Performs the spatially-regularized semi-blind unmixing / PALM optimization
    that fits the single synchrotron component across the cube
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Trained spectral model encoding the synchrotron power-law; its interpolatory
    parameter encoding converts latent maps to photon index
  repo: https://github.com/jbobin/IAE
- name: Chandra X-ray telescope
  role: Source of the ACIS X-ray observations (ObsIDs 1994-2001) that form the data
    cube
- name: Python 3.10
  role: Implementation language; SUSHI environment is created via SUSHI_env.yml with
    conda
  repo: https://github.com/jmlascar/SUSHI
- name: JAX
  role: Backend for the IAE spectral model code (IAE_JAX_v2_devl_2023.py)
  repo: https://github.com/jmlascar/SUSHI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:8df9173bcb185361ea07a29106957e0abc0ccf26ada6373c17bf945410859930
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - Chandra X-ray telescope
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - Python 3.10
  - SUSHI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# synchrotron-power-law-spectral-fitting

## Summary

Maps synchrotron photon-index at fine spatial scales in X-ray hyperspectral (Chandra) data using SUSHI's spatially-regularized semi-blind unmixing with a trained power-law IAE spectral model, instead of per-pixel 1D spectral fits.

## When to use

Use when the data cube contains a single synchrotron (non-thermal power-law) component with no unmixing needed, and reliable spectral parameters are required at spatial scales finer than what re-binned per-pixel 1D spectral fits achieve.

## When NOT to use

- The data contain multiple mixed physical components (e.g. thermal plus non-thermal emission) that require unmixing rather than single-component power-law fitting
- The spatial scales of interest are coarse enough that re-binned per-pixel 1D spectral fits already provide reliable synchrotron spectral parameters
- No trained IAE spectral model exists for the assumed power-law component and no training spectra (at least 1e3) from the physical model are available

## Inputs

- Chandra X-ray data cube (e.g. Crab Nebula observations from 2000-2001, ObsID: 1994, 1995, 1996, 1997, 1999, 2000, 2001)
- Background spectrum derived from the Chandra ACIS calibration team's blank-sky observations
- Trained power-law IAE spectral model for a single synchrotron component (n_C = 1)

## Outputs

- Synchrotron photon-index map (saved as crab_photon_index_map.fits)
- Fitted synchrotron cube Ac*Mc(theta_c) as an auxiliary product
- Total fit cube as an auxiliary product

## How to apply

Build a data cube from the Chandra observations and an associated background spectrum (e.g. the Chandra ACIS calibration team's blank-sky observations), then load a trained power-law Interpolatory Auto-Encoder (IAE) spectral model for the single synchrotron component (n_C = 1). Initialize the amplitude map as per-pixel total counts divided by the number of components and the latent parameters as uniform weights (theta_i = 1/nA). Run the SUSHI PALM loop: alternate a gradient-descent step on the latent theta parameters (step size alpha = 0.1/max(A0)), a starlet-wavelet l1 sparsity proximal step on the theta maps with an MAD-based soft threshold (threshold factor k = 1), a Newton-style gradient step on the amplitude using the explicit Hessian diagonal, and a non-negativity projection, iterating until the cost-function fluctuation criterion (epsilon = 1e-6) or ~1e4 maximum iterations. Convert the fitted latent parameter map to the synchrotron photon-index map via the IAE's interpolatory parameter encoding.

## Related tools

- **SUSHI** (Performs the spatially-regularized semi-blind unmixing / PALM optimization that fits the single synchrotron component across the cube) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Trained spectral model encoding the synchrotron power-law; its interpolatory parameter encoding converts latent maps to photon index) — https://github.com/jbobin/IAE
- **Chandra X-ray telescope** (Source of the ACIS X-ray observations (ObsIDs 1994-2001) that form the data cube)
- **Python 3.10** (Implementation language; SUSHI environment is created via SUSHI_env.yml with conda) — https://github.com/jmlascar/SUSHI
- **JAX** (Backend for the IAE spectral model code (IAE_JAX_v2_devl_2023.py)) — https://github.com/jmlascar/SUSHI

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env; jupyter notebook "SUSHI Test Notebook.ipynb"
```

## Evaluation signals

- Reconstructed photon-index map is consistent with past findings on the same data set (Mori et al. 2004)
- Photon-index map resolves fine spatial detail beyond what re-binned per-pixel 1D spectral fits achieve, thanks to the starlet-wavelet spatial regularization
- PALM loop cost-function fluctuation falls below the convergence criterion epsilon = 1e-6 (or ~1e4 maximum iterations reached)
- Latent weights remain valid barycentric weights summing to 1 across the theta map, and amplitudes stay non-negative after each projection
- Average RMSE per spectrum is lower than the classic per-pixel 1D fit baseline (paper reports 0.16 vs 0.19 for synchrotron in the toy model)

## Limitations

- SUSHI's final output is fitted unmixed spectra; physical parameters like photon index must be extracted through the IAE's parameter encoding, not directly provided
- Where the synchrotron component is weak relative to other emission, spatial regularization with large patches may fail to compensate, as seen for the toy model
- Trained IAE weights are not stated as shared (only code availability is given); the power-law IAE must be trained per component using the provided tutorial

## Evidence

- [discussion] Finally, we applied SUSHI to a simpler real case: X-ray Chandra data of the Crab Nebula, which only contains one component, the synchrotron emission.: [evidence span withheld — anchor status: not_found]
- [results] The data cube was constructed from the Chandra X-ray observation from the years 2000 and 2001 (ObsID: 1994, 1995, 1996, 1997, 1999, 2000, 2001): "The data cube was constructed from the Chandra X-ray observation from the years 2000 and 2001 (ObsID: 1994, 1995, 1996, 1997, 1999, 2000, 2001)"
- [methods] Then, a proximal step to minimize the norm of W(θc) while remaining close to the previously found value of θc.: [evidence span withheld — anchor status: not_found]
- [abstract] The results obtained are realistic and in accordance with past findings but have a much better spatial resolution.: "The results obtained are realistic and in accordance with past findings but have a much better spatial resolution."
- [readme] In order to obtain more robust results on voxels with low signal to noise ratio, a spatial regularization is applied.: "In order to obtain more robust results on voxels with low signal to noise ratio, a spatial regularization is applied."
