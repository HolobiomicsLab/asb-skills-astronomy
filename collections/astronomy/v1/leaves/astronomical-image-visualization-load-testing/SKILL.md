---
name: astronomical-image-visualization-load-testing
description: Use when comparing elapsed load time and resident memory of a multi-GB
  HDS/HDF5 dataset opened in a visualization tool (e.g. GAIA) with datMap memory mapping
  enabled versus disabled.
when_to_use_negative:
- Dataset is small (reasonably-sized, well under RAM) — HDSv5 performs comparably
  to HDSv4 in such tests and the mmap contrast will not be visible
- Goal is raw HDS/HDF5 file read throughput rather than end-user visualization display
  load time in a GUI tool
- Input is a native HDS v4 file — HDSv5/HDF5 memory-mapping behavior applies only
  to HDF5-backed files; v4 migration is not addressed
tools:
- name: GAIA
  role: Visualization tool used to load the 4 GB dataset and measure display load
    time and peak process memory
- name: HDS (HDSv5)
  role: API layer over HDF5 whose datMap/datUnmap anonymous-mmap implementation is
    toggled to create the mapped/unmapped benchmark configurations
  repo: https://github.com/Starlink
- name: HDF5
  role: Underlying storage layer; when mapping is disabled the dataset is accessed
    directly through HDF5
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:24c9cd906e0ac08443fef42c1627de4344cc49f4df1938d62bcb94fa48dbd35a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GAIA
  - HDF5
  - HDS
  - HDS (HDSv5)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# astronomical-image-visualization-load-testing

## Summary

Benchmark skill for measuring load time and peak process memory when a large (e.g. 4 GB) HDS/HDF5 dataset is opened in the GAIA visualization tool through the HDSv5 library, contrasting memory-mapped versus direct HDF5 access.

## When to use

Use when comparing elapsed load time and resident memory of a multi-GB HDS/HDF5 dataset opened in a visualization tool (e.g. GAIA) with datMap memory mapping enabled versus disabled.

## When NOT to use

- Dataset is small (reasonably-sized, well under RAM) — HDSv5 performs comparably to HDSv4 in such tests and the mmap contrast will not be visible
- Goal is raw HDS/HDF5 file read throughput rather than end-user visualization display load time in a GUI tool
- Input is a native HDS v4 file — HDSv5/HDF5 memory-mapping behavior applies only to HDF5-backed files; v4 migration is not addressed

## Inputs

- 4 GB dataset in HDSv5/HDF5 format (e.g. SCUBA-2 acquisition-style file)
- HDSv5 library build with tunable memory mapping (datMap/datUnmap via anonymous mmap)
- GAIA visualization tool (ascl:1403.024)

## Outputs

- gaia_load_benchmark.csv with per-configuration elapsed load time (seconds) and peak resident memory (GB)
- Two benchmark records: memory-mapped vs unmapped (direct HDF5 access) GAIA loads

## How to apply

Obtain a ~4 GB dataset in HDSv5/HDF5 format and open it in GAIA (Draper et al., 2009, ascl:1403.024) via the HDSv5 API layer. Run the load twice: once with memory mapping enabled, where datMap mmaps an anonymous memory area and datUnmap copies data back to the underlying dataset; and once with mapping disabled so the data is read through direct HDF5 dataset access. Record display load time in seconds and peak resident process memory for each configuration. Tabulate elapsed seconds and resident memory side by side (e.g. gaia_load_benchmark.csv). The mapped run should load in a few seconds using tens of MB, while the unmapped run should take roughly an order of magnitude longer (~10 s) and consume memory comparable to the full dataset size (~5 GB), reproducing the Section 5.9 contrast.

## Related tools

- **GAIA** (Visualization tool used to load the 4 GB dataset and measure display load time and peak process memory)
- **HDS (HDSv5)** (API layer over HDF5 whose datMap/datUnmap anonymous-mmap implementation is toggled to create the mapped/unmapped benchmark configurations) — https://github.com/Starlink
- **HDF5** (Underlying storage layer; when mapping is disabled the dataset is accessed directly through HDF5)

## Evaluation signals

- Memory-mapped run: display load time on the order of a few seconds and peak process memory in the tens of MB
- Unmapped run: load time roughly an order of magnitude longer (~10 s) with resident memory comparable to the full dataset size (~5 GB)
- Both runs complete against the same 4 GB dataset so the elapsed-time and memory metrics are directly comparable in gaia_load_benchmark.csv
- Mapped-run memory stays far below dataset size, confirming datMap used an anonymous mmap area rather than copying data into the process

## Limitations

- HDF5 does not natively support memory mapping (due to compression filters and data type conversion), so HDSv5 must mmap an anonymous area and copy back on datUnmap — correct unmap/program-exit behavior must be handled
- datGet/datPut must complain when a locator is memory mapped; behavior for double-mapped primitives or program exit before datUnmap is an open question
- Results are specific to the ~4 GB dataset and workstation used in the Section 5.9 benchmark; absolute times will vary by hardware and dataset chunking

## Evidence

- [methods] datMap must therefore mmap and anonymous memory area to receive the data and then datUnmap must copy the data back: "datMap must therefore mmap and anonymous memory area to receive the data and then datUnmap must copy the data back"
- [intro] a 4 GB dataset was loaded into the GAIA visualization tool: [evidence span withheld — anchor status: not_found]
- [readme] Not supported by HDF5. Presumably because of things like compression filters and data type conversion.: "Memory mapping

Not supported by HDF5. Presumably because of things like
compression filters and data type conversion."
- [discussion] the library performs as well as the HDSv4 implementation in most tests involving reasonably-sized datasets: [evidence span withheld — anchor status: not_found]
