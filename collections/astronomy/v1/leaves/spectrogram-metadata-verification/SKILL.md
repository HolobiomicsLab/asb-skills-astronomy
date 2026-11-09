---
name: spectrogram-metadata-verification
description: Use when a GWpy TimeSeries (e.g., L1 strain from GWOSC around GW150914)
  has been transformed with ts.spectrogram() and the resulting Spectrogram's time/frequency
  binning (dt, df, shape) must be verified before rendering or further analysis.
when_to_use_negative:
- Input is already a frequency-domain quantity (PSD/ASD or Spectrogram) rather than
  a raw time-domain observatory TimeSeries requiring the time-frequency transform
- The analysis goal is a single averaged spectrum for the whole segment, not a time-resolved
  Spectrogram, so stride/dt binning metadata does not apply
- The strain data cannot be fetched through GWpy's remote-data access system from
  GWOSC (no network or non-GWOSC channel source), so the expected dt/df validation
  values are not meaningful
edam_operation: http://edamontology.org/operation_2928
tools:
- name: GWpy
  role: Provides TimeSeries.spectrogram() (windowed-FFT machinery) to build the Spectrogram,
    the object attributes (shape, dt, df) inspected for verification, and the Matplotlib-backed
    plotting interface used to write spectrogram.png
  repo: https://github.com/gwpy/gwpy
- name: Matplotlib
  role: Backend supporting GWpy's visualisation interface used to render the verified
    Spectrogram figure
- name: NumPy
  role: Underlying numerical array layer over which the Spectrogram's shape/binning
    metadata are reported
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:86c0a976f1c7c33aeef674d5378e2ae1d89ef1b10cba91a5e35c0b85d107fc0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gwpy/gwpy
  tools:
  - GWpy
  - Matplotlib
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spectrogram-metadata-verification

## Summary

Validates the metadata of a GWpy Spectrogram (shape, dt, df) generated from LIGO/Virgo TimeSeries strain data via ts.spectrogram(), ensuring the time-frequency binning matches the configured stride and FFT length before figures or downstream analysis are produced.

## When to use

Use when a GWpy TimeSeries (e.g., L1 strain from GWOSC around GW150914) has been transformed with ts.spectrogram() and the resulting Spectrogram's time/frequency binning (dt, df, shape) must be verified before rendering or further analysis.

## When NOT to use

- Input is already a frequency-domain quantity (PSD/ASD or Spectrogram) rather than a raw time-domain observatory TimeSeries requiring the time-frequency transform
- The analysis goal is a single averaged spectrum for the whole segment, not a time-resolved Spectrogram, so stride/dt binning metadata does not apply
- The strain data cannot be fetched through GWpy's remote-data access system from GWOSC (no network or non-GWOSC channel source), so the expected dt/df validation values are not meaningful

## Inputs

- GWpy TimeSeries of LIGO/Virgo strain data downloaded from GWOSC (e.g., TimeSeries.get("L1", 1126259446, 1126259478))
- Spectrogram call parameters: stride (e.g., 1 s), fftlength (e.g., 4 s), and optional overlap, plus the spectral averaging method (e.g., 'median')

## Outputs

- GWpy Spectrogram (PSD Spectrogram by default) with verified shape (time_bins, frequency_bins), dt = 1.0 s, and df = 0.25 Hz
- Rendered figure saved as spectrogram.png via GWpy's Matplotlib-backed plotting interface

## How to apply

After downloading a 32-second strain series via gwpy.timeseries.TimeSeries.get("L1", 1126259446, 1126259478), build the Spectrogram with ts.spectrogram(stride=1, fftlength=4), giving 1-second time bins and 4-second FFTs. Inspect the returned object: confirm its shape is reported as (time_bins, frequency_bins), that dt equals 1.0 s (the stride), and that df equals 0.25 Hz (1/fftlength). Confirm the underlying PSD estimation used the 'median' spectral averaging method, since the median of overlapping FFTs is robust to outliers. Only after these checks pass, render the Spectrogram with GWpy's Matplotlib-backed plotting interface and save it as spectrogram.png.

## Related tools

- **GWpy** (Provides TimeSeries.spectrogram() (windowed-FFT machinery) to build the Spectrogram, the object attributes (shape, dt, df) inspected for verification, and the Matplotlib-backed plotting interface used to write spectrogram.png) — https://github.com/gwpy/gwpy
- **Matplotlib** (Backend supporting GWpy's visualisation interface used to render the verified Spectrogram figure)
- **NumPy** (Underlying numerical array layer over which the Spectrogram's shape/binning metadata are reported)

## Examples

```
from gwpy.timeseries import TimeSeries; ts = TimeSeries.get("L1", 1126259446, 1126259478); spec = ts.spectrogram(stride=1, fftlength=4); print(spec.shape, spec.dt, spec.df)
```

## Evaluation signals

- Spectrogram object reports the expected shape as (time_bins, frequency_bins) — for a 32 s segment at stride=1, 32 time bins
- Spectrogram.dt equals 1.0 s, matching the configured stride
- Spectrogram.df equals 0.25 Hz, matching 1/fftlength for fftlength=4
- Underlying PSD estimation used the 'median' method (median of overlapping FFTs, robust to outliers) rather than the default 'welch' mean
- Figure spectrogram.png is written only after the metadata checks pass, via GWpy's Matplotlib-backed interface

## Limitations

- Expected dt = 1.0 s and df = 0.25 Hz are valid only for stride=1 and fftlength=4; changing stride, fftlength, or overlap changes the expected metadata
- Median spectral averaging is robust to outliers but differs from the default 'welch' (mean of overlapping FFTs) and 'bartlett' (mean of non-overlapping FFTs) methods, affecting PSD values though not binning metadata
- Remote data access depends on downloading from GWOSC, so availability/network issues block the verification workflow
- GWpy code requires Python >= 3.11, so older environments cannot run the transform

## Evidence

- [abstract] Many research applications require transforming the ‘raw’ time-domain data recorded at an observatory into the frequency domain, or another format in order to study the features of the data.: "Many research applications require transforming the ‘raw’ time-domain data recorded at an observatory into the frequency domain, or another format"
- [other] Source card finding on spectrogram method: [evidence span withheld — anchor status: not_found]
- [other] Validation workflow step from source card: [evidence span withheld — anchor status: not_found]
- [methods] Median averaging rationale: "``'median'`` - median of overlapping FFTs (robust to outliers)"
- [readme] GWpy README scope statement: "GWpy provides a user-friendly, intuitive interface to the common time-domain and frequency-domain data produced by the LIGO and Virgo observatories"
