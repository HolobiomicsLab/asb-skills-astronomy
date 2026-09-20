---
name: spectroscopy-data-serialization-to
description: Use when an Athena Project file (.prj, e.g. fe_athena.prj with Fe2O3/Fe3C/FeO
  XAFS data or cyanobacteria.prj) must be imported into Larch and its per-group spectra
  and processing parameters extracted into a structured JSON record.
when_to_use_negative:
- Input is plain ASCII XAFS data (e.g. fe.060) rather than an Athena project — use
  Larch's read_ascii instead
- Input is not X-ray absorption/fluorescence spectroscopy data (e.g. CIF structures
  or Feff path files, which follow different Larch workflows)
tools:
- name: larch (xraylarch)
  role: Provides larch.io.read_athena to parse the Athena .prj file into a Group of
    Groups and gives access to each group's energy/mu arrays and processing parameters
  repo: https://github.com/xraypy/xraylarch
- name: Python
  role: Host language for the read_athena call, group enumeration, and JSON serialization
    of the extracted records
  repo: https://github.com/xraypy/xraylarch
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:2f28b27dab511478d976a7b67faad37aea8656f8a57f7d0edbb912eb4a659014
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Python
  - larch
  - larch (xraylarch)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# spectroscopy-data-serialization-to-

## Summary

Reads Athena Project (.prj) files into Larch using larch.io.read_athena, which returns a Group of Groups (one per Athena Group), then serializes each group's energy/mu spectra arrays and saved XAFS processing parameters (e0, edge_step, pre-edge/normalization settings) into a structured JSON record for downstream reuse.

## When to use

Use when an Athena Project file (.prj, e.g. fe_athena.prj with Fe2O3/Fe3C/FeO XAFS data or cyanobacteria.prj) must be imported into Larch and its per-group spectra and processing parameters extracted into a structured JSON record.

## When NOT to use

- Input is plain ASCII XAFS data (e.g. fe.060) rather than an Athena project — use Larch's read_ascii instead
- Input is not X-ray absorption/fluorescence spectroscopy data (e.g. CIF structures or Feff path files, which follow different Larch workflows)

## Inputs

- Athena Project file (.prj), e.g. fe_athena.prj containing Fe2O3, Fe3C, and FeO XAFS data
- Other Athena projects such as cyanobacteria.prj (Au standards and unknowns)

## Outputs

- Larch Group of Groups, one sub-Group per Athena Group in the project file
- Structured JSON record (e.g. athena_groups_record) with one field per parsed group containing its energy and mu spectra arrays and saved processing parameters (e0, edge_step, pre-edge/normalization settings)

## How to apply

Load the Athena project file with larch.io.read_athena, which returns a Group of Groups, one for each Athena Group stored in the project (e.g. fe_athena.prj holds Fe2O3, Fe3C, and FeO XAFS data). Enumerate the members of the returned Group, extracting each group's energy and mu spectra arrays. Collect each group's saved processing parameters — e0, edge_step, and pre-edge/normalization settings — that Athena stored with the project. Serialize the groups, spectra arrays, and parameters into a structured JSON record and save it (e.g. as athena_groups_record). The rationale is that Larch, written in Python and built on IFEFFIT-style XAFS analysis (normalization, background subtraction, Fourier transforms), can act as a bridge from Athena GUI workflows to scripted, programmatic analysis of larger data sets.

## Related tools

- **larch (xraylarch)** (Provides larch.io.read_athena to parse the Athena .prj file into a Group of Groups and gives access to each group's energy/mu arrays and processing parameters) — https://github.com/xraypy/xraylarch
- **Python** (Host language for the read_athena call, group enumeration, and JSON serialization of the extracted records) — https://github.com/xraypy/xraylarch

## Examples

```
from larch.io import read_athena; project = read_athena('../xafsdata/fe_athena.prj')
```

## Evaluation signals

- The script runs without error and athena_groups_record exists on disk
- athena_groups_record parses as valid JSON
- The JSON contains a field for every group parsed from the project file (e.g. Fe2O3, Fe3C, FeO for fe_athena.prj)
- Each group entry contains non-empty energy and mu spectra arrays
- Each group entry carries its saved processing parameters (e0, edge_step, pre-edge/normalization settings); parameter values should be plausible against Larch's own examples (e.g. Cu foil e0=8980.5 eV, edge_step=2.729)

## Limitations

- read_athena returns whatever processing parameters Athena saved in the project file; if parameters were not stored, they cannot be recovered from the record
- The record structure is tied to Larch's Group-of-Groups layout, so downstream consumers must expect nested groups rather than a flat table
- Larch's Python support depends on the interpreter version available (the article notes Python 2.7, 3.5, or 3.6 for installation)

## Evidence
