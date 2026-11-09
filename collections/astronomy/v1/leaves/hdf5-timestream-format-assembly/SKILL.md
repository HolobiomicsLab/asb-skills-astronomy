---
name: hdf5-timestream-format-assembly
description: Use when timelines (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) from a CMB
  polarimeter such as B-Machine or a LATTE-scale simulation must be serialized into
  the single HDF5 file format dst expects.
when_to_use_negative:
- When data are already in the dst HDF5 schema — re-assembly is unnecessary; just
  copy the file (e.g. ch6_fill_3d.h5) into the h5/ folder and run dst.py
- When the target tool is not dst and does not require this exact column dtype; the
  format is specific to dst_io.read_data
- When systematic effects such as gain variations or spin-synchronous effects still
  need correction — destriping (and this input assembly for it) must come after those
  are handled separately
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: h5py
  role: Python library used to write the simulated timelines into the single HDF5
    file with the required column dtype
- name: dst
  role: Consumes the assembled HDF5 file via dst_io.read_data; format extensions are
    made by modifying dst_io.read_data
  repo: http://github.com/zonca/dst
- name: HDF5
  role: Binary machine-independent hierarchical file format used for data storage
    of the timestream
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:459181ab3e348d48b51f16df96d976571f56987db37cdc98b4948da4baa78632
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HDF5
  - Python
  - dst
  - h5py
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# HDF5 timestream format assembly

## Summary

Assembles polarimeter timelines (pointing, flags, temperature, Q/U) into a single HDF5 file with the exact column dtype required by dst_io.read_data, so the dst parallel destriper can consume the data.

## When to use

Use when timelines (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) from a CMB polarimeter such as B-Machine or a LATTE-scale simulation must be serialized into the single HDF5 file format dst expects.

## When NOT to use

- When data are already in the dst HDF5 schema — re-assembly is unnecessary; just copy the file (e.g. ch6_fill_3d.h5) into the h5/ folder and run dst.py
- When the target tool is not dst and does not require this exact column dtype; the format is specific to dst_io.read_data
- When systematic effects such as gain variations or spin-synchronous effects still need correction — destriping (and this input assembly for it) must come after those are handled separately

## Inputs

- Simulated or observed polarimeter timelines: signal (I, Q, U) plus pointing (TIME, PHI, THETA, PSI), flags, and temperature
- Noise-added signal-only timestream (white noise variance plus 1/f knee frequency and slope, e.g. from B-Machine channel 6 properties)
- h5py (python-h5py in ubuntu)

## Outputs

- A single HDF5 file (e.g. ch6_fill_3d.h5) with columns dtype=[('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')]
- For the LATTE-scale design case: one file holding about 5e8 (half billion) samples covering ~12 months

## How to apply

Write all timestream data with h5py into one HDF5 file whose columns follow the required dtype: [('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')]; the >f8/>f4 big-endian types and the >i2 flag field are what dst_io.read_data expects. Use TIME, PHI, THETA and PSI to carry the pointing, FLAG to mark masked samples (e.g. sections near sunrise and sunset contaminated by thermally induced gain variations), and TEMP, Q, U for the thermometry and polarized signal channels. Place the file where the destriper expects it (e.g. the dst h5/ folder) and point the configuration file (e.g. ch6_256.cfg) at it. If a different data layout must be supported, modify dst_io.read_data rather than deviating from the on-disk schema.

## Related tools

- **h5py** (Python library used to write the simulated timelines into the single HDF5 file with the required column dtype)
- **dst** (Consumes the assembled HDF5 file via dst_io.read_data; format extensions are made by modifying dst_io.read_data) — http://github.com/zonca/dst
- **HDF5** (Binary machine-independent hierarchical file format used for data storage of the timestream)

## Examples

```
import h5py; f = h5py.File('latte_sim_12mo.h5','w'); f.create_dataset('data', data=timelines, dtype=[('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')])
```

## Evaluation signals

- The output file exists and is readable as HDF5 (e.g. opened with h5py)
- The dataset columns match the required dtype exactly: TIME/PHI/THETA/PSI as >f8, FLAG as >i2, TEMP/Q/U as >f4
- Sample count is on the expected order (e.g. ~5e8 samples for the 12-month LATTE-scale simulation; the 3-day test dataset ch6_fill_3d.h5 is much smaller)
- dst_io.read_data loads the file without error and dst.py runs on it (serially: python dst.py ch6_256.cfg; in parallel: mpirun -np 3 python dst.py ch6_256.cfg)

## Limitations

- The format is currently restricted: 'currently data needs to be in a single HDF5 file'; supporting other data formats requires modifying dst_io.read_data
- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility
- Flagged/missing samples must be handled (removed and gaps filled with white noise) for FFT computation and fixed-samples-per-offset destriping, so FLAG semantics matter for downstream destriping

## Evidence

- [methods] HDF5 as the storage format: [evidence span withheld — anchor status: not_found]
- [discussion] LATTE-scale dataset size: [evidence span withheld — anchor status: not_found]
