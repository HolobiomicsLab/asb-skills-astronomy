---
name: chandra-instrument-response-convolution
description: Use when generating spectra from a physical model (e.g. APEC or power-law)
  as a training set for a learned spectral surrogate like an IAE, and the target data
  were observed with Chandra so the surrogate must operate in instrumental counts
  space.
when_to_use_negative:
- The target data are not in instrumental counts space (e.g. already unfolded/fluffed
  to physical flux units), so convolution would double-apply the instrument response
- The surrogate is being trained for a different instrument or observation configuration
  — the ARF/RMF files must match that specific Chandra setup (e.g. ACIS-S), not be
  borrowed from another dataset
- The analysis goal is per-pixel 1D fitting of physical flux spectra rather than training
  a learned surrogate for source separation
edam_operation: http://edamontology.org/operation_3208
tools:
- name: Chandra X-ray telescope
  role: Instrument whose ARF and RMF response files define the convolution applied
    to the model spectra
- name: XSpec
  role: Look-up-table-based physical model (used via Sherpa) whose spectra are convolved
    with the Chandra response to build the IAE training set
- name: SUSHI
  role: Downstream algorithm whose IAE spectral models are trained on the Chandra-convolved
    spectra
  repo: https://github.com/jmlascar/SUSHI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:568ccad6f0932ee54d6c8f343a4f10933ea16f45708fe95c453f1ab2012aaf7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - Chandra X-ray telescope
  - IAE training code (github.com/jbobin/IAE)
  - JAX
  - Python
  - SUSHI
  - XSpec
  edam_operation: http://edamontology.org/operation_3208
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# chandra-instrument-response-convolution

## Summary

Convolve synthetic physical-model spectra (e.g. APEC collisionally-ionized thermal plasma from AtomDB, or power-law synchrotron emission) with the Chandra instrumental response (ARF and RMF files) so that spectra fed into IAE training live in the same instrumental space as the observed Chandra data.

## When to use

Use when generating spectra from a physical model (e.g. APEC or power-law) as a training set for a learned spectral surrogate like an IAE, and the target data were observed with Chandra so the surrogate must operate in instrumental counts space.

## When NOT to use

- The target data are not in instrumental counts space (e.g. already unfolded/fluffed to physical flux units), so convolution would double-apply the instrument response
- The surrogate is being trained for a different instrument or observation configuration — the ARF/RMF files must match that specific Chandra setup (e.g. ACIS-S), not be borrowed from another dataset
- The analysis goal is per-pixel 1D fitting of physical flux spectra rather than training a learned surrogate for source separation

## Inputs

- Physical-model spectra: APEC collisionally-ionized thermal plasma (AtomDB, http://www.atomdb.org/physics.php) or power-law synchrotron emission
- Chandra ARF (ancillary response file) for the observation
- Chandra RMF (redistribution matrix file) for the observation
- Energy grid and physical-parameter ranges (Appendix Table A.2)

## Outputs

- Instrument-convolved spectra on the Chandra energy grid, as (N, n_E) numpy arrays ready for IAE training sets (Output['Psi'], Output['X_train'], Output['X_valid'])

## How to apply

Generate a large number of spectra (at least 1e3) from the physical model — APEC collisionally-ionized hot gas from AtomDB for thermal components, or power-law synchrotron emission — over the energy and physical-parameter ranges of Appendix Table A.2. Where required, convolve each spectrum with the Chandra instrumental response using the ARF and RMF files appropriate to the observation (e.g. Chandra ACIS-S 2004). Keep the convolved spectra on the same n_E-channel energy grid as the data cube so they can be stored as (N, n_E) numpy arrays for Output['Psi'] (anchor points at boundary parameters), Output['X_train'] and Output['X_valid'] (70/30 split of the shuffled remainder). The rationale is that the trained IAE decoder D then acts as a fast surrogate in instrumental space — a call to D was over 60 times faster than calling the look-up-table-based XSpec model in Sherpa — while remaining physically consistent with the Chandra data.

## Related tools

- **Chandra X-ray telescope** (Instrument whose ARF and RMF response files define the convolution applied to the model spectra)
- **XSpec** (Look-up-table-based physical model (used via Sherpa) whose spectra are convolved with the Chandra response to build the IAE training set)
- **SUSHI** (Downstream algorithm whose IAE spectral models are trained on the Chandra-convolved spectra) — https://github.com/jmlascar/SUSHI

## Evaluation signals

- Convolved spectra have exactly n_E channels matching the Chandra data cube energy grid, with shape (N, n_E)
- Anchor-point set Output['Psi'] contains spectra at boundary physical parameters (e.g. lowest/highest temperature and redshift) after convolution
- Training/validation split of the remaining spectra is 70/30 with anchors excluded from both splits
- The trained decoder D evaluated on test parameters produces spectra statistically consistent with spectra obtained by calling the XSpec/Sherpa look-up-table model convolved with the same ARF/RMF
- The IAE training cost curve converges (plateaus) over the three epochs before the model is used in SUSHI

## Limitations

- The look-up-table-based physical model (XSpec in Sherpa) is slow to call; the convolution-based surrogate is worthwhile precisely because a call to D is over 60 times faster, but generating the convolved training set itself still requires the slow physical model
- The convolution is instrument-specific: ARF and RMF files must correspond to the Chandra observation being analyzed (e.g. ACIS-S 2004 Cassiopeia A data); results are not transferable to other instruments without regenerating the training set
- The energy band and physical-parameter ranges of the training set (Appendix Table A.2) bound where the surrogate is valid

## Evidence

- [methods] These were convolved by the Chandra instrumental response (ARF and RMF files).: "These were convolved by the Chandra instrumental response (ARF and RMF files)."
- [methods] an emission spectrum from collisionally ionized hot gas (APEC model1) convolved with the Chandra X-ray telescope instrumental response: "an emission spectrum from collisionally ionized hot gas (APEC model1) convolved with the Chandra X-ray telescope instrumental response"
- [methods] a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
- [readme] Provide a large number of spectra (1e3 at least) from your physical model.: "Provide a large number of spectra (1e3 at least) from your physical model."
