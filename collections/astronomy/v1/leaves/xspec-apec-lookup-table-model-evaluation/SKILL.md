---
name: xspec-apec-lookup-table-model-evaluation
description: Use when a learned spectral surrogate (e.g. an IAE decoder) needs a baseline
  computational-cost comparison against the physical look-up-table model it was trained
  to replace, in an X-ray spectroscopy workflow.
when_to_use_negative:
- When the goal is accuracy comparison of spectra rather than computational-cost benchmarking
  — this skill measures call latency, not RMSE of reconstructed spectra
- When the physical model is analytical rather than look-up-table based, since the
  >60x speed-up claim is specific to XSpec APEC look-up-table evaluation via Sherpa
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SUSHI
  role: Repository providing the environment file (SUSHI_env.yml, Python 3.10) and
    readily trained IAE models to benchmark
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Trained surrogate spectral decoder D whose call time is measured on latent
    parameter vectors
  repo: https://github.com/jbobin/IAE
- name: Sherpa
  role: Framework through which the XSpec physical model call is timed
- name: XSpec
  role: Provides the look-up-table-based physical spectral model (APEC) used as the
    timing baseline
- name: JAX
  role: Backend in which the IAE decoder D is coded and executed during timing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:33a51d46d3ae3cb42d564b10dababcfde7b0ae04f738ed79fa177e9b100dcb77
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# xspec-apec-lookup-table-model-evaluation

## Summary

Benchmarks a single call to the physical look-up-table spectral model (XSpec APEC convolved with Chandra instrumental response, evaluated via Sherpa) against a trained IAE decoder call, to quantify the surrogate's speed-up (reported >60x).

## When to use

Use when a learned spectral surrogate (e.g. an IAE decoder) needs a baseline computational-cost comparison against the physical look-up-table model it was trained to replace, in an X-ray spectroscopy workflow.

## When NOT to use

- When the goal is accuracy comparison of spectra rather than computational-cost benchmarking — this skill measures call latency, not RMSE of reconstructed spectra
- When the physical model is analytical rather than look-up-table based, since the >60x speed-up claim is specific to XSpec APEC look-up-table evaluation via Sherpa

## Inputs

- Trained IAE spectral model from the SUSHI repository's 'IAE models' directory
- Representative latent parameter vectors for the decoder (e.g. kT in [0.8, 6] keV, redshift in [-0.033, +0.033])
- Chandra instrumental response files (ARF and RMF) for folding the physical model
- Sherpa/XSpec APEC model configuration (AtomDB lookup tables)

## Outputs

- Measured decoder call timings (per-call latency)
- Measured physical look-up-table model call timings (per-call latency)
- Speed ratio between physical-model call and decoder call
- CSV file of timings and ratio (iae_decoder_timing_benchmark.csv)

## How to apply

Set up the SUSHI conda environment (SUSHI_env.yml, Python 3.10) and load the readily trained IAE spectral model shipped with the repository (thermal component trained on APEC spectra over kT in [0.8, 6] keV and redshift [-0.033, +0.033]). Time repeated calls of the IAE decoder D on representative latent parameter vectors. Then time an equivalent single call of the look-up-table-based physical model — the APEC model (AtomDB) convolved with the Chandra instrumental response (ARF and RMF files) — evaluated through the XSpec model in Sherpa. Compute the ratio (physical-model time / decoder time); the SUSHI paper reports this exceeds 60x, which is the threshold indicating the surrogate's computational advantage. Write the measured timings and ratio to a CSV (e.g. iae_decoder_timing_benchmark.csv).

## Related tools

- **SUSHI** (Repository providing the environment file (SUSHI_env.yml, Python 3.10) and readily trained IAE models to benchmark) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Trained surrogate spectral decoder D whose call time is measured on latent parameter vectors) — https://github.com/jbobin/IAE
- **Sherpa** (Framework through which the XSpec physical model call is timed)
- **XSpec** (Provides the look-up-table-based physical spectral model (APEC) used as the timing baseline)
- **JAX** (Backend in which the IAE decoder D is coded and executed during timing)

## Examples

```
python -c "import time; from SUSHI import IAE; import sherpa.astro.ui as ui; t0=time.time(); [iae.dec(z) for z in latent_vectors]; t_iae=(time.time()-t0)/len(latent_vectors); ui.set_model('xs*apec'); t0=time.time(); ui.calc_model(); t_lut=time.time()-t0; print('speed-up:', t_lut/t_iae)"
```

## Evaluation signals

- Computed speed ratio (physical-model time / decoder time) exceeds 60, matching the paper's reported result
- Timing CSV (iae_decoder_timing_benchmark.csv) contains per-call timings for both models plus the ratio
- The physical-model call folds APEC through the Chandra ARF and RMF, matching the surrogate's training condition
- Decoder timings are taken over multiple latent vectors spanning the trained parameter range (kT [0.8, 6] keV, redshift [-0.033, +0.033]) rather than a single point

## Limitations

- The >60x ratio was measured in the toy-model setting with a specific thermal component (APEC, kT [0.8, 6] keV); it may not transfer to other components, energy bands, or instrument responses
- Timing results are hardware- and environment-dependent (Python 3.10, JAX backend, conda env SUSHI_env.yml)
- The IAE models trained in the work were coded in Python using JAX, but no JAX or Python version is specified for the IAE training itself, so exact reproducibility of timings is uncertain

## Evidence

- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [methods] APEC convolved with Chandra response: "an emission spectrum from collisionally ionized hot gas (APEC model1) convolved with the Chandra X-ray telescope instrumental response"
- [readme] SUSHI_env.yml setup: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
- [readme] readily trained IAE models available: "*IAE models:* readily trained IAE spectral models for testing SUSHI."
