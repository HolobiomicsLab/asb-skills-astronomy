---
name: gravitational-wave-strain-data-handling
description: Use when you need to download public GWOSC strain data for a LIGO/Virgo
  event — given only an interferometer prefix and a GPS start/stop interval — into
  a time-domain TimeSeries object in Python.
when_to_use_negative:
- The data are proprietary LIGO instrumental channels requiring authentication — GWpy's
  public prefix-based path is for GWOSC open data only
- The data are already stored locally in HDF5 or another format — reload directly
  rather than re-fetching from the GWOSC host
- The analysis needs frequency-domain or auxiliary-channel data (e.g., accelerometer
  or mains power monitor channels) rather than open strain data
edam_operation: http://edamontology.org/operation_2422
edam_topics: []
tools:
- name: GWpy
  role: Provides the TimeSeries class and remote-data access system used to download
    public GWOSC strain data directly into a time-domain object
  repo: https://github.com/gwpy/gwpy
- name: Python
  role: Runtime environment for GWpy (>= 3.11 per article); install via pip or conda-forge
- name: GWOSC
  role: Public data host serving the open strain data for events such as GW150914
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:a602908ae31b920b4af8b5390f4b4339f4bd588c794c71e34f9bdf220f65a46b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gwpy/gwpy
  tools:
  - GWOSC
  - GWpy
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gravitational-wave-strain-data-handling

## Summary

Retrieves public gravitational-wave strain data (e.g., LIGO GW150914 open data from GWOSC) directly into a GWpy TimeSeries object and stores it for downstream analyses such as amplitude spectral density estimation.

## When to use

Use when you need to download public GWOSC strain data for a LIGO/Virgo event — given only an interferometer prefix and a GPS start/stop interval — into a time-domain TimeSeries object in Python.

## When NOT to use

- The data are proprietary LIGO instrumental channels requiring authentication — GWpy's public prefix-based path is for GWOSC open data only
- The data are already stored locally in HDF5 or another format — reload directly rather than re-fetching from the GWOSC host
- The analysis needs frequency-domain or auxiliary-channel data (e.g., accelerometer or mains power monitor channels) rather than open strain data

## Inputs

- Interferometer prefix string (e.g., 'L1')
- GPS start and stop times bracketing the event (e.g., 1126259446–1126259478 for GW150914)
- GWOSC public strain data host (https://gwosc.org/)

## Outputs

- GWpy TimeSeries object with duration 32 s, populated sample_rate and unit attributes
- HDF5 file (e.g., gw150914_strain.hdf5) containing the retrieved strain data

## How to apply

Import the GWpy TimeSeries class and call its remote public-data access path with only the interferometer prefix (e.g., 'L1') and GPS start/stop times around the event — for GW150914 that is the 32-second span 1126259446 to 1126259478. Verify the returned TimeSeries metadata: duration must equal the requested 32 s, with sample_rate and unit attributes populated from the GWOSC host. Write the retrieved strain data to an HDF5 file (e.g., 'gw150914_strain.hdf5') so downstream analyses such as ASD estimation can reload it. This prefix-based public access is distinct from proprietary channel-based access, which requires instrument/channel names and authentication. Install GWpy via 'python -m pip install gwpy' or 'conda install -c conda-forge gwpy'.

## Related tools

- **GWpy** (Provides the TimeSeries class and remote-data access system used to download public GWOSC strain data directly into a time-domain object) — https://github.com/gwpy/gwpy
- **Python** (Runtime environment for GWpy (>= 3.11 per article); install via pip or conda-forge)
- **GWOSC** (Public data host serving the open strain data for events such as GW150914)

## Examples

```
from gwpy.timeseries import TimeSeries; data = TimeSeries.get('L1', 1126259446, 1126259478); data.write('gw150914_strain.hdf5')
```

## Evaluation signals

- Returned TimeSeries duration equals the requested 32-second GPS span (1126259446–1126259478 for GW150914)
- sample_rate and unit attributes on the TimeSeries are populated from the GWOSC host
- Output file gw150914_strain.hdf5 exists and is readable as HDF5
- Reloading the HDF5 file yields a TimeSeries of 32 s over the requested GPS interval
- GWpy version is importable and printed via 'import gwpy; print(gwpy.__version__)'

## Limitations

- GWpy requires Python >= 3.11 for current code compatibility
- Public access covers GWOSC open strain data; proprietary instrumental channels require a separate authenticated access path
- Stored output is strain data only — downstream steps (e.g., bandpass 50–250 Hz, notch at 60/120/180 Hz, ASD estimation) must be applied separately

## Evidence

- [abstract] Remote data access / downloading time-domain data directly into a TimeSeries object: [evidence span withheld — anchor status: not_found]
- [abstract] Public GWOSC strain retrieval workflow: "downloading public GW event data associated with GW150914, estimating the amplitude spectral density of the strain data, and visualising these in a figure"
- [intro] GWpy interface scope: "GWpy provides a user-friendly, intuitive interface to the common time-domain and frequency-domain data produced by the LIGO and Virgo observatories"
- [readme] GWpy installation: "python -m pip install gwpy"
- [results] GW150914 signal detection in processed strain: "We can clearly see a large spike (above 17!) at the time of the |GW150914|_ signal!"
