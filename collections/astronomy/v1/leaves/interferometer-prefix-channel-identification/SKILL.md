---
name: interferometer-prefix-channel-identification
description: Use when the analysis goal is to download public GWOSC strain data for
  a known event (e.g. GW150914) and you must decide whether to supply an interferometer
  prefix plus GPS interval rather than an instrument channel name.
when_to_use_negative:
- The target data are proprietary LIGO instrumental channels (e.g. accelerometer or
  calibrated strain used in coherence estimation) — these require channel-based access,
  not a public prefix
- The user already has local strain files and only needs filtering or ASD steps (highpass/bandpass/notch,
  Welch averaging) — no remote retrieval specifier is needed
- The analysis targets auxiliary channels from the GWOSC O3 Auxiliary Channel Data
  Release rather than public event strain data
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3368
tools:
- name: GWpy
  role: Provides the TimeSeries.get remote-data-access path that accepts an interferometer
    prefix plus GPS interval for public GWOSC strain retrieval
  repo: https://github.com/gwpy/gwpy
- name: Python
  role: Execution environment in which the GWpy TimeSeries retrieval and HDF5 write
    are performed (GWpy requires Python >= 3.11)
- name: GWOSC
  role: Open-data host supplying the public strain data addressed by the interferometer
    prefix
  repo: https://gwosc.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:1bcdf462a48d46b485afd13874bfcc3c28e0cb3e971f013b20de764b9d753399
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gwpy/gwpy
  tools:
  - GWOSC
  - GWpy
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3368
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# interferometer-prefix-channel-identification

## Summary

Identifies the correct access specifier for retrieving gravitational-wave strain data through GWpy: a bare interferometer prefix (e.g. 'L1') for public GWOSC event data, as opposed to a proprietary channel name for internal LIGO/Virgo instrumental data.

## When to use

Use when the analysis goal is to download public GWOSC strain data for a known event (e.g. GW150914) and you must decide whether to supply an interferometer prefix plus GPS interval rather than an instrument channel name.

## When NOT to use

- The target data are proprietary LIGO instrumental channels (e.g. accelerometer or calibrated strain used in coherence estimation) — these require channel-based access, not a public prefix
- The user already has local strain files and only needs filtering or ASD steps (highpass/bandpass/notch, Welch averaging) — no remote retrieval specifier is needed
- The analysis targets auxiliary channels from the GWOSC O3 Auxiliary Channel Data Release rather than public event strain data

## Inputs

- Interferometer prefix string (e.g. 'L1', 'H1') identifying the detector
- GPS start and stop times bracketing the event (e.g. 1126259446 to 1126259478 for GW150914)
- GWOSC public strain data host (https://gwosc.org/)

## Outputs

- GWpy TimeSeries object of strain data with populated sample_rate, unit, and 32-second duration metadata
- Optional HDF5 file (e.g. gw150914_strain.hdf5) containing the retrieved strain for downstream ASD estimation

## How to apply

Determine the data provenance first: for open GWOSC event data, call GWpy's TimeSeries.get with only the interferometer prefix (e.g. 'L1' for the LIGO Livingston detector) and the GPS start/stop times spanning the event — for GW150914 this is the 32-second interval 1126259446 to 1126259478. For proprietary instrumental data (e.g. coherence between an accelerometer signal and calibrated strain), a channel-based access path is required instead. After retrieval, verify the returned TimeSeries metadata: duration must equal the requested 32-second span, and the sample_rate and unit attributes should be populated from the GWOSC host. The result can then be written to HDF5 (e.g. 'gw150914_strain.hdf5') for downstream steps such as amplitude spectral density estimation.

## Related tools

- **GWpy** (Provides the TimeSeries.get remote-data-access path that accepts an interferometer prefix plus GPS interval for public GWOSC strain retrieval) — https://github.com/gwpy/gwpy
- **Python** (Execution environment in which the GWpy TimeSeries retrieval and HDF5 write are performed (GWpy requires Python >= 3.11))
- **GWOSC** (Open-data host supplying the public strain data addressed by the interferometer prefix) — https://gwosc.org/

## Examples

```
from gwpy.timeseries import TimeSeries; data = TimeSeries.get("L1", 1126259446, 1126259478); data.write('gw150914_strain.hdf5')
```

## Evaluation signals

- The returned TimeSeries duration equals exactly the requested span (32 seconds for the 1126259446–1126259478 GW150914 interval)
- The TimeSeries sample_rate and unit attributes are populated (non-null) after retrieval from the GWOSC host
- The written output file exists and is readable as HDF5, and reloading it yields a TimeSeries whose duration is 32 s over the requested GPS interval
- No channel name was required in the call — the query succeeded with only the prefix and GPS times, confirming public-data access rather than authenticated channel access

## Limitations

- The prefix-based path covers public GWOSC event strain only; proprietary instrumental data still require authenticated channel-based access
- GWpy code must be compatible with Python >= 3.11, constraining the environment in which the retrieval runs
- GPS times must correctly bracket the event of interest; an incorrect interval will return data without the signal of interest

## Evidence

- [other] Public GWOSC strain data can be retrieved with a single TimeSeries.get call supplying only the interferometer prefix: [evidence span withheld — anchor status: not_found]
- [abstract] GWpy also provides an intuitive remote-data access system for downloading time-domain data directly into a: "GWpy also provides an intuitive remote-data access system for downloading time-domain data directly into a"
- [abstract] accessing proprietary LIGO instrumental data and estimating the coherence: "accessing proprietary LIGO instrumental data and estimating the coherence between an accelerometer signal and the calibrated GW strain data"
- [methods] The Gravitational-Wave Open Science Centre (GWOSC): "The Gravitational-Wave Open Science Centre (GWOSC)"
- [readme] GWpy is a collaboration-driven Python package providing tools for: "GWpy is a collaboration-driven Python package providing tools for studying data from ground-based gravitational-wave detectors."
- [methods] GWpy code must be compatible with Python >= 3.11: "**GWpy code must be compatible with Python >= 3.11.**"
