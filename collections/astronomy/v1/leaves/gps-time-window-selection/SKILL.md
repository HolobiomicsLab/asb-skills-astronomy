---
name: gps-time-window-selection
description: Use when an analysis needs a time-bounded slice of strain data around
  a catalogued GW event, expressed in GPS seconds, before downloading via GWpy's remote-data
  access into a TimeSeries.
when_to_use_negative:
- The data are already local (e.g. GWF or HDF5 files on disk) — window selection then
  reduces to a simple crop, not a GWOSC remote fetch
- The analysis is event-rate/catalogue-level (e.g. EventTable.fetch_open_data('GWTC'))
  where no strain time series or GPS window is involved
- The target is proprietary instrumental channels requiring authenticated access rather
  than public GWOSC event data
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: GWpy
  role: TimeSeries.get consumes the GPS start/end window and downloads the strain
    segment from GWOSC directly into a TimeSeries object
  repo: https://gitlab.com/gwpy/gwpy
- name: GWOSC
  role: Public data host whose event catalogue (GWTC) supplies the GPS epochs and
    strain segments the window is defined against
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:8dd9c33088946f93080a32ca5019fc64889c0fb8ca1128b6cd7e21e2d35bf63b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GWOSC
  - GWpy
  - Matplotlib
  - NumPy
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# gps-time-window-selection

## Summary

Selects GPS-second start/end windows around a gravitational-wave event (e.g. GW150914: 1126259446–1126259478, a 32-second span) so that public strain data can be fetched from GWOSC directly into a GWpy TimeSeries for downstream spectral work.

## When to use

Use when an analysis needs a time-bounded slice of strain data around a catalogued GW event, expressed in GPS seconds, before downloading via GWpy's remote-data access into a TimeSeries.

## When NOT to use

- The data are already local (e.g. GWF or HDF5 files on disk) — window selection then reduces to a simple crop, not a GWOSC remote fetch
- The analysis is event-rate/catalogue-level (e.g. EventTable.fetch_open_data('GWTC')) where no strain time series or GPS window is involved
- The target is proprietary instrumental channels requiring authenticated access rather than public GWOSC event data

## Inputs

- Event identifier and GPS epoch from the GWOSC/GWTC event catalogue (e.g. GW150914 at GPS 1126259446)
- Window length in GPS seconds (e.g. 1126259446 to 1126259478 = 32 s)
- Interferometer channel prefix ('H1', 'L1', 'V1')

## Outputs

- A GWpy TimeSeries of strain data covering exactly the chosen GPS window
- A validated start/end pair that downstream ASD computation consumes (fftlength=4 → df=0.25 Hz FrequencySeries)

## How to apply

Identify the event's GPS epoch from the GWOSC event catalogue (e.g. GWTC) and define a window in GPS seconds — the documented GW150914 example uses 1126259446 to 1126259478, giving a 32-second analysis window. Pass the start and end GPS times to GWpy's TimeSeries.get along with the interferometer prefix ('H1' for LIGO-Hanford, 'L1' for LIGO-Livingston) so the time-domain data are downloaded directly into a TimeSeries object. Keep the window wide enough to supply multiple FFT segments for spectral averaging (a 32 s window with fftlength=4 s yields the documented 0.25 Hz frequency spacing in the resulting ASD). The rationale is that GWpy's remote-data access system takes GPS boundaries as its primary query interface, so a correct window is the prerequisite for every downstream step (ASD estimation, filtering, plotting).

## Related tools

- **GWpy** (TimeSeries.get consumes the GPS start/end window and downloads the strain segment from GWOSC directly into a TimeSeries object) — https://gitlab.com/gwpy/gwpy
- **GWOSC** (Public data host whose event catalogue (GWTC) supplies the GPS epochs and strain segments the window is defined against)

## Examples

```
from gwpy.timeseries import TimeSeries; data = TimeSeries.get('H1', 1126259446, 1126259478)
```

## Evaluation signals

- The retrieved TimeSeries spans exactly 32 seconds (1126259446 to 1126259478) at the interferometer's sample rate
- The ASD computed from the window has the documented 0.25 Hz frequency spacing (asd.df)
- The ASD frequency axis starts at f0 and extends to the highest frequency bin expected for a 32-second analysis window
- Downstream steps (e.g. matched-filter SNR) show the expected event signature (SNR spike above 17 for GW150914) only inside the chosen window
- The plot artefact gw150914_asd.png exists and is a valid PNG built from the windowed FrequencySeries

## Limitations

- Window boundaries must be given in GPS seconds, not UTC — mixing epochs silently fetches the wrong segment
- A too-narrow window leaves too few FFT segments for meaningful spectral averaging, degrading the ASD
- GWpy's remote-data access requires network access to GWOSC; errors in the event catalogue epoch propagate to the fetch

## Evidence

- [abstract] remote-data access into a TimeSeries: "GWpy also provides an intuitive remote-data access system for downloading time-domain data directly into a"
- [abstract] ASD example workflow: "downloading public GW event data associated with GW150914, estimating the amplitude spectral density of the strain data, and visualising these in a figure"
- [intro] GWpy purpose: "GWpy is a collaboration-driven Python package providing tools for studying data from ground-based gravitational-wave detectors."
- [readme] README positioning: "GWpy provides a user-friendly, intuitive interface to the common
time-domain and frequency-domain data produced by the LIGO and Virgo
observatories"
