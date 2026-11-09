---
name: sherpa-spectral-model-timing
description: Use when adopting or training a learned spectral surrogate (e.g. an IAE
  decoder) and you need to justify that a surrogate evaluation is faster than calling
  the physical look-up-table model (XSpec in Sherpa) per spectral evaluation.
when_to_use_negative:
- You need absolute accuracy or physics fidelity validation of the surrogate rather
  than a timing comparison — this skill measures speed only, not spectral reconstruction
  error
- Your physical spectral model is analytical or otherwise cheap to evaluate, so the
  look-up-table overhead the surrogate avoids does not exist
- The IAE has not been trained (or packaged weights are unavailable for your component)
  — benchmarking an untrained decoder is meaningless
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SUSHI
  role: Repository providing the SUSHI_env.yml conda environment, the readily trained
    IAE spectral models, and the timing-benchmark context in which decoder vs. physical-model
    call time is compared
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: Learned spectral surrogate whose decoder D is timed on latent parameter vectors
    and compared against the physical model
  repo: https://github.com/jbobin/IAE
- name: Sherpa
  role: Fitting environment in which the physical look-up-table model (XSpec model)
    is called for the timing comparison
- name: XSpec
  role: Supplies the look-up-table-based physical spectral model (APEC) evaluated
    through Sherpa as the timing baseline
- name: JAX
  role: Package in which the IAE models are coded and executed during timing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:cfc5627ddfb18140e243feb21612cf9996ca931cc3eb59153318a733d6896d53
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# sherpa-spectral-model-timing

## Summary

Benchmark the per-call computational cost of a trained IAE surrogate spectral decoder against the physical look-up-table XSpec model (e.g. APEC convolved with the Chandra instrumental response) evaluated in Sherpa, to quantify the surrogate speed advantage used to justify SUSHI's learned spectral model.

## When to use

Use when adopting or training a learned spectral surrogate (e.g. an IAE decoder) and you need to justify that a surrogate evaluation is faster than calling the physical look-up-table model (XSpec in Sherpa) per spectral evaluation.

## When NOT to use

- You need absolute accuracy or physics fidelity validation of the surrogate rather than a timing comparison — this skill measures speed only, not spectral reconstruction error
- Your physical spectral model is analytical or otherwise cheap to evaluate, so the look-up-table overhead the surrogate avoids does not exist
- The IAE has not been trained (or packaged weights are unavailable for your component) — benchmarking an untrained decoder is meaningless

## Inputs

- Trained IAE decoder D (packaged in the SUSHI repository 'IAE models' directory) for a physical component (e.g. thermal APEC model over kT in [0.8, 6] keV, redshift in [-0.033, +0.033])
- Latent parameter vectors (temperature, redshift) sampled over the training domain for timing the decoder
- XSpec APEC model as configured in Sherpa, convolved with Chandra instrumental response (ARF and RMF files)
- SUSHI_env.yml conda environment specification (Python 3.10)

## Outputs

- Measured decoder call timings (e.g. per-call seconds over repeated calls)
- Measured physical-model (XSpec APEC in Sherpa) call timings
- Speed ratio (physical-model time / decoder time); >60x in the paper's toy-model setting
- iae_decoder_timing_benchmark.csv with timings and ratio

## How to apply

Create and activate the conda environment from SUSHI_env.yml (Python 3.10) from the JMLascar/SUSHI repository and load the readily trained IAE spectral model packaged with it (e.g. the thermal model trained on APEC spectra over kT in [0.8, 6] keV and redshift in [-0.033, +0.033]). Time repeated calls of the IAE decoder D on representative latent parameter vectors. Then time an equivalent single call of the look-up-table-based physical model — the APEC model (AtomDB) convolved with the Chandra instrumental response (ARF and RMF files) — via the XSpec model in Sherpa. Compute the ratio of physical-model call time to decoder call time; in the paper's toy-model setting this ratio exceeded 60x. Write the measured timings and ratio to a results file (iae_decoder_timing_benchmark.csv) so the surrogate's computational advantage is documented for the downstream unmixing workload, where SUSHI evaluates the spectral model many times during PALM optimization.

## Related tools

- **SUSHI** (Repository providing the SUSHI_env.yml conda environment, the readily trained IAE spectral models, and the timing-benchmark context in which decoder vs. physical-model call time is compared) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (Learned spectral surrogate whose decoder D is timed on latent parameter vectors and compared against the physical model) — https://github.com/jbobin/IAE
- **Sherpa** (Fitting environment in which the physical look-up-table model (XSpec model) is called for the timing comparison)
- **XSpec** (Supplies the look-up-table-based physical spectral model (APEC) evaluated through Sherpa as the timing baseline)
- **JAX** (Package in which the IAE models are coded and executed during timing)

## Examples

```
conda env create -f SUSHI_env.yml && conda activate SUSHI_env  # then time decoder D vs. XSpec APEC (Sherpa) calls; ratio > 60x -> write iae_decoder_timing_benchmark.csv
```

## Evaluation signals

- The computed speed ratio (physical-model call time / decoder call time) should exceed ~60x, matching the published toy-model benchmark for the APEC/Chandra configuration
- Decoder timings must be taken over repeated calls on representative latent vectors within the training domain (kT in [0.8, 6] keV, redshift in [-0.033, +0.033]) to avoid extrapolation artifacts
- The physical-model timing must correspond to an equivalent call — APEC convolved with the Chandra instrumental response (ARF and RMF) — not a bare unconvolved model evaluation
- Results are reproducible from the packaged SUSHI environment (conda env create -f SUSHI_env.yml) and recorded in iae_decoder_timing_benchmark.csv

## Limitations

- The >60x speed-up is demonstrated only in the toy-model setting (thermal APEC component trained on Chandra-convolved spectra); absolute timings and ratios may differ for other instruments, energy grids, or components
- Timing benchmarks do not establish that the IAE surrogate reproduces the physical model's spectra — accuracy must be validated separately
- Trained IAE networks/weights are not stated as shared in the paper (only code availability), so reproducing the exact benchmark may depend on weights packaged in the repository
- No software version is specified for JAX or the IAE training beyond Python 3.10, so timing portability across JAX versions is not guaranteed

## Evidence

- [methods] The trained IAE decoder is over 60 times faster to call than the look-up-table-based physical model: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [methods] The physical model benchmarked is APEC convolved with the Chandra instrumental response.: "an emission spectrum from collisionally ionized hot gas (APEC model1) convolved with the Chandra X-ray telescope instrumental response"
- [readme] The SUSHI repository packages readily trained IAE models and the conda environment for testing.: "IAE models: readily trained IAE spectral models for testing SUSHI."
- [readme] SUSHI environment setup is via SUSHI_env.yml with Python 3.10.: "SUSHI was coded in python 3.10. To test SUSHI, you will need the packages listed in SUSHI_env.yml."
