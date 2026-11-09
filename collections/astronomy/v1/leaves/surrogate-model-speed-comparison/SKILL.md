---
name: surrogate-model-speed-comparison
description: Use when a learned surrogate model (e.g. an Interpolatory Auto-Encoder
  decoder) is claimed to be faster than the physical spectral model it was trained
  to mimic, and the speed-up ratio must be measured and reported.
when_to_use_negative:
- The learned model has not yet been trained — benchmarking speed is meaningless until
  the IAE decoder D exists and converges (check the training cost converged per the
  'Training an IAE.ipynb' tutorial)
- The comparison target is not a look-up-table-based physical model callable through
  XSpec/Sherpa (the ratio only makes sense against the specific surrogate target)
- The spectral evaluation is a one-off call where setup cost dominates — the >60x
  advantage matters for repeated calls in unmixing loops, not single invocations
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SUSHI
  role: Repository providing the conda environment (SUSHI_env.yml) and the readily
    trained IAE spectral models used in the benchmark
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: The trained surrogate decoder D whose per-call timing is measured against
    the physical model
  repo: https://github.com/jbobin/IAE
- name: Sherpa
  role: Fitting environment in which the look-up-table-based XSpec APEC physical model
    is called for the comparison timing
- name: XSpec
  role: Provides the APEC thermal spectral model whose look-up-table evaluation speed
    is benchmarked
- name: JAX
  role: Package in which the IAE models are coded, executing the decoder D calls
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:e5339071ec16026e9056b5552bc07d8deb47cf8121263c9b24dc8f03727fe77e
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# surrogate-model-speed-comparison

## Summary

Benchmarks the per-call wall-clock cost of a trained IAE decoder D against its look-up-table-based physical surrogate target (XSpec APEC model in Sherpa, convolved with the Chandra ARF/RMF instrumental response), quantifying the surrogate's computational advantage (>60x).

## When to use

Use when a learned surrogate model (e.g. an Interpolatory Auto-Encoder decoder) is claimed to be faster than the physical spectral model it was trained to mimic, and the speed-up ratio must be measured and reported.

## When NOT to use

- The learned model has not yet been trained — benchmarking speed is meaningless until the IAE decoder D exists and converges (check the training cost converged per the 'Training an IAE.ipynb' tutorial)
- The comparison target is not a look-up-table-based physical model callable through XSpec/Sherpa (the ratio only makes sense against the specific surrogate target)
- The spectral evaluation is a one-off call where setup cost dominates — the >60x advantage matters for repeated calls in unmixing loops, not single invocations

## Inputs

- Trained IAE spectral model (decoder D) packaged with the SUSHI repository under 'IAE models'
- Representative latent parameter vectors (e.g. kT in [0.8, 6] keV, redshift in [-0.033, +0.033] for the thermal APEC component)
- XSpec APEC model callable via Sherpa
- Chandra instrumental response files (ARF and RMF) for convolving the physical model
- conda environment file SUSHI_env.yml (Python 3.10)

## Outputs

- iae_decoder_timing_benchmark.csv with measured decoder call times, physical-model call times, and the speed ratio
- Reported speed-up ratio (expected to exceed 60x per call)

## How to apply

Create the conda environment from SUSHI_env.yml (Python 3.10) and load the readily trained IAE spectral model packaged with the JMLascar/SUSHI repository (e.g. the thermal model trained on APEC spectra over kT in [0.8, 6] keV and redshift [-0.033, +0.033]). Time repeated calls of the IAE decoder D on representative latent parameter vectors. Separately time an equivalent single call of the look-up-table-based physical model — the XSpec APEC model in Sherpa convolved with the Chandra instrumental response (ARF and RMF files). Compute the speed ratio as physical-model call time divided by decoder call time; the paper reports the decoder is over 60 times faster. Write the measured timings and the ratio to a CSV file (e.g. iae_decoder_timing_benchmark.csv). Rationale: per-call cost is the relevant metric for unmixing loops like SUSHI's PALM optimization, where D is invoked thousands of times, so even a per-call benchmark establishes the surrogate's utility.

## Related tools

- **SUSHI** (Repository providing the conda environment (SUSHI_env.yml) and the readily trained IAE spectral models used in the benchmark) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (The trained surrogate decoder D whose per-call timing is measured against the physical model) — https://github.com/jbobin/IAE
- **Sherpa** (Fitting environment in which the look-up-table-based XSpec APEC physical model is called for the comparison timing)
- **XSpec** (Provides the APEC thermal spectral model whose look-up-table evaluation speed is benchmarked)
- **JAX** (Package in which the IAE models are coded, executing the decoder D calls)

## Examples

```
conda env create -f SUSHI_env.yml && conda activate SUSHI_env; python -c "import time, numpy as np; from IAE_JAX_v2_devl_2023 import *; t0=time.time(); [D(theta) for theta in np.random.rand(100,2)]; t_dae=(time.time()-t0)/100; t0=time.time(); sherpa.astro.xspec.XSapec().calc([1.0,3.0,0.0], x); t_lut=time.time()-t0; print(t_lut/t_dae)"
```

## Evaluation signals

- The computed speed ratio (physical-model call time / decoder call time) exceeds 60, matching the paper's methods-section claim
- The benchmark CSV contains both timing columns and the derived ratio, not just the ratio
- Decoder timing is measured over repeated calls on representative latent vectors within the trained parameter range (kT in [0.8, 6] keV, redshift in [-0.033, +0.033]), not extrapolated outside it
- The physical-model timing is for a single equivalent call of the APEC model convolved with the Chandra instrumental response via Sherpa, matching the same spectral channels as the decoder
- Timings are run in the SUSHI_env conda environment (Python 3.10) so that JAX-based decoder calls execute in the same software stack as the paper

## Limitations

- The >60x ratio is demonstrated in the toy-model setting; speed-up on other hardware, JAX versions, or other physical models may differ
- No software version is specified for JAX, and the article does not state random seeds or hardware, so exact timings are not reproducible — treat the ratio as an order-of-magnitude claim
- Only per-call inference speed is benchmarked; IAE training time and accuracy-vs-physical-model error are separate concerns not covered by this benchmark

## Evidence

- [methods] decoder speed advantage over look-up-table model: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [methods] physical model definition (APEC + Chandra response): "an emission spectrum from collisionally ionized hot gas (APEC model1) convolved with the Chandra X-ray telescope instrumental response"
- [readme] environment setup instructions: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
- [readme] readily trained IAE models provided: "*IAE models:* readily trained IAE spectral models for testing SUSHI."
- [discussion] IAE models coded in JAX: "The IAE models trained in this work were coded in Python using the package JAX."
