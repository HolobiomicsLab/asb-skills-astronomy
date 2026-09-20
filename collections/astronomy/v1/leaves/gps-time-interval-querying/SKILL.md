---
name: gps-time-interval-querying
description: Use when the analysis target is public GWOSC strain data (e.g. GW150914)
  for a known GPS time interval and no channel name or authentication path is available.
when_to_use_negative:
- Accessing proprietary LIGO instrumental channels (e.g. accelerometer data for coherence
  estimation) — that path requires authentication and a channel name, not just an
  interferometer prefix
- Data are already available locally (e.g. an existing HDF5 strain file) and no remote
  query is needed
- Browsing event catalogues rather than strain data — use EventTable.fetch_open_data('GWTC',
  ...) instead of a TimeSeries GPS query
edam_operation: http://edamontology.org/operation_2422
tools:
- name: GWpy
  role: Provides the TimeSeries.get public-data access path used to download GWOSC
    strain into a TimeSeries by prefix and GPS interval
  repo: https://gitlab.com/gwpy/gwpy
- name: Python
  role: Language runtime in which the TimeSeries.get query and HDF5 write are executed
    (GWpy installed via python -m pip install gwpy)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:3c4befacb916af3070564bdaec7e91da957fed7a577dcca1101bd266c518b1c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GWpy
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gps-time-interval-querying

## Summary

Retrieves public gravitational-wave strain data from GWOSC by supplying only an interferometer prefix (e.g. 'L1') and a GPS start/stop interval to GWpy's TimeSeries.get, distinguishing this prefix-based public path from proprietary channel-based access.

## When to use

Use when the analysis target is public GWOSC strain data (e.g. GW150914) for a known GPS time interval and no channel name or authentication path is available.

## When NOT to use

- Accessing proprietary LIGO instrumental channels (e.g. accelerometer data for coherence estimation) — that path requires authentication and a channel name, not just an interferometer prefix
- Data are already available locally (e.g. an existing HDF5 strain file) and no remote query is needed
- Browsing event catalogues rather than strain data — use EventTable.fetch_open_data('GWTC', ...) instead of a TimeSeries GPS query

## Inputs

- Interferometer prefix string (e.g. 'L1')
- GPS start and stop times in integer seconds (e.g. 1126259446, 1126259478)
- GWOSC public data host (gwosc.org) reachable over the network

## Outputs

- GWpy TimeSeries object with duration, sample_rate, and unit populated from GWOSC
- HDF5 file of strain data (e.g. gw150914_strain.hdf5) for downstream ASD/filter analysis

## How to apply

Import GWpy's TimeSeries and issue a single get call with the interferometer prefix ('L1') and the GPS start/stop seconds spanning the event — e.g. 1126259446 to 1126259478 for GW150914, a 32-second span. Do not supply a channel name: for public GWOSC data the prefix plus GPS interval is sufficient, whereas proprietary LIGO instrumental access (e.g. accelerometer channels for coherence estimation) follows a different, authenticated route. Verify the returned TimeSeries metadata: duration must equal the requested 32-second span, and sample_rate and unit must be populated from the GWOSC host. Persist the result to an HDF5 file (e.g. 'gw150914_strain.hdf5') so downstream steps like amplitude spectral density estimation or bandpass/notch filtering (e.g. bandpass(50, 250), notches at 60/120/180 Hz) can reload it without re-querying.

## Related tools

- **GWpy** (Provides the TimeSeries.get public-data access path used to download GWOSC strain into a TimeSeries by prefix and GPS interval) — https://gitlab.com/gwpy/gwpy
- **Python** (Language runtime in which the TimeSeries.get query and HDF5 write are executed (GWpy installed via python -m pip install gwpy))

## Examples

```
from gwpy.timeseries import TimeSeries; data = TimeSeries.get("L1", 1126259446, 1126259478)
```

## Evaluation signals

- Returned TimeSeries duration equals the requested span (32 s for the 1126259446–1126259478 interval)
- sample_rate and unit attributes are non-empty and consistent with the GWOSC-hosted strain release
- The output HDF5 file exists and is readable back as a GWpy TimeSeries covering the same GPS interval
- The call required only the interferometer prefix and GPS times — no channel name — confirming the public-data path was used

## Limitations

- The prefix-plus-GPS-interval pattern applies to public GWOSC data; proprietary observatory channels follow a separate access mechanism and cannot be fetched this way
- Requires network access to gwosc.org; queries are bounded by the data available in the GWOSC public releases
- GWpy code must be compatible with Python >= 3.11 per the article's support policy, so environment version matters

## Evidence

- [abstract] Remote data access / downloading time-domain data directly into a TimeSeries object: "GWpy also provides an intuitive remote-data access system for downloading time-domain data directly into a"
- [intro] GWpy provides tools for studying data from ground-based gravitational-wave detectors: "GWpy is a collaboration-driven Python package providing tools for studying data from ground-based gravitational-wave detectors."
- [readme] GWpy provides a user-friendly, intuitive interface to the common: "GWpy provides a user-friendly, intuitive interface to the common time-domain and frequency-domain data produced by the LIGO and Virgo"
