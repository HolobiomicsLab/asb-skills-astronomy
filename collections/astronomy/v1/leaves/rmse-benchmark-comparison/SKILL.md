---
name: rmse-benchmark-comparison
description: Use when a simulated hyperspectral cube with known ground-truth component
  maps is available and you must quantify whether a new unmixing/fitting method (e.g.
  SUSHI) reconstructs component spectra more accurately than a baseline (e.g. pixel-by-pixel
  1D fit).
when_to_use_negative:
- The analysis targets real observational data (e.g. the 2004 Cas A or Crab Nebula
  Chandra cubes) where no ground-truth component maps exist to compute RMSE against
- The goal is to derive physical parameters themselves — note that SUSHI's output
  is fitted unmixed spectra and does not directly provide physical parameters, so
  RMSE on parameters requires a separate fitting/decoding step
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: SUSHI
  role: 'New method under benchmark: non-stationary unmixing with starlet-wavelet
    sparsity regularization, run to produce reconstructed thermal and synchrotron
    cubes'
  repo: https://github.com/jmlascar/SUSHI
- name: Sherpa
  role: 'Baseline method: classic per-pixel 1D spectral fit (XSpec model in Sherpa)
    with no spatial regularization on the same cube'
- name: IAE (Interpolatory Auto-Encoder)
  role: Pretrained spectral surrogate model per component used by SUSHI to generate
    reconstructed spectra for the RMSE comparison
  repo: https://github.com/jbobin/IAE
- name: Python 3.10 / JAX
  role: Implementation environment for SUSHI and the IAE models, used to compute RMSE
    metrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:45af8ce5d45038ca7e676cfc70150fbedf03d3341cf0de2de5944cc706ba7427
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - Python 3.10 / JAX
  - SUSHI
  - Sherpa
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rmse-benchmark-comparison

## Summary

Benchmark a new hyperspectral unmixing method against a classic baseline by computing the average root mean squared error (RMSE) per spectrum of each reconstructed component cube against known ground truth, as done when comparing SUSHI to per-pixel 1D spectral fits on the Cassiopeia A-like toy model.

## When to use

Use when a simulated hyperspectral cube with known ground-truth component maps is available and you must quantify whether a new unmixing/fitting method (e.g. SUSHI) reconstructs component spectra more accurately than a baseline (e.g. pixel-by-pixel 1D fit).

## When NOT to use

- The analysis targets real observational data (e.g. the 2004 Cas A or Crab Nebula Chandra cubes) where no ground-truth component maps exist to compute RMSE against
- The goal is to derive physical parameters themselves — note that SUSHI's output is fitted unmixed spectra and does not directly provide physical parameters, so RMSE on parameters requires a separate fitting/decoding step

## Inputs

- Ground-truth hyperspectral toy-model cube (94×94 pixels, 375 energy channels between 3 keV and 8.48 keV, Poisson-noised) with known component amplitude and parameter maps
- Reconstructed component cubes from the new method (SUSHI output using pretrained IAE models for APEC thermal and synchrotron components)
- Reconstructed component cubes from the baseline classic pixel-by-pixel 1D spectral fit (e.g. Sherpa/XSpec)

## Outputs

- Average per-spectrum RMSE for each component and each method (e.g. 0.19 thermal / 0.16 synchrotron for SUSHI vs 0.22 / 0.19 for classic)
- Residual standard deviations per fitted physical parameter (temperature, redshift, photon index, amplitude)
- Comparison table saved as sushi_vs_classic_rmse.csv
- Reconstructed component cubes for both methods

## How to apply

Run both the new method (SUSHI, Algorithm 1: PALM optimization with starlet-wavelet l1 sparsity on the spectral-parameter maps and non-negativity on amplitudes, stopping when the relative cost fluctuation over fifty iterations falls below ~1e-6) and the baseline (classic 1D spectral fit, e.g. XSpec model in Sherpa, with no spatial regularization) on the same ground-truth-bearing cube — for the SUSHI paper, a 94×94 pixel, 375-channel (3–8.48 keV) toy model with Poisson noise at a level similar to the 2004 980 ks Chandra Cas A observation. For each method and each physical component (APEC thermal, synchrotron power-law), compute the RMSE between reconstructed and true spectra for every pixel, then average per spectrum. Compare the resulting averages component-by-component and save the reconstructed cubes plus the comparison table (e.g. sushi_vs_classic_rmse.csv). SUSHI's reported averages were 0.19 (thermal) and 0.16 (synchrotron) versus 0.22 and 0.19 for the classic method; temperature and photon-index residual standard deviations improved by an order of magnitude, while thermal amplitude was equally accurate — so evaluate per-parameter, not only globally.

## Related tools

- **SUSHI** (New method under benchmark: non-stationary unmixing with starlet-wavelet sparsity regularization, run to produce reconstructed thermal and synchrotron cubes) — https://github.com/jmlascar/SUSHI
- **Sherpa** (Baseline method: classic per-pixel 1D spectral fit (XSpec model in Sherpa) with no spatial regularization on the same cube)
- **IAE (Interpolatory Auto-Encoder)** (Pretrained spectral surrogate model per component used by SUSHI to generate reconstructed spectra for the RMSE comparison) — https://github.com/jbobin/IAE
- **Python 3.10 / JAX** (Implementation environment for SUSHI and the IAE models, used to compute RMSE metrics)

## Examples

```
conda env create -f SUSHI_env.yml && conda activate SUSHI_env && jupyter notebook SUSHI\ Test\ Notebook.ipynb
```

## Evaluation signals

- SUSHI's average per-spectrum RMSE is lower than the classic method's for each component (paper values: 0.19 thermal / 0.16 synchrotron for SUSHI vs 0.22 / 0.19 for classic)
- Residual standard deviations of temperature and photon index are reduced by roughly an order of magnitude relative to the classic fit
- Thermal amplitude RMSE is approximately equal between the two methods — a large discrepancy here would signal a pipeline error
- Comparison table is written with one RMSE row per component per method (sushi_vs_classic_rmse.csv) and reconstructed cubes are saved
- Both methods are run on the identical Poisson-noised cube so the comparison is not confounded by differing noise realizations

## Limitations

- RMSE evaluation requires ground truth, so it is only possible on simulated data (the toy model used the Orlando et al. 2016 simulation provided by S. Orlando, not a public dataset link)
- SUSHI fails to reconstruct the synchrotron component where it is weak relative to the thermal component because patches were large enough that spatial regularization could not compensate — such regions will inflate RMSE
- SUSHI's output is fitted unmixed spectra and does not directly provide physical parameters, so parameter-level residuals require additional processing

## Evidence

- [results] RMSE values reported in paper: "the average root mean squared error for each spectra was found to be 0.19 for the thermal and 0.16 for the synchrotron, while for the classic method it was 0.22 for the thermal and 0.19 for the"
- [methods] toy model noise regime: [evidence span withheld — anchor status: not_found]
- [abstract] baseline comparison claim: "We compared this result to the one obtained by a classic 1D fit on each individual pixel. We find that SUSHI obtains more accurate results, particularly when it comes to reconstructing physical"
- [results] output limitation: "SUSHI’s final output is a fitted unmixed spectra, but it does not provide physical parameters"
- [readme] environment setup from README: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
