---
name: frequency-domain-visualisation-of-detector-noise
description: Use when the goal is to characterise detector noise from public GW event
  strain data (e.g. GW150914 from GWOSC) by transforming time-domain strain into a
  frequency-domain ASD and rendering it as a figure.
when_to_use_negative:
- The goal is time-domain analysis (e.g. matched-filter SNR around the event time)
  rather than frequency-domain noise characterisation.
- The goal is coherence between two instrumental channels (e.g. accelerometer vs calibrated
  strain), which is a separate GWpy workflow.
- The data are proprietary instrumental channels requiring authentication rather than
  public GWOSC event data accessible via TimeSeries.get.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: GWpy
  role: Provides TimeSeries.get for remote GWOSC data access, TimeSeries.asd for spectral
    estimation, and the Matplotlib-backed plot interface for the ASD figure
  repo: https://gitlab.com/gwpy/gwpy
- name: Matplotlib
  role: Backend supporting GWpy's visualisation interface used to render the ASD FrequencySeries
- name: GWOSC
  role: Source of the public GW event strain data (GW150914 window) downloaded into
    TimeSeries objects
  repo: https://gwosc.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:8d39e4a1c09a1e7913f3ed4d3114c1a7eaca3eaed9f1a665d8379cb4516b4220
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GWOSC
  - GWpy
  - Matplotlib
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# frequency-domain-visualisation-of-detector-noise

## Summary

Download public GWOSC strain TimeSeries around a gravitational-wave event such as GW150914 and estimate the amplitude spectral density (ASD) with GWpy to characterise and visualise detector noise in the frequency domain.

## When to use

Use when the goal is to characterise detector noise from public GW event strain data (e.g. GW150914 from GWOSC) by transforming time-domain strain into a frequency-domain ASD and rendering it as a figure.

## When NOT to use

- The goal is time-domain analysis (e.g. matched-filter SNR around the event time) rather than frequency-domain noise characterisation.
- The goal is coherence between two instrumental channels (e.g. accelerometer vs calibrated strain), which is a separate GWpy workflow.
- The data are proprietary instrumental channels requiring authentication rather than public GWOSC event data accessible via TimeSeries.get.

## Inputs

- Public GWOSC strain data for a GW event (e.g. GW150914), accessed via detector name and GPS window 1126259446-1126259478
- ASD estimation parameters: fftlength=4 s, overlap=2 s, spectral-averaging method ('welch' or 'median')

## Outputs

- GWpy FrequencySeries object containing the amplitude spectral density with 0.25 Hz frequency spacing
- PNG figure (gw150914_asd.png) of the ASD produced via GWpy's Matplotlib-backed plot interface

## How to apply

Retrieve the public strain TimeSeries from GWOSC with GWpy's TimeSeries.get for the documented GW150914 window GPS 1126259446 to 1126259478 (32 seconds), for detectors such as LIGO-Hanford (H1) and LIGO-Livingston (L1). Compute the amplitude spectral density with TimeSeries.asd(4, 2), i.e. fftlength=4 s, which yields a 0.25 Hz frequency spacing; choose the default 'welch' (mean of overlapping FFTs) or 'median' (robust to outliers) spectral-averaging method depending on whether transient outliers are a concern. Render the resulting FrequencySeries with GWpy's Matplotlib-backed plot interface and save it as gw150914_asd.png. When interpreting the figure, expect the ASD to vary by factors of a few across most of the frequency band with notable features such as the 60 Hz power line and its harmonics.

## Related tools

- **GWpy** (Provides TimeSeries.get for remote GWOSC data access, TimeSeries.asd for spectral estimation, and the Matplotlib-backed plot interface for the ASD figure) — https://gitlab.com/gwpy/gwpy
- **Matplotlib** (Backend supporting GWpy's visualisation interface used to render the ASD FrequencySeries)
- **GWOSC** (Source of the public GW event strain data (GW150914 window) downloaded into TimeSeries objects) — https://gwosc.org/

## Examples

```
from gwpy.timeseries import TimeSeries; asd = TimeSeries.get('H1', 1126259446, 1126259478).asd(4, 2); asd.plot().save('gw150914_asd.png')
```

## Evaluation signals

- gw150914_asd.png exists on disk and is a valid PNG image.
- The ASD FrequencySeries has the documented 0.25 Hz frequency spacing, verifiable via asd.df.
- The ASD frequency axis spans from f0 to the highest frequency bin of the 32-second analysis window.
- The plotted ASD shows expected detector-noise structure, e.g. elevated noise around the 60 Hz power line and its harmonics.

## Limitations

- ASD requires network access to GWOSC to fetch the event window; results depend on the chosen spectral-averaging method ('welch' mean is sensitive to outliers, 'median' is robust).
- With fftlength=4 s over the 32-second window, frequency resolution is fixed at 0.25 Hz; shorter windows or fftlengths change the spacing.
- GWpy code must be compatible with Python >= 3.11.
- Narrow-band features such as the 60 Hz mains power line and its harmonics can dominate the visualised ASD.

## Evidence

- [abstract] downloading public GW event data associated with GW150914, estimating the amplitude spectral density of the strain data, and visualising these in a figure: "downloading public GW event data associated with GW150914, estimating the amplitude spectral density of the strain data, and visualising these in a figure"
- [methods] welch spectral averaging: "``'welch'`` - mean of overlapping FFTs (default)"
- [results] Median ASD variation across band: "the ASD varies by factors of a few across most of the
    frequency band, with notable exceptions, e.g. around the 60-Hz power line"
- [readme] GWpy purpose: "GWpy is a collaboration-driven Python package providing tools for
studying data from ground-based gravitational-wave detectors."
