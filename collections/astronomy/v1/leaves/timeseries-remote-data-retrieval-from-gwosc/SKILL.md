---
name: timeseries-remote-data-retrieval-from-gwosc
description: Use when you need to download public GWOSC strain data for a GW event
  directly into a TimeSeries by supplying only an interferometer prefix ('L1'/'H1')
  and a GPS start/stop interval, with no channel name.
when_to_use_negative:
- Accessing proprietary LIGO instrumental channels (e.g. accelerometer signals for
  coherence estimation) — that path is channel-based and distinct from this public
  prefix-based GWOSC access
- Fetching event catalog metadata rather than strain time series — use the EventTable.fetch_open_data('GWTC')
  path instead
- Retrieving auxiliary/trend channel data — those come from separate GWOSC releases
  (O3 Auxiliary Channel Data Release, O3 Auxiliary Trend Release)
tools:
- name: GWpy
  role: Provides the TimeSeries class and remote-data access system used to download
    public GWOSC strain data directly into a TimeSeries object
  repo: https://gitlab.com/gwpy/gwpy
- name: Python
  role: Runtime environment for GWpy (installable via `python -m pip install gwpy`);
    GWpy code must be compatible with Python >= 3.11
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:6bef86d3dda9e7054c5fc14b2aed319df61a018368f9aeb39beac27d7ec949db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GWpy
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# timeseries-remote-data-retrieval-from-gwosc

## Summary

Retrieves public gravitational-wave strain data from GWOSC directly into a GWpy TimeSeries object using only an interferometer prefix (e.g. 'L1') and GPS start/stop times, without needing channel names or authentication. This is GWpy's remote-data access path demonstrated on the GW150914 event.

## When to use

Use when you need to download public GWOSC strain data for a GW event directly into a TimeSeries by supplying only an interferometer prefix ('L1'/'H1') and a GPS start/stop interval, with no channel name.

## When NOT to use

- Accessing proprietary LIGO instrumental channels (e.g. accelerometer signals for coherence estimation) — that path is channel-based and distinct from this public prefix-based GWOSC access
- Fetching event catalog metadata rather than strain time series — use the EventTable.fetch_open_data('GWTC') path instead
- Retrieving auxiliary/trend channel data — those come from separate GWOSC releases (O3 Auxiliary Channel Data Release, O3 Auxiliary Trend Release)

## Inputs

- Interferometer prefix string (e.g. 'L1' for LIGO Livingston)
- GPS start time in seconds (e.g. 1126259446 for GW150914)
- GPS stop time in seconds (e.g. 1126259478)
- Optional: destination filename for serialized HDF5 output (e.g. 'gw150914_strain.hdf5')

## Outputs

- gwpy TimeSeries object containing strain data with duration, sample_rate, and unit attributes populated
- HDF5 file (e.g. 'gw150914_strain.hdf5') storing the retrieved TimeSeries for downstream reuse

## How to apply

Import the GWpy TimeSeries class and call the public-data access path TimeSeries.get('L1', 1126259446, 1126259478) — the documented prefix-based query pattern for the 32-second GW150914 interval. Supply only the interferometer prefix and GPS start/stop times; for public GWOSC data no proprietary channel name or credential is required, which distinguishes this path from GWpy's channel-based access to LIGO instrumental data. After retrieval, verify the returned TimeSeries metadata: duration should equal the requested 32-second span, and the sample_rate and unit attributes should be populated from the GWOSC host. Optionally write the strain to 'gw150914_strain.hdf5' so downstream analyses (e.g. amplitude spectral density estimation) can reload it without re-fetching.

## Related tools

- **GWpy** (Provides the TimeSeries class and remote-data access system used to download public GWOSC strain data directly into a TimeSeries object) — https://gitlab.com/gwpy/gwpy
- **Python** (Runtime environment for GWpy (installable via `python -m pip install gwpy`); GWpy code must be compatible with Python >= 3.11)

## Examples

```
from gwpy.timeseries import TimeSeries; data = TimeSeries.get("L1", 1126259446, 1126259478); data.write("gw150914_strain.hdf5")
```

## Evaluation signals

- Returned TimeSeries duration equals the requested GPS span (32 s for the GW150914 example, 1126259446–1126259478)
- sample_rate attribute is populated (non-zero) and consistent with the GWOSC host metadata
- unit attribute is populated from the GWOSC host (strain data, not dimensionless/empty)
- TimeSeries GPS start/end match the requested interval exactly
- If serialized, 'gw150914_strain.hdf5' exists, is readable as HDF5, and reloads to a TimeSeries with the same 32-s duration

## Limitations

- This skill covers only public GWOSC data access via interferometer prefix; proprietary channel-based access requires different inputs and credentials
- GWpy requires Python >= 3.11, so the retrieval environment must satisfy that minimum version
- Download requires live network access to the GWOSC host (gwosc.org); retrieval will fail offline or if the requested interval is not publicly released

## Evidence

- [abstract] Remote data access mechanism: "GWpy also provides an intuitive remote-data access system for downloading time-domain data directly into a"
- [abstract] GW150914 public data download use case: "downloading public GW event data associated with GW150914, estimating the amplitude spectral density of the strain data, and visualising these in a figure"
- [readme] GWpy scope for detector data: "GWpy provides a user-friendly, intuitive interface to the common time-domain and frequency-domain data produced by the LIGO and Virgo"
- [methods] Python version constraint for retrieval environment: "**GWpy code must be compatible with Python >= 3.11.**"
