---
name: 1f-plus-white-noise-synthesis
description: Use when building simulated polarimeter timestreams (signal-only and
  signal-plus-noise) to test a destriping pipeline, e.g. with B-Machine-like 1/f noise
  parameters and ~33.4 Hz sampling.
when_to_use_negative:
- The goal is to destripe an existing real timestream rather than synthesize a simulated
  one (use the dst destriper directly instead).
- The data format is not the dst single-HDF5 layout with TIME/PHI/THETA/PSI/FLAG/TEMP/Q/U
  columns, and dst_io.read_data cannot be adapted to it.
- The 1/f knee frequency is higher than the scan rate — the paper's destriping approach
  assumes correlated noise with knee frequency lower than the scan rate.
tools:
- name: dst
  role: Defines the target HDF5 dtype consumed by dst_io.read_data and the destriping
    noise model (n = Fa + w) the simulated timestream must satisfy
  repo: http://github.com/zonca/dst
- name: HEALPix
  role: Loads and smooths the input sky map (WMAP 7 years Q band, 1 degree smoothing)
    and manages pixelization/hit counting for the pointing timeline
- name: HDF5 (h5py)
  role: Serializes both simulated timestreams into single HDF5 files with the required
    column dtype
- name: B-Machine
  role: Provides the scanning model (45 deg elevation, 70 s revolution, 33.4 Hz sampling)
    and the channel 6 noise properties (white variance, 1/f knee and slope)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:17d78b1fb5e54cd288cdaef12964950c3202f303a5e1883ad012edc2702f0b7c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HDF5 (h5py)
  - HEALPix
  - Python
  - dst
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# 1/f plus white noise synthesis

## Summary

Generates realistic simulated timestreams for CMB polarimeter data by scanning a sky map through a pointing model and adding noise whose timeline follows the destriping data model n = Fa + w, i.e. a white noise component plus a correlated 1/f component represented as baselines multiplied by a sparse matrix F.

## When to use

Use when building simulated polarimeter timestreams (signal-only and signal-plus-noise) to test a destriping pipeline, e.g. with B-Machine-like 1/f noise parameters and ~33.4 Hz sampling.

## When NOT to use

- The goal is to destripe an existing real timestream rather than synthesize a simulated one (use the dst destriper directly instead).
- The data format is not the dst single-HDF5 layout with TIME/PHI/THETA/PSI/FLAG/TEMP/Q/U columns, and dst_io.read_data cannot be adapted to it.
- The 1/f knee frequency is higher than the scan rate — the paper's destriping approach assumes correlated noise with knee frequency lower than the scan rate.

## Inputs

- Sky map (e.g. WMAP 7 years Q band HEALPix map, smoothed by 1 degree in map domain)
- Pointing/rotation model parameters (B-Machine: 45 degree elevation, 70 s revolution, Equatorial frame, 33.4 Hz sampling)
- Noise parameters from a real channel (white noise variance, 1/f knee frequency and slope, e.g. B-Machine channel 6)

## Outputs

- Signal-only timestream HDF5 file (e.g. timestream_signal_only.h5) with TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns in dst_io.read_data dtype
- Signal + 1/f + white noise timestream HDF5 file (e.g. timestream_signal_1f_white.h5) with the same schema
- Pointing timeline covering ~44.8% of the sky (~20 hits per pixel median at Nside 512)

## How to apply

First load a sky map (the paper used WMAP 7 years Q band, smoothed by 1 degree in map domain with HEALPix to reduce noise), then generate a pointing timeline (B-Machine model: fixed 45 degree elevation, one revolution per 70 seconds, Equatorial frame, ~33.4 Hz sampling) and scan the map through it to produce the signal-only timestream. Add noise to a copy of the timestream using white noise variance and 1/f knee frequency and slope taken from real channel noise properties (B-Machine channel 6); in the destriping model the noise timeline is n = Fa + w, with correlated noise modeled as a baselines array a multiplied by a sparse matrix F. Serialize both timelines as single HDF5 files with columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U in the exact dtype expected by dst_io.read_data (big-endian f8 for TIME/PHI/THETA/PSI, i2 FLAG, f4 TEMP/Q/U). Keep baseline length consistent with the downstream destriper: about 60 seconds, i.e. 2000 samples at 33.4 Hz.

## Related tools

- **dst** (Defines the target HDF5 dtype consumed by dst_io.read_data and the destriping noise model (n = Fa + w) the simulated timestream must satisfy) — http://github.com/zonca/dst
- **HEALPix** (Loads and smooths the input sky map (WMAP 7 years Q band, 1 degree smoothing) and manages pixelization/hit counting for the pointing timeline)
- **HDF5 (h5py)** (Serializes both simulated timestreams into single HDF5 files with the required column dtype)
- **B-Machine** (Provides the scanning model (45 deg elevation, 70 s revolution, 33.4 Hz sampling) and the channel 6 noise properties (white variance, 1/f knee and slope))

## Evaluation signals

- Both output files exist and open as valid HDF5 files.
- Each file contains the TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U fields with the exact dst_io.read_data dtype (e.g. ('TIME', '>f8'), ('FLAG', '>i2'), ('Q', '>f4')).
- Timeline length matches ~2000 samples per 60-second baseline interval at the 33.4 Hz sampling rate.
- Signal-only timestream reproduces the smoothed sky map scanned through the pointing model; signal+noise file differs by an n = Fa + w noise timeline with the prescribed white variance and 1/f knee/slope.
- The generation script runs end-to-end without error.

## Limitations

- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility.
- Destriping (and hence this simulated test data) only handles low-frequency correlated noise; systematic effects like gain variations or spin-synchronous effects must be dealt with separately before map-making.
- The noise model assumes 1/f knee frequency lower than the scan rate; higher-knee noise is outside the model's scope.

## Evidence

- [readme] HDF5 input dtype defines the output schema: "currently data needs to be in a single HDF5 file with columns: dtype=[('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2')"
- [methods] Baseline length tied to sampling rate: "we chose a baseline length slightly shorter, of about 60 seconds, i.e. 2000 samples at 33.4 Hz."
- [methods] Input sky map preparation: [evidence span withheld — anchor status: not_found]
- [intro] Scope of correlated noise model: [evidence span withheld — anchor status: not_found]
