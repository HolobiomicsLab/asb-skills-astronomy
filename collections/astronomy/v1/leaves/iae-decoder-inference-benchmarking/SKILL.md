---
name: iae-decoder-inference-benchmarking
description: Use when you need to justify replacing a physical (look-up-table) spectral
  model with a learned surrogate by measuring wall-clock inference cost per call for
  both models in a common environment.
when_to_use_negative:
- When the benchmark target is a physical model without a trained surrogate — there
  is no IAE decoder to time yet (train one first via 'Training an IAE.ipynb')
- When you need accuracy or reconstruction quality of the surrogate rather than inference
  speed — this skill measures only wall-clock cost, not spectral fidelity
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SUSHI
  role: Repository supplying the pre-trained IAE spectral models, SUSHI_env.yml environment
    definition, and IAE_JAX_v2_devl_2023.py decoder code being benchmarked
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Learned surrogate spectral model whose decoder D inference time is measured
    against the physical model
  repo: https://github.com/jbobin/IAE
- name: Sherpa
  role: Fitting environment through which the physical look-up-table XSpec model call
    is timed
- name: XSpec
  role: Provides the APEC look-up-table physical model used as the surrogate target
    and timing baseline
- name: JAX
  role: Package in which the IAE decoder is coded and evaluated during timing runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:7e8628c55af0cfa4732303ad926c37548b9399aadde114787716379bf84ea064
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
yes_rate: 0.5555555555555556
n_claims: 9
---

# iae-decoder-inference-benchmarking

## Summary

Benchmarks the inference-time cost of a trained IAE decoder call against the physical look-up-table spectral model (XSpec APEC in Sherpa) it surrogates, quantifying the >60x per-call speed-up that justifies using the learned surrogate in SUSHI.

## When to use

Use when you need to justify replacing a physical (look-up-table) spectral model with a learned surrogate by measuring wall-clock inference cost per call for both models in a common environment.

## When NOT to use

- When the benchmark target is a physical model without a trained surrogate — there is no IAE decoder to time yet (train one first via 'Training an IAE.ipynb')
- When you need accuracy or reconstruction quality of the surrogate rather than inference speed — this skill measures only wall-clock cost, not spectral fidelity

## Inputs

- Trained IAE spectral model from the SUSHI repository's 'IAE models' directory (decoder D and encoder networks)
- Representative latent parameter vectors (e.g. kT in [0.8, 6] keV, redshift in [-0.033, +0.033] for the thermal APEC component)
- XSpec APEC model invoked through Sherpa, convolved with Chandra ARF and RMF instrumental response files
- SUSHI conda environment (SUSHI_env.yml, Python 3.10, JAX)

## Outputs

- Measured per-call timings for the IAE decoder D and for the Sherpa/XSpec APEC call
- Computed speed ratio between physical-model call time and decoder call time (expected >60x)
- Timing summary written to iae_decoder_timing_benchmark.csv

## How to apply

Create and activate the conda environment from SUSHI_env.yml (Python 3.10), then load the readily trained IAE spectral model packaged in the SUSHI repository. Time repeated calls of the IAE decoder D on representative latent parameter vectors drawn from the training domain (e.g. the thermal IAE trained on APEC spectra over kT in [0.8, 6] keV and redshift in [-0.033, +0.033]). Time an equivalent single call of the look-up-table-based physical model — the APEC emission spectrum convolved with the Chandra instrumental response (ARF/RMF) — evaluated via the XSpec model in Sherpa. Compute the speed ratio of physical-model call time over decoder call time and record measured timings to a CSV (e.g. iae_decoder_timing_benchmark.csv). Judge success by obtaining a ratio consistent with the reported >60x advantage, ensuring both timings are measured on comparable hardware and over the same spectral channel grid.

## Related tools

- **SUSHI** (Repository supplying the pre-trained IAE spectral models, SUSHI_env.yml environment definition, and IAE_JAX_v2_devl_2023.py decoder code being benchmarked) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Learned surrogate spectral model whose decoder D inference time is measured against the physical model) — https://github.com/jbobin/IAE
- **Sherpa** (Fitting environment through which the physical look-up-table XSpec model call is timed)
- **XSpec** (Provides the APEC look-up-table physical model used as the surrogate target and timing baseline)
- **JAX** (Package in which the IAE decoder is coded and evaluated during timing runs)

## Examples

```
import time; t0=time.time(); [D(z) for z in latent_vectors]; t_d=(time.time()-t0)/len(latent_vectors)  # then time sherpa's set_model(xs.apec) evaluation and log ratio to iae_decoder_timing_benchmark.csv
```

## Evaluation signals

- Computed speed ratio (physical-model call time / decoder call time) should exceed 60, matching the reported result
- Both timings must be measured on the same hardware, same spectral channel count, and via repeated calls averaged over representative parameter vectors
- Benchmark CSV (iae_decoder_timing_benchmark.csv) contains per-model timing entries plus the derived ratio
- The timed IAE latent vectors fall within the trained parameter domain (e.g. kT 0.8–6 keV, redshift ±0.033) so the decoder call is representative of legitimate surrogate use

## Limitations

- The >60x speed-up is demonstrated in the toy-model setting (APEC thermal component); it does not by itself establish surrogate accuracy over the full parameter domain
- Timings depend on hardware and software stack (Python 3.10, JAX); the article specifies no JAX version, so exact reproducibility of absolute timings is not guaranteed
- Benchmarking reflects per-call cost only; SUSHI's iterative PALM optimization calls the model many times, so end-to-end speed-up depends on convergence iteration counts not captured here

## Evidence

- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [methods] an emission spectrum from collisionally ionized hot gas (APEC model1) convolved with the Chandra X-ray telescope instrumental response: "an emission spectrum from collisionally ionized hot gas (APEC model1) convolved with the Chandra X-ray telescope instrumental response"
- [readme] SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml.: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
- [readme] IAE models: readily trained IAE spectral models for testing SUSHI.: "*IAE models:* readily trained IAE spectral models for testing SUSHI."
