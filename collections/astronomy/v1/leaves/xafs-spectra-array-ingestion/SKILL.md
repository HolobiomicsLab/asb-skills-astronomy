---
name: xafs-spectra-array-ingestion
description: Use when input is a raw ASCII XAFS data file (energy, it, i0 columns)
  or an Athena .prj project file that must be loaded into Larch Groups before normalization,
  XANES fitting, or feffit analysis.
when_to_use_negative:
- The spectra are already loaded in Larch Groups and only downstream steps (pre_edge,
  autobk, xftf, feffit) are needed
- The input is a Feff path or feff.inp calculation rather than measured spectra —
  use feffrunner/feffpath instead
- The task is GUI-based visualization of non-XAS modes (e.g. RIXS planes), which Larch
  does not cover with a dedicated processing GUI
tools:
- name: larch (xraylarch)
  role: Provides read_ascii (larch.xafs) and read_athena (larch.io) for ingesting
    ASCII and Athena .prj spectra into Larch Groups
  repo: https://github.com/xraypy/xraylarch
- name: Python
  role: Driver language for calling Larch functions and serializing ingested arrays/parameters
    (e.g. to JSON)
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:34e0e8cc23176668453123485fdc77d757021a7e68d52176081da5f3daabf04a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Larch (xraylarch)
  - Python
  - larch
  - larch (xraylarch)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# xafs-spectra-array-ingestion

## Summary

Ingest raw X-ray absorption spectroscopy data into Larch data structures: read ASCII beamline files (e.g. fe.060) with larch.xafs read_ascii and Athena project files (e.g. fe_athena.prj) with larch.io.read_athena, exposing energy and mu arrays as Larch Groups ready for pre_edge normalization and downstream XAFS processing.

## When to use

Use when input is a raw ASCII XAFS data file (energy, it, i0 columns) or an Athena .prj project file that must be loaded into Larch Groups before normalization, XANES fitting, or feffit analysis.

## When NOT to use

- The spectra are already loaded in Larch Groups and only downstream steps (pre_edge, autobk, xftf, feffit) are needed
- The input is a Feff path or feff.inp calculation rather than measured spectra — use feffrunner/feffpath instead
- The task is GUI-based visualization of non-XAS modes (e.g. RIXS planes), which Larch does not cover with a dedicated processing GUI

## Inputs

- ASCII XAFS data file with energy and detector columns (e.g. fe.060 with it, i0)
- Athena Project file (.prj), e.g. fe_athena.prj with Fe2O3/Fe3C/FeO groups or cyanobacteria.prj
- Larch example data directory xafsdata

## Outputs

- Larch Group with energy, it, i0, and computed mu arrays (ASCII path)
- Group of Groups from read_athena, one child Group per Athena Group with energy/mu arrays and processing parameters
- Optional serialized record (e.g. athena_groups_record JSON) mapping each group to its spectra arrays and parameters

## How to apply

Load ASCII column data with Larch's read_ascii reader, then compute absorbance explicitly as data.mu = -np.log(data.it/data.i0) when only it and i0 are stored, as done for the Fe-foil file fe.060. For Athena projects, call larch.io.read_athena on the .prj file (e.g. xafsdata/fe_athena.prj containing Fe2O3, Fe3C, FeO groups); it returns a Group of Groups, one per Athena Group. Enumerate the returned group members and extract each group's energy and mu spectra arrays plus its saved processing parameters (e0, edge_step, pre-edge/normalization settings such as pre1=-150, pre2=-30, nnorm=1, norm1=150, norm2=850). Rationale: Larch preserves the core IFEFFIT XAFS procedures, so these ingested Groups feed directly into pre_edge, autobk, xftf, and feffit without format conversion.

## Related tools

- **larch (xraylarch)** (Provides read_ascii (larch.xafs) and read_athena (larch.io) for ingesting ASCII and Athena .prj spectra into Larch Groups) — https://github.com/xraypy/xraylarch
- **Python** (Driver language for calling Larch functions and serializing ingested arrays/parameters (e.g. to JSON))

## Examples

```
from larch.io import read_athena; project = read_athena('../xafsdata/fe_athena.prj')
```

## Evaluation signals

- The ingestion script runs without error and the exported record file (e.g. athena_groups_record) exists and is valid JSON
- The record contains a field for every group parsed from the .prj file, each with spectra arrays and processing parameters present
- Extracted e0 and edge_step fields are finite values (e.g. Cu foil e0 = 8980.5 eV, edge_step = 2.729 in the article example)
- After normalization, the normalized mu(E) values settle near ~1 in the post-edge region, consistent with IFEFFIT-core normalization preserved in Larch

## Limitations

- read_athena only exposes what the Athena project saved; groups carry whatever e0/edge_step/normalization parameters were stored at export time
- ASCII files without a mu column require an explicit mu = -np.log(it/i0) computation before pre_edge can be applied
- Larch has no GUI for some related modes (e.g. RIXS processing), so ingestion/analysis there must be scripted

## Evidence

- [results] from larch.xafs import autobk, xftf, feffrunner, feffpath, ff2chi: [evidence span withheld — anchor status: not_found]
