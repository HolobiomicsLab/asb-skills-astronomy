---
name: reduced-chi2-map-goodness-of-fit
description: Use when you need a goodness-of-fit metric proving that a timestream
  filtering step (e.g. destriping with 60-second baselines) removed low-frequency
  correlated noise without biasing the signal, given HEALPix residual maps for I/Q.
when_to_use_negative:
- Systematic effects such as thermally induced gain variations (e.g. sections near
  sunrise/sunset) have not been masked; the paper states such effects must be dealt
  with before map-making and destriping can be 'ineffective of even harmful' there
- The noise properties of the timestream are unknown, so no expected white-noise level
  exists against which the reduced chi2 of the residual map can be normalized
edam_operation: http://edamontology.org/operation_0335
tools:
- name: dst
  role: Produces the destriped maps whose reduced chi2 is compared against the plain-binning
    baseline
  repo: http://github.com/zonca/dst
- name: HEALPix
  role: Pixelization scheme (Nside 512) on which the residual maps and their reduced
    chi2 are computed
- name: Python
  role: Language in which the dst destriper and the chi2 computation/writing to chi2_table.csv
    are implemented
  repo: http://github.com/zonca/dst
- name: HDF5
  role: File format of the input timestream (ch6_fill_3d.h5) read via dst_io.read_data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:7c34e0652e1a15ad484b1f2b7b3e8827755d57c9746af7bbbe8ada1837c45d97
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HEALPix
  - MPI
  - Python
  - Trilinos
  - dst
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# reduced-chi2-map-goodness-of-fit

## Summary

Validates a CMB map-making (destriping) pipeline by computing the reduced chi2 of residual maps — the difference between output maps and the input smoothed signal — for I and Q, expecting destriped residuals to fall to the white-noise level (~1.0) while plain binning leaves correlated-noise excess (~1.05).

## When to use

Use when you need a goodness-of-fit metric proving that a timestream filtering step (e.g. destriping with 60-second baselines) removed low-frequency correlated noise without biasing the signal, given HEALPix residual maps for I/Q.

## When NOT to use

- Systematic effects such as thermally induced gain variations (e.g. sections near sunrise/sunset) have not been masked; the paper states such effects must be dealt with before map-making and destriping can be 'ineffective of even harmful' there
- The noise properties of the timestream are unknown, so no expected white-noise level exists against which the reduced chi2 of the residual map can be normalized

## Inputs

- HEALPix Nside 512 maps of I and Q produced by plain binning of the timestream
- HEALPix Nside 512 destriped I and Q maps from dst (python dst.py ch6_256.cfg)
- Input smoothed signal map (WMAP 7-year Q band signal smoothed by 1 degree) used to form residuals
- Simulated timestream from ch6_fill_3d.h5 (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns; WMAP Q-band signal plus B-Machine channel 6 noise properties)

## Outputs

- Four reduced chi2 values (binned I=1.052, binned Q=1.026, destriped I=1.0002, destriped Q=1.0012)
- chi2_table.csv with fields for I/Q x binned/destriped
- Residual maps (input signal minus binned map, input signal minus destriped map) for I and Q

## How to apply

Compute two residual maps per Stokes parameter by subtracting the input signal (WMAP 7-year Q band smoothed by 1 degree) from both the plain-binned map and the destriped map produced by the dst destriper. For each residual map (binned I/Q and destriped I/Q), compute the reduced chi2, i.e. the pixel variance of the residual normalized by the expected white-noise level. The rationale: destriping with 60-second baselines (2000 samples at 33.4 Hz) should act as a high-pass filter affecting only correlated noise, so destriped reduced chi2 values (I=1.0002, Q=1.0012) should sit at unity while binned values (I=1.052, Q=1.026) show correlated-noise excess. Write the four values to a table (chi2_table.csv) with fields for I/Q x binned/destriped and compare against the reference values within a small tolerance.

## Related tools

- **dst** (Produces the destriped maps whose reduced chi2 is compared against the plain-binning baseline) — http://github.com/zonca/dst
- **HEALPix** (Pixelization scheme (Nside 512) on which the residual maps and their reduced chi2 are computed)
- **Python** (Language in which the dst destriper and the chi2 computation/writing to chi2_table.csv are implemented) — http://github.com/zonca/dst
- **HDF5** (File format of the input timestream (ch6_fill_3d.h5) read via dst_io.read_data)

## Examples

```
python dst.py ch6_256.cfg
```

## Evaluation signals

- Destriped reduced chi2 for I and Q fall within a small tolerance of 1.0002 and 1.0012 respectively, i.e. residual noise at the white-noise level
- Binned (no destriping) reduced chi2 for I and Q fall within a small tolerance of 1.052 and 1.026, confirming the metric detects correlated-noise excess
- chi2_table.csv contains exactly four fields covering the I/Q x binned/destriped combinations
- Residual maps are computed against the same input signal smoothed by 1 degree in both cases, so the comparison isolates the effect of destriping

## Limitations

- Reduced chi2 at the white-noise level only certifies removal of low-frequency correlated noise; systematic effects like gain variations or spin-synchronous effects must be handled separately before map-making
- Reference chi2 values are tied to the specific simulation setup (B-Machine channel 6 noise properties at 33.4 Hz, WMAP 7-year Q band signal, 60-second baselines); other datasets need re-derived noise expectations

## Evidence

- [methods] the difference between the input signal and the output before destriping (top of Fig. 3) smoothed by 1◦: [evidence span withheld — anchor status: not_found]
- [intro] Destriping can be thought of as a high-pass filter that only affects the noise in the timestream: [evidence span withheld — anchor status: not_found]
- [results] The destriping algorithm allows the removal of the correlated noise with minimal impact on the signal.: [evidence span withheld — anchor status: not_found]
