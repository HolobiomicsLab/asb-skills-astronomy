---
name: cmb-timestream-simulation
description: 'Use when validating a destriping pipeline with simulated polarimeter
  timelines: a signal-only timestream from a known input map, and a signal-plus-noise
  timestream with white variance and 1/f knee/slope from measured detector noise properties.'
when_to_use_negative:
- The goal is to destripe or map real data — this skill only fabricates simulated
  input timelines; use the dst destriper instead
- The data contain systematic effects (thermally induced gain variations near sunrise/sunset,
  spin-synchronous effects) that must be masked or handled separately before map-making
  — simulation output does not model these
- Small angular scales matter — the 1-degree smoothing of the input WMAP Q-band map
  deliberately suppresses structure in map domain
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: dst
  role: simulation module used to scan the smoothed sky map through the B-Machine
    pointing model and produce the timestream
  repo: http://github.com/zonca/dst
- name: HEALPix
  role: loads and smooths the WMAP 7-year Q-band map by 1 degree before scanning
- name: h5py
  role: writes the simulated timelines (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) to
    a single HDF5 file
- name: Python
  role: host language for the fully parallel destriping/simulation implementation
  repo: http://github.com/zonca/dst
- name: B-Machine
  role: source of the scanning strategy (45 deg elevation, 1 rev/70 s) and channel
    6 white noise variance, 1/f knee frequency and slope
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:b3893a098f68306cac9396fb743e10ca2eb0b60f2e68060f41542d2cac6b6db1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  - dst
  - h5py
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cmb-timestream-simulation

## Summary

Generates simulated CMB polarimeter timestreams by scanning a smoothed WMAP 7-year Q-band map through the B-Machine pointing model, then optionally adding white plus 1/f correlated noise (n = Fa + w), serialized as a single HDF5 file readable by dst_io.read_data.

## When to use

Use when validating a destriping pipeline with simulated polarimeter timelines: a signal-only timestream from a known input map, and a signal-plus-noise timestream with white variance and 1/f knee/slope from measured detector noise properties.

## When NOT to use

- The goal is to destripe or map real data — this skill only fabricates simulated input timelines; use the dst destriper instead
- The data contain systematic effects (thermally induced gain variations near sunrise/sunset, spin-synchronous effects) that must be masked or handled separately before map-making — simulation output does not model these
- Small angular scales matter — the 1-degree smoothing of the input WMAP Q-band map deliberately suppresses structure in map domain

## Inputs

- WMAP 7-year Q-band map (NASA LAMBDA), smoothed by 1 degree gaussian beam in map domain
- B-Machine scanning strategy parameters: 45 deg fixed elevation, 1 revolution per 70 s, Equatorial frame
- B-Machine channel 6 noise properties: white noise variance, 1/f knee frequency and slope
- dst simulation module (github.com/zonca/dst), HEALPix for map manipulation

## Outputs

- timestream_signal_only.h5: single HDF5 file with columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U
- timestream_signal_1f_white.h5: same HDF5 schema, signal plus white + 1/f correlated noise (n = Fa + w)
- ~2000 samples per 60-second baseline interval at 33.4 Hz sampling; ~5e8 samples for a 12-month LATTE-scale run

## How to apply

Download the WMAP 7-year Q-band map from NASA LAMBDA and smooth it by 1 degree in map domain with HEALPix to reduce map-domain noise. Generate the B-Machine pointing timeline: fixed elevation scanning at 45 degrees, 1 revolution every 70 seconds in the Equatorial reference frame (~44.8% sky coverage, median hit count 20 per pixel at Nside 512), sampled at 33.4 Hz; for LATTE-scale design cases extend to ~12 months and mask sections near sunrise and sunset contaminated by thermally induced gain variations. Scan the smoothed sky map through the pointing/rotation model to produce the signal-only timestream. For the noisy variant, add white noise plus 1/f correlated noise using the white noise variance and the knee frequency and slope taken from B-Machine channel 6 noise properties, with the correlated component stored as baselines a multiplied by a sparse matrix F (n = Fa + w). Serialize each timeline to a single HDF5 file with columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U in the big-endian dtype expected by dst_io.read_data, using h5py.

## Related tools

- **dst** (simulation module used to scan the smoothed sky map through the B-Machine pointing model and produce the timestream) — http://github.com/zonca/dst
- **HEALPix** (loads and smooths the WMAP 7-year Q-band map by 1 degree before scanning)
- **h5py** (writes the simulated timelines (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) to a single HDF5 file)
- **Python** (host language for the fully parallel destriping/simulation implementation) — http://github.com/zonca/dst
- **B-Machine** (source of the scanning strategy (45 deg elevation, 1 rev/70 s) and channel 6 white noise variance, 1/f knee frequency and slope)

## Examples

```
import numpy as np, h5py; dtype=np.dtype([('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')]); h5py.File('timestream_signal_only.h5','w').create_dataset('data', shape=(n,), dtype=dtype)
```

## Evaluation signals

- Both output files (timestream_signal_only.h5, timestream_signal_1f_white.h5) exist and open as valid HDF5
- Each file contains the required columns with the exact big-endian dtype [('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2'), ('TEMP', '>f4'), ('Q', '>f4'), ('U', '>f4')]
- Timeline length matches ~2000 samples per 60-second baseline interval at 33.4 Hz (~5e8 samples for a 12-month run)
- The noisy timestream equals the signal-only timestream plus a noise residual consistent with white variance plus 1/f correlated component built from channel 6 knee frequency and slope
- The generation script runs without error and, for the design-case pointing, yields ~44.8% sky coverage with median hit count 20 per pixel at Nside 512

## Limitations

- Sections near sunrise and sunset are contaminated by large thermally induced gain variations and must be masked out before use
- Destriping itself does not remove systematic effects such as gain variations or spin-synchronous effects; they must be dealt with before the map-making stage, so simulated data should not be assumed to cover them
- Destriping only efficiently removes 1/f noise with a knee frequency lower than the scan rate, so noise parameters outside this regime will not validate the destriper properly
- The input WMAP map is smoothed by 1 degree specifically to reduce noise in map domain, limiting fidelity at small angular scales

## Evidence

- [methods] Input signal map and smoothing: [evidence span withheld — anchor status: not_found]
- [methods] Sunrise/sunset contamination masking: [evidence span withheld — anchor status: not_found]
