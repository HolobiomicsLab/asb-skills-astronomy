---
name: astropy-unified-io-registry-usage
description: Use when reading time-domain GW data files (GWOSC HDF5 strain) into GWpy
  array classes and you need the attached source, t0, duration, sample_rate and unit
  metadata extracted and validated.
when_to_use_negative:
- Input is proprietary LIGO instrumental channel data requiring remote-data access
  rather than a local GWOSC HDF5 file read
- The goal is frequency-domain analysis (e.g. ASD estimation) with no need to inspect
  or serialize the I/O-attached metadata
- Data are already loaded into a GWpy array class and only filtering steps (highpass/bandpass/notch)
  are needed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GWpy
  role: Provides TimeSeries.read and the unified I/O system that attaches source/sampling
    metadata to array classes
  repo: https://gitlab.com/gwpy/gwpy/badges/main/pipeline.svg
- name: Astropy
  role: Supplies the io/registry infrastructure for unified input/output that GWpy's
    I/O is built on
  repo: https://github.com/astropy/astropy/
- name: Python
  role: Runtime for the read, GPS conversion, and JSON serialization steps
- name: NumPy
  role: Numeric layer underlying GWpy array classes during metadata extraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:743f31e4361dc546f74d36ff600fc367dc4b5b25195ecc9c489a956d86e75a14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Astropy
  - GWpy
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# astropy-unified-io-registry-usage

## Summary

Loads GW detector data (e.g. GWOSC HDF5 strain) through GWpy's unified I/O system, which routes file reads via the Astropy class-based io/registry and attaches descriptive metadata (source/channel, t0, duration, sample_rate/dt, unit) to array classes like TimeSeries.

## When to use

Use when reading time-domain GW data files (GWOSC HDF5 strain) into GWpy array classes and you need the attached source, t0, duration, sample_rate and unit metadata extracted and validated.

## When NOT to use

- Input is proprietary LIGO instrumental channel data requiring remote-data access rather than a local GWOSC HDF5 file read
- The goal is frequency-domain analysis (e.g. ASD estimation) with no need to inspect or serialize the I/O-attached metadata
- Data are already loaded into a GWpy array class and only filtering steps (highpass/bandpass/notch) are needed

## Inputs

- Public GWOSC strain data file (HDF5) covering a GW event window, e.g. GW150914 GPS 1126259446-1126259478
- GWpy array class read target (gwpy.timeseries.TimeSeries)
- GPS time conversion inputs via gwpy.time.to_gps/from_gps

## Outputs

- TimeSeries object with attached metadata attributes (source/channel, t0, span/duration, sample_rate, dt, unit)
- JSON metadata record (strain_metadata.json) with non-null source, t0, duration, sample_rate, unit

## How to apply

Obtain a public GWOSC strain HDF5 file covering the target window (e.g. GW150914 GPS 1126259446-1126259478) and call gwpy.timeseries.TimeSeries.read on it; the read is routed through the class-based Astropy io/registry on which GWpy's unified I/O system is built. On load, GWpy attaches metadata attributes to the array class describing the source of the original data (when appropriate) and the sampling along the relevant physical axis, typically time or frequency. Extract the attributes: source/channel name, t0, span/duration, sample_rate and its reciprocal dt, and unit. Cross-check GPS timestamps using gwpy.time.to_gps/from_gps, remembering the GPS epoch of 1980-01-06 00:00:00 UTC. Serialize the metadata as a JSON record (e.g. strain_metadata.json) and validate that it parses and that sample_rate is consistent with 1/dt.

## Related tools

- **GWpy** (Provides TimeSeries.read and the unified I/O system that attaches source/sampling metadata to array classes) — https://gitlab.com/gwpy/gwpy/badges/main/pipeline.svg
- **Astropy** (Supplies the io/registry infrastructure for unified input/output that GWpy's I/O is built on) — https://github.com/astropy/astropy/
- **Python** (Runtime for the read, GPS conversion, and JSON serialization steps)
- **NumPy** (Numeric layer underlying GWpy array classes during metadata extraction)

## Examples

```
from gwpy.timeseries import TimeSeries; ts = TimeSeries.read('H-H1_GWOSC_4km-1126259446-32.hdf5', 'strain'); json.dump({'source': ts.channel.name, 't0': ts.t0.value, 'duration': ts.duration.value, 'sample_rate': ts.sample_rate.value, 'unit': str(ts.unit)}, open('strain_metadata.json', 'w'))
```

## Evaluation signals

- strain_metadata.json parses as valid JSON
- JSON contains non-null entries for source, t0, duration, sample_rate, and unit
- sample_rate in the JSON is consistent with 1/dt of the loaded TimeSeries
- GPS timestamps round-trip via gwpy.time.to_gps/from_gps using the GPS epoch 1980-01-06 00:00:00 UTC
- t0 and duration match the requested GWOSC window (e.g. GPS 1126259446-1126259478 => duration 32 s)

## Limitations

- Metadata attachment applies 'when appropriate' — source attributes are only present when the original data provenance is known
- The sampling axis described may be time or frequency depending on the array class, so the attached attributes differ between TimeSeries and frequency-series classes
- GWpy requires Python >= 3.11 for current versions, per the methods section

## Evidence

- [abstract] Astropy io/registry routing: "Astropy provides an infrastructure for unified input/output"
- [abstract] remote-data access into TimeSeries: "GWpy also provides an intuitive remote-data access system for downloading time-domain data directly into a"
- [intro] GWpy purpose and interface: "GWpy is a collaboration-driven Python package providing tools for studying data from ground-based gravitational-wave detectors."
- [readme] installation and version check: "python -c "import gwpy; print(gwpy.__version__)""
- [methods] Python version requirement: "**GWpy code must be compatible with Python >= 3.11.**"
- [methods] GWOSC as data source: "The Gravitational-Wave Open Science Centre (GWOSC)"
