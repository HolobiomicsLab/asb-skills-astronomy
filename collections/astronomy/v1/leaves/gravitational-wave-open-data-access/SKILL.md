---
name: gravitational-wave-open-data-access
description: Use when an analysis needs public LIGO/Virgo strain or event data (e.g.
  GW150914 from GWOSC) and it must be retrieved remotely into a TimeSeries object
  before frequency-domain or visualisation steps.
when_to_use_negative:
- The target channels are proprietary LIGO instrumental data (e.g. auxiliary accelerometer
  channels) requiring authenticated access rather than GWOSC public data
- The data are already in hand as a local TimeSeries or file and no remote download
  is needed
- The desired output is a frequency-domain or spectrogram object rather than raw time-domain
  data retrieval
edam_operation: http://edamontology.org/operation_2422
tools:
- name: GWpy
  role: Provides the remote-data access system (TimeSeries.get) that downloads GWOSC
    time-domain data directly into a TimeSeries object
  repo: https://gitlab.com/gwpy/gwpy/
- name: Astropy
  role: Underlying infrastructure for unified input/output supporting GWpy data access
  repo: https://github.com/astropy/astropy/
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:cc9bbc8739d25f72b562dad6475f315cd024e35deedd8fd54985436e50805573
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Astropy
  - GWpy
  - Matplotlib
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gravitational-wave-open-data-access

## Summary

Download public gravitational-wave time-domain data (e.g. GW150914 strain around GPS 1126259446–1126259478) from GWOSC directly into a GWpy TimeSeries, providing the entry point for downstream ASD/spectrogram analysis.

## When to use

Use when an analysis needs public LIGO/Virgo strain or event data (e.g. GW150914 from GWOSC) and it must be retrieved remotely into a TimeSeries object before frequency-domain or visualisation steps.

## When NOT to use

- The target channels are proprietary LIGO instrumental data (e.g. auxiliary accelerometer channels) requiring authenticated access rather than GWOSC public data
- The data are already in hand as a local TimeSeries or file and no remote download is needed
- The desired output is a frequency-domain or spectrogram object rather than raw time-domain data retrieval

## Inputs

- Interferometer prefix string (e.g. 'L1', 'H1') for a GWOSC-hosted channel
- GPS start and end times in seconds (e.g. 1126259446 to 1126259478 for GW150914)
- GWOSC public data catalogues such as GWOSC and GWTC event releases

## Outputs

- gwpy.timeseries.TimeSeries object containing the downloaded strain data for the requested window

## How to apply

Use GWpy's remote-data access system, gwpy.timeseries.TimeSeries.get, passing the interferometer prefix (e.g. 'L1' or 'H1' for LIGO-Livingston or LIGO-Hanford) and the start and end GPS seconds, which transparently downloads the data from GWOSC. For documented event examples, use the GW150914 window 1126259446 to 1126259478 (32 seconds). The call returns a TimeSeries with sample-rate metadata that feeds directly into ASD estimation (TimeSeries.asd with fftlength=4 giving 0.25 Hz spacing) or spectrogram transforms (ts.spectrogram with stride=1, fftlength=4). Validate the retrieval by checking the TimeSeries duration matches the requested GPS window before applying transforms.

## Related tools

- **GWpy** (Provides the remote-data access system (TimeSeries.get) that downloads GWOSC time-domain data directly into a TimeSeries object) — https://gitlab.com/gwpy/gwpy/
- **Astropy** (Underlying infrastructure for unified input/output supporting GWpy data access) — https://github.com/astropy/astropy/

## Examples

```
from gwpy.timeseries import TimeSeries; data = TimeSeries.get("L1", 1126259446, 1126259478)
```

## Evaluation signals

- The returned TimeSeries spans exactly the requested GPS window (e.g. 32 s from 1126259446 to 1126259478)
- The TimeSeries carries correct metadata (sample rate, interferometer prefix) for the requested detector
- Downstream transforms succeed and report documented parameters, e.g. ASD df of 0.25 Hz with fftlength=4
- Retrieval works for both LIGO-Hanford and LIGO-Livingston prefixes over the same event window

## Limitations

- GWpy's remote access covers public GWOSC data; proprietary instrumental channels require separate authenticated access (e.g. token-based credential tooling such as htgettoken)
- Remote download depends on GWOSC availability and network access; failures surface as data-retrieval errors rather than analysis errors
- GWpy code must be compatible with Python >= 3.11 in current development versions

## Evidence

- [abstract] Remote data access / downloading time-domain data directly into a TimeSeries object: "GWpy also provides an intuitive remote-data access system for downloading time-domain data directly into a"
- [methods] GWOSC as the public data source: "The Gravitational-Wave Open Science Centre (GWOSC) .. _GWOSCl: https://gwosc.org/"
- [abstract] GW150914 example workflow combining access and ASD estimation: "downloading public GW event data associated with GW150914, estimating the amplitude spectral density of the strain data, and visualising these in a figure"
- [readme] GWpy's interface to LIGO/Virgo observatory data: "GWpy provides a user-friendly, intuitive interface to the common time-domain and frequency-domain data produced by the LIGO and Virgo"
