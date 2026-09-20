---
name: hdf5-dataset-access
description: Use when opening Starlink HDS/NDF-format data through the HDF5-backed
  HDSv5 API (e.g. loading a large multi-GB dataset into GAIA) or when benchmarking
  load time and resident memory with datMap memory mapping enabled vs disabled.
when_to_use_negative:
- Input is a legacy HDS v4 file — migration of HDS v4 files to HDF5 is not addressed
  and HDSv5 cannot open them directly
- The goal is writing type-converted data through H5Dwrite directly — HDS string-to-numeric
  type conversion must happen in datPut before the HDF5 call, so raw H5Dwrite bypasses
  that behavior
- The workload relies on unsupported HDF5-native memory mapping of the dataset file
  (compression filters, type conversion) rather than the anonymous-mmap copy-back
  scheme
tools:
- name: HDSv5 (libhdsh5)
  role: Provides the HDS API re-implemented over HDF5; drop-in replacement so Starlink
    software can read HDF5 files behind the scenes
  repo: https://github.com/Starlink
- name: HDF5
  role: Underlying storage layer accessed via H5Dwrite/H5Dread-style calls wrapped
    by the HDSv5 layer
- name: GAIA
  role: Visualization client used to load the 4 GB dataset through HDSv5 and measure
    display load time and peak process memory
- name: h5py
  role: General-purpose tool that can read the resulting HDF5 files outside the HDS
    layer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:f9a345fdc483a27d780972e37e0593a7d48cbcac9537f97411560403898807cb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GAIA
  - HDF5
  - HDS
  - HDSv5 (libhdsh5)
  - h5py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hdf5-dataset-access

## Summary

Access and load scientific datasets through the HDSv5 layer that re-implements the Starlink Hierarchical Data System (HDS) API on top of HDF5, including evaluation of datMap memory-mapping behavior (anonymous mmap with copy-back on datUnmap) versus direct HDF5 dataset access.

## When to use

Use when opening Starlink HDS/NDF-format data through the HDF5-backed HDSv5 API (e.g. loading a large multi-GB dataset into GAIA) or when benchmarking load time and resident memory with datMap memory mapping enabled vs disabled.

## When NOT to use

- Input is a legacy HDS v4 file — migration of HDS v4 files to HDF5 is not addressed and HDSv5 cannot open them directly
- The goal is writing type-converted data through H5Dwrite directly — HDS string-to-numeric type conversion must happen in datPut before the HDF5 call, so raw H5Dwrite bypasses that behavior
- The workload relies on unsupported HDF5-native memory mapping of the dataset file (compression filters, type conversion) rather than the anonymous-mmap copy-back scheme

## Inputs

- 4 GB HDSv5/HDF5-format dataset (e.g. a SCUBA-2 acquisition file with data plus table structures)
- HDSv5 library (Starlink/hds-v5) providing the HDS API over HDF5
- GAIA visualization tool (Draper et al., 2009, ascl:1403.024) as the loading client
- Tuning parameter controlling whether datMap memory mapping is enabled or disabled

## Outputs

- Loaded in-memory dataset accessible via HDS locators (or h5py over the same .h5 file)
- Load-time and peak-process-memory metrics per configuration (e.g. gaia_load_benchmark.csv with elapsed seconds and resident memory for mapped vs unmapped runs)

## How to apply

Open the dataset through the HDSv5 library, which implements the HDS application interface (about 150 API functions) as a layer over HDF5. For mapped access, datMap allocates an anonymous memory area to receive the data and datUnmap copies it back to the underlying HDF5 dataset, so mapped loads should complete in a few seconds with process memory in the tens of MB. With memory mapping disabled (direct HDF5 dataset access), a 4 GB dataset loaded into GAIA took about ten seconds and consumed roughly 5 GB of resident memory, so unmapped reads of large files require memory comparable to the full dataset size. Expect structural differences in the file layout: arrays of structures are stored as /HDSCELL groups with type attributes, and hdsTrace strips '/HDSCELL' occurrences to reconstruct HDS syntax such as .HISTORY.RECORDS(5). Tabulate elapsed load time and peak resident memory for mapped vs unmapped configurations to quantify the memory-mapping benefit.

## Related tools

- **HDSv5 (libhdsh5)** (Provides the HDS API re-implemented over HDF5; drop-in replacement so Starlink software can read HDF5 files behind the scenes) — https://github.com/Starlink
- **HDF5** (Underlying storage layer accessed via H5Dwrite/H5Dread-style calls wrapped by the HDSv5 layer)
- **GAIA** (Visualization client used to load the 4 GB dataset through HDSv5 and measure display load time and peak process memory)
- **h5py** (General-purpose tool that can read the resulting HDF5 files outside the HDS layer)

## Examples

```
import h5py; f = h5py.File('dataset.hdf5', 'r'); dset = f['/HISTORY/RECORDS/HDSCELL(1)/DATE'][:]
```

## Evaluation signals

- Mapped (datMap) run: load time on the order of a few seconds and resident memory in the tens of MB
- Unmapped run: load time roughly an order of magnitude longer (~10 s for 4 GB) and peak memory comparable to the full dataset size (~5 GB)
- Structure access via hdsTrace returns correct HDS syntax (e.g. .HISTORY.RECORDS(5)) after stripping '/HDSCELL' group names
- All but a handful of the ~150 HDS API functions respond; only low-level implementation-detail query routines are expected to be unimplemented
- datGet/datPut complain when the locator is memory mapped, and datAnnul/hdsClose handle pending copy-back correctly

## Limitations

- datMap must mmap an anonymous memory area and datUnmap copies data back — HDF5 itself does not support memory mapping, so behavior differs from classic HDS and from native HDF5 mmap expectations
- Arrays of structures are emulated with explicit HDSCELL collection groups; slices and vectorization of structure arrays break this layout
- hdsdim is currently a 32-bit int while HDF5 uses unsigned 64-bit hsize_t; dimension-order transposition (Fortran vs C order) is required
- Behavior of program termination before datUnmap, double-mapping a primitive, and closing mmapped arrays via hdsClose/datAnnul needs care
- HDS v4 file migration to HDF5 is not yet addressed; no wrapper for mixed-format forwarding exists yet

## Evidence

- [intro] a 4 GB dataset was loaded into the GAIA visualization tool (Draper et al., 2009, ascl:1403.024): [evidence span withheld — anchor status: not_found]
- [discussion] the library performs as well as the HDSv4 implementation in most tests involving reasonably-sized datasets: [evidence span withheld — anchor status: not_found]
