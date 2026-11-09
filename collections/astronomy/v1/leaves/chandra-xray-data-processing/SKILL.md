---
name: chandra-xray-data-processing
description: Use when Chandra ACIS observation(s) must be turned into a spatial–spectral
  cube (e.g. 3.0–8.48 keV band covering continuum and Fe-K) for component separation
  or parameter mapping.
when_to_use_negative:
- The data are not IFU/hyperspectral cubes (no spatial–spectral cube can be constructed,
  e.g. single 1D spectra)
- No fixed number of physical components with a spectral model can be assumed for
  the target
- A per-pixel 1D spectral fit is already sufficient — SUSHI was compared to the classic
  approach and only outperformed it in parameter reconstruction accuracy
tools:
- name: SUSHI
  role: Semi-blind unmixing algorithm applied to the prepared Chandra cube with variable
    number of components
  repo: https://github.com/JMLascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Learned spectral decoder per physical component, trained on spectra convolved
    with the Chandra response
  repo: https://github.com/jbobin/IAE
- name: Chandra X-ray telescope
  role: Instrument providing ACIS-S observations and ARF/RMF response used to build
    and forward-model the cube
- name: Python 3.10
  role: Implementation language for SUSHI and IAE training
- name: JAX
  role: Auto-differentiation backend for the IAE decoder used in the PALM gradient
    steps
- name: Sherpa
  role: Baseline reference for the look-up-table XSpec physical model against which
    the IAE surrogate speed was compared
- name: XSpec
  role: Table-based physical model (APEC) used for comparison with the learned IAE
    decoder
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:f1c948339e4cadd03064b6257ee532c1d1295be66c30d4087ec21bf22baf975c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JMLascar/SUSHI
  tools:
  - Chandra X-ray telescope
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - Python 3.10
  - SUSHI
  - Sherpa
  - XSpec
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# chandra-xray-data-processing

## Summary

Prepare Chandra X-ray observations (e.g. ACIS-S Cassiopeia A 2004, 980 ks; Crab ObsIDs 1994–2001) into hyperspectral data cubes with blank-sky background and ARF/RMF instrumental response, ready for spectral unmixing with SUSHI and IAE spectral models.

## When to use

Use when Chandra ACIS observation(s) must be turned into a spatial–spectral cube (e.g. 3.0–8.48 keV band covering continuum and Fe-K) for component separation or parameter mapping.

## When NOT to use

- The data are not IFU/hyperspectral cubes (no spatial–spectral cube can be constructed, e.g. single 1D spectra)
- No fixed number of physical components with a spectral model can be assumed for the target
- A per-pixel 1D spectral fit is already sufficient — SUSHI was compared to the classic approach and only outperformed it in parameter reconstruction accuracy

## Inputs

- Chandra ACIS-S event/observation data for the target (e.g. Cas A 2004, 980 ks merged; Crab Nebula ObsIDs 1994, 1995, 1996, 1997, 1999, 2000, 2001)
- Chandra instrumental response files (ARF and RMF)
- Chandra blank-sky observations (ACIS calibration team) for background spectrum
- APEC spectral model look-up tables (AtomDB)
- Pre-trained IAE spectral models (one per physical component)

## Outputs

- 3D hyperspectral data cube (spatial x energy, e.g. 3.0–8.48 keV)
- Background spectrum for the cube
- Per-component fitted cubes X̂c = Ac⊙Mc(θc) after SUSHI unmixing
- Thermal and synchrotron amplitude and parameter maps

## How to apply

Merge the target's Chandra observations (e.g. the Cassiopeia A SNR Chandra ACIS-S 2004 observations with 980 ks total exposure) and construct a 3D data cube over a selected energy band — the Cas A analysis used 3.0–8.48 keV covering the continuum and Fe-K line complex. Compute a background spectrum from the blank-sky observations provided by the Chandra ACIS calibration team and account for the Chandra instrumental response using the ARF and RMF files; physical spectra (e.g. collisionally ionized hot gas, APEC model based on AtomDB look-up tables) must be convolved with this response. Process in Python 3.10 within the SUSHI_env conda environment (SUSHI_env.yml), using JAX for the IAE spectral models and Sherpa/XSpec for the table-based physical models when needed. For validation-style comparisons, note that Cas A simulations used noise levels similar to the real 2004, 980 ks observations in the same energy band. The resulting cube is then fed to SUSHI with one pre-trained IAE decoder per physical component (thermal, synchrotron).

## Related tools

- **SUSHI** (Semi-blind unmixing algorithm applied to the prepared Chandra cube with variable number of components) — https://github.com/JMLascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Learned spectral decoder per physical component, trained on spectra convolved with the Chandra response) — https://github.com/jbobin/IAE
- **Chandra X-ray telescope** (Instrument providing ACIS-S observations and ARF/RMF response used to build and forward-model the cube)
- **Python 3.10** (Implementation language for SUSHI and IAE training)
- **JAX** (Auto-differentiation backend for the IAE decoder used in the PALM gradient steps)
- **Sherpa** (Baseline reference for the look-up-table XSpec physical model against which the IAE surrogate speed was compared)
- **XSpec** (Table-based physical model (APEC) used for comparison with the learned IAE decoder)

## Examples

```
conda env create -f SUSHI_env.yml && conda activate SUSHI_env
```

## Evaluation signals

- The merged cube covers the intended band (Cas A: 3.0–8.48 keV, continuum plus Fe-K line complex) with the stated exposure (980 ks for the 2004 ACIS-S data)
- The background spectrum is derived from Chandra ACIS calibration team blank-sky observations, not an ad-hoc estimate
- All model spectra are convolved with the ARF/RMF instrumental response before comparison to data
- Resulting component maps reproduce the spatial morphology of reference Cas A component maps (e.g. Picquenot et al. 2019 stationary unmixing): synchrotron amplitude similar, thermal amplitude comparable to the sum of their two thermal components
- Convergence of the SUSHI run: average relative cost fluctuation over fifty iterations < 10^-6 or ~10^4 maximum iterations

## Limitations

- SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component because patches were too large for the spatial regularization to compensate
- SUSHI's final output is fitted unmixed spectra and does not directly provide physical parameters
- No software version specified for JAX used to code and train the IAE models; no random seeds or training hyperparameter randomness handling described
- The toy model depends on a simulation provided by a third party (S. Orlando) rather than a publicly hosted dataset link

## Evidence

- [results] Chandra ACIS-S 2004 merged cube: "observations of the Cassiopeia A SNR as taken by the Chandra ACIS-S instrument in 2004, with a total exposure time of 980 ks"
- [results] Blank-sky background: [evidence span withheld — anchor status: not_found]
- [methods] Instrumental response convolution: "an emission spectrum from collisionally ionized hot gas (APEC model1) convolved with the Chandra X-ray telescope instrumental response"
- [results] Crab cube construction: "The data cube was constructed from the Chandra X-ray observation from the years 2000 and 2001 (ObsID: 1994, 1995, 1996, 1997, 1999, 2000, 2001)"
- [readme] Environment setup: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
