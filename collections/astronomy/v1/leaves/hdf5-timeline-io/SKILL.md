---
name: hdf5-timeline-io
description: Use when raw or simulated polarimeter timelines (e.g. B-Machine channel
  6 at 33.4 Hz) must be stored or loaded for the dst destriper, or when adapting a
  new dataset to dst's HDF5 input format.
when_to_use_negative:
- The data are already HEALPix maps (FITS) rather than timestreams — destriping input
  must be timeline-level HDF5
- The dataset cannot be consolidated into a single HDF5 file, since dst currently
  requires all data in one file with the fixed dtype
- The timeline still contains uncorrected systematic effects (e.g. thermally induced
  gain variations near sunrise/sunset) that must be masked before map-making; I/O
  formatting alone does not address them
tools:
- name: dst
  role: Provides dst_io.read_data, the reader that consumes the HDF5 timeline files
    for destriping
  repo: http://github.com/zonca/dst
- name: HDF5
  role: Binary machine-independent hierarchical file format for single-file timeline
    storage and parallel I/O
- name: MPI
  role: Enables parallel reading and writing of the HDF5 data across processors during
    destriping
provenance:
  source_task_ids:
  - task_006
  - task_004
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:2ed879c25be3306ff5e75ecb42e8dea823254ede012c8b2fa5e8284f798a7e09
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HDF5
  - MPI
  - Python
  - Trilinos
  - dst
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hdf5-timeline-io

## Summary

Serializes and reads polarimeter timestreams in the dst code's single-file HDF5 layout with the exact dtype (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U), enabling both serial and MPI-parallel destriping runs.

## When to use

Use when raw or simulated polarimeter timelines (e.g. B-Machine channel 6 at 33.4 Hz) must be stored or loaded for the dst destriper, or when adapting a new dataset to dst's HDF5 input format.

## When NOT to use

- The data are already HEALPix maps (FITS) rather than timestreams — destriping input must be timeline-level HDF5
- The dataset cannot be consolidated into a single HDF5 file, since dst currently requires all data in one file with the fixed dtype
- The timeline still contains uncorrected systematic effects (e.g. thermally induced gain variations near sunrise/sunset) that must be masked before map-making; I/O formatting alone does not address them

## Inputs

- Single HDF5 timeline file with columns TIME (f8), PHI (f8), THETA (f8), PSI (f8), FLAG (i2), TEMP (f4), Q (f4), U (f4), all big-endian
- ch6_fill_3d.h5 — 3-day simulated test dataset from figshare (placed in the h5/ folder)
- dst_io.read_data reader in the dst package

## Outputs

- Calibrated timeline arrays (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) consumed by the destriper
- Optionally, written-back HDF5 files (e.g. solved baseline arrays) using the same machine-independent format

## How to apply

Write the timeline to a single HDF5 file with the dst dtype: [('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')]; flagged samples are marked per the FLAG column and gaps are filled with white noise so that each ~60-second baseline holds 2000 samples at 33.4 Hz. Place the file (e.g. the 3-day test dataset ch6_fill_3d.h5 from figshare) inside the h5/ folder of the dst repository. dst_io.read_data loads the file; if your data come in another format, modify dst_io.read_data rather than reshaping the data externally. HDF5 was chosen because it supports binary machine-independent hierarchical data and parallel reading/writing through MPI, so the same file serves both 'python dst.py ch6_256.cfg' and 'mpirun -np N python dst.py ch6_256.cfg' runs.

## Related tools

- **dst** (Provides dst_io.read_data, the reader that consumes the HDF5 timeline files for destriping) — http://github.com/zonca/dst
- **HDF5** (Binary machine-independent hierarchical file format for single-file timeline storage and parallel I/O)
- **MPI** (Enables parallel reading and writing of the HDF5 data across processors during destriping)

## Examples

```
cp ch6_fill_3d.h5 h5/ && python dst.py ch6_256.cfg
```

## Evaluation signals

- File opens as valid HDF5 and the compound dtype matches exactly, including byte order (>f8 for TIME/PHI/THETA/PSI, >i2 for FLAG, >f4 for TEMP/Q/U)
- Sample count is consistent with the destriping baseline length: about 2000 samples per 60-second baseline at the 33.4 Hz sampling rate
- dst_io.read_data loads the file without error and the subsequent 'python dst.py ch6_256.cfg' run completes the I/O stage
- FLAG column values correctly identify flagged samples so gaps can be filled with white noise before FFT/baseline operations

## Limitations

- Data must currently reside in a single HDF5 file; multi-file or alternative-format inputs require modifying dst_io.read_data
- No version numbers are given for HDF5 (or Python/Trilinos/HEALPix), limiting reproducibility of the I/O stack

## Evidence

- [methods] HDF5 is the most widespread file format for new applications in the scientific community: [evidence span withheld — anchor status: not_found]
- [methods] supports parallel reading and writing through MPI: [evidence span withheld — anchor status: not_found]
