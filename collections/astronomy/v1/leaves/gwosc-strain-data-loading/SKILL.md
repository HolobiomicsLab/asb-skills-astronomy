---
name: gwosc-strain-data-loading
description: Use when a workflow must ingest a public GWOSC strain file (e.g. HDF5
  covering the GW150914 window GPS 1126259446-1126259478) into GWpy's TimeSeries and
  surface the attached I/O metadata.
when_to_use_negative:
- The task requires transforming or conditioning already-loaded data (e.g. highpass/lowpass/bandpass
  30-300 Hz or notch at 60 Hz) — use the filtering skill instead; this skill only
  covers read-in and metadata attachment
- The target data are proprietary LIGO instrumental channels accessible only via remote-data
  access (NDS2-style downloading directly into a TimeSeries), not a public GWOSC file
  on disk
- The file format has no reader registered in GWpy/Astropy's unified I/O registry
  (e.g. raw frame files without GWpy's GWF backend) — the class-based read would fail
edam_operation: http://edamontology.org/operation_0222
tools:
- name: GWpy
  role: Provides TimeSeries.read, the entry point routed through the Astropy unified
    I/O registry that attaches source, t0, sample_rate, and unit metadata to the array
    class; also supplies gwpy.time.to_gps/from_gps for GPS timestamp cross-checks
  repo: https://gitlab.com/gwpy/gwpy
- name: Astropy
  role: Supplies the infrastructure for unified input/output — the class-based io/registry
    that GWpy's unified I/O mechanism and format-aware read routing are built on
  repo: https://github.com/astropy/astropy
- name: Python
  role: Runtime for executing the TimeSeries.read workflow and serializing the metadata
    record as JSON
- name: NumPy
  role: Underlying array backend of the GWpy array classes into which the strain data
    are read
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2021.100657
    title: 'GWpy: A Python package for gravitational-wave astrophysics'
  source_content_hash: sha256:c5242cf7cbffc038d460aa76aaa2b2e5cae5ac95f4808de8ef81607ff576b8ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Astropy
  - GWpy
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_0222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gwosc-strain-data-loading

## Summary

Loads public GWOSC strain data (HDF5) into a GWpy TimeSeries via the Astropy unified I/O registry, capturing the metadata GWpy attaches on read (source/channel, t0, span/duration, sample_rate and its reciprocal dt, unit) for reuse in a serialized JSON record.

## When to use

Use when a workflow must ingest a public GWOSC strain file (e.g. HDF5 covering the GW150914 window GPS 1126259446-1126259478) into GWpy's TimeSeries and surface the attached I/O metadata.

## When NOT to use

- The task requires transforming or conditioning already-loaded data (e.g. highpass/lowpass/bandpass 30-300 Hz or notch at 60 Hz) — use the filtering skill instead; this skill only covers read-in and metadata attachment
- The target data are proprietary LIGO instrumental channels accessible only via remote-data access (NDS2-style downloading directly into a TimeSeries), not a public GWOSC file on disk
- The file format has no reader registered in GWpy/Astropy's unified I/O registry (e.g. raw frame files without GWpy's GWF backend) — the class-based read would fail

## Inputs

- Public GWOSC strain data file in HDF5 format (e.g. H1 strain covering GW150914, GPS 1126259446-1126259478)
- GWpy >= version compatible with Python 3.11
- Astropy io/registry reader registered for the file format

## Outputs

- gwpy.timeseries.TimeSeries object carrying source/channel, t0, span/duration, sample_rate, dt, and unit metadata
- strain_metadata.json: JSON record of the extracted metadata attributes

## How to apply

Call gwpy.timeseries.TimeSeries.read on the HDF5 file so the read routes through the class-based Astropy io/registry that GWpy's unified I/O system is built on. On read, GWpy attaches metadata attributes describing the source of the original data and the sampling along the physical axis (typically time or frequency): the source/channel name, t0, span/duration, sample_rate (and its reciprocal dt), and unit. Extract these attributes from the resulting TimeSeries object and cross-check the GPS timestamps using gwpy.time.to_gps/from_gps, which use the GPS epoch 1980-01-06 00:00:00 UTC. Serialize the extracted metadata as a JSON record (strain_metadata.json) and verify it parses as JSON with non-null entries and sample_rate consistent with 1/dt.

## Related tools

- **GWpy** (Provides TimeSeries.read, the entry point routed through the Astropy unified I/O registry that attaches source, t0, sample_rate, and unit metadata to the array class; also supplies gwpy.time.to_gps/from_gps for GPS timestamp cross-checks) — https://gitlab.com/gwpy/gwpy
- **Astropy** (Supplies the infrastructure for unified input/output — the class-based io/registry that GWpy's unified I/O mechanism and format-aware read routing are built on) — https://github.com/astropy/astropy
- **Python** (Runtime for executing the TimeSeries.read workflow and serializing the metadata record as JSON)
- **NumPy** (Underlying array backend of the GWpy array classes into which the strain data are read)

## Examples

```
from gwpy.timeseries import TimeSeries; ts = TimeSeries.read('H1_GW150914_strain.hdf5'); print(ts.channel, ts.t0, ts.duration, ts.sample_rate, ts.dt, ts.unit)
```

## Evaluation signals

- strain_metadata.json parses as valid JSON
- JSON contains non-null entries for source, t0, duration, sample_rate, and unit
- sample_rate is consistent with its reciprocal dt (sample_rate == 1/dt)
- GPS timestamps cross-check with gwpy.time.to_gps/from_gps under GPS epoch 1980-01-06 00:00:00 UTC; e.g. the GW150914 window maps to GPS 1126259446-1126259478
- duration matches the expected 32 s window span (1126259478 - 1126259446)

## Limitations

- Metadata attachment is only 'when appropriate' — the source-of-data attributes are added per GWpy array class, so attributes may be absent or null for some classes or formats
- Sampling metadata is axis-specific (time or frequency); TimeSeries metadata like sample_rate does not apply to frequency-domain classes in the same way
- GPS timestamps use the GPS epoch 1980-01-06 00:00:00 UTC and include leap-second conventions — naive UTC arithmetic will mismatch to_gps results
- GWpy requires Python >= 3.11 for current versions; older environments cannot run this workflow as-is

## Evidence

- [abstract] Astropy unified I/O infrastructure: "Astropy provides an infrastructure for unified input/output"
- [abstract] GWpy object-oriented interface: "GWpy is a Python software package that provides an intuitive, object-oriented interface through which to access, process, and visualise data from gravitational-wave detectors."
- [abstract] Remote data access: "GWpy also provides an intuitive remote-data access system for downloading time-domain data directly into a"
- [readme] GWpy package purpose: "GWpy is a collaboration-driven Python package providing tools for
studying data from ground-based gravitational-wave detectors."
- [methods] Version support: "**GWpy code must be compatible with Python >= 3.11.**"
