---
name: spectroscopic-line-list-file-handling
description: Use when a task requires acquiring molecular line list files from exomol.com
  or HITRAN/HITEMP, laying them out in PyExoCross's expected directory structure,
  or converting between ExoMol and HITRAN formats before running post-processing.
when_to_use_negative:
- When the line list data are already organized in the ExoMol or HITRAN directory
  layout expected by PyExoCross and no format conversion or filtering is required
- When the analysis target is an atomic spectrum requiring NIST or Kurucz data and
  no molecular line list files exist to handle
- When the task is computing cross-section profiles or lifetimes themselves rather
  than acquiring and arranging the underlying line list files
edam_operation: http://edamontology.org/operation_2422
tools:
- name: PyExoCross
  role: Program that imports/exports ExoMol and HITRAN/HITEMP line list formats, converts
    between them, and applies batch-download code for ExoMol files
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Fortran application of which PyExoCross is the Python adaptation for post-processing
    the same line lists
  repo: https://github.com/ExoMol/ExoCross
- name: Python
  role: Runtime environment (3.8-3.12) in which PyExoCross is executed on the line
    list files
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:7eb65b7a1bd299814d74c677163ce28ea7ab00debb05d4263a64a6a7f4998ddc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoCross
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spectroscopic-line-list-file-handling

## Summary

Skills for obtaining, organizing, converting, and reading large molecular line list files (def, states, transitions) from the ExoMol, ExoMolHR, ExoAtom, HITRAN, and HITEMP databases so they can be consumed by PyExoCross for spectroscopic post-processing.

## When to use

Use when a task requires acquiring molecular line list files from exomol.com or HITRAN/HITEMP, laying them out in PyExoCross's expected directory structure, or converting between ExoMol and HITRAN formats before running post-processing.

## When NOT to use

- When the line list data are already organized in the ExoMol or HITRAN directory layout expected by PyExoCross and no format conversion or filtering is required
- When the analysis target is an atomic spectrum requiring NIST or Kurucz data and no molecular line list files exist to handle
- When the task is computing cross-section profiles or lifetimes themselves rather than acquiring and arranging the underlying line list files

## Inputs

- ExoMol line list files (.def definition file, .states file, .trans transitions file) downloaded from exomol.com
- ExoMol .def, .def., and .adef. definition files
- HITRAN/HITEMP line list files keyed by HITRAN SpeciesID
- ExoMolHR quantum number metadata (from exomol.com/exomolhr/qn/)
- PyExoCross .inp input file specifying Database, Molecule, Isotopologue, Dataset, ReadPath, SavePath, LogFilePath

## Outputs

- Correctly laid-out line list directory tree (ReadPath/Molecule/Isotopologue/Dataset/) readable by PyExoCross
- Converted line list files in the target format (ExoMol or HITRAN)
- Filtered states/transitions subsets produced by UncFilter, Threshold, or QNsFilter

## How to apply

First obtain the line list files (.def definition file plus states and transitions files, e.g. for 24Mg-1H Yadin) from exomol.com or use PyExoCross's batch-download code to automate retrieval from the ExoMol database. Arrange files in the standard directory layout ReadPath/Molecule/Isotopologue/Dataset/ (e.g. ReadPath/MgH/24Mg-1H/Yadin/), and point ReadPath, SavePath, and LogFilePath to these locations in the .inp input file. Identify the database type (ExoMol, ExoMolHR, ExoAtom, HITRAN, HITEMP) and set Database, Molecule, Isotopologue, Dataset, and SpeciesID accordingly; SpeciesID for HITRAN comes from the HITRANOnline isotopologue metadata, and ExoMol definition files come as .def, .def., or .adef.. When interoperability is needed, use PyExoCross's conversion functions to move data between the ExoMol (ExoMol, ExoMolHR, ExoAtom) and HITRAN (HITRAN, HITEMP) formats, optionally applying UncFilter, Threshold, or QNsFilter during conversion. Then run the program with `python run.py -p input_filepath` so the states and transitions files feed downstream functions such as lifetimes, specific heats, or cross-sections.

## Related tools

- **PyExoCross** (Program that imports/exports ExoMol and HITRAN/HITEMP line list formats, converts between them, and applies batch-download code for ExoMol files) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Fortran application of which PyExoCross is the Python adaptation for post-processing the same line lists) — https://github.com/ExoMol/ExoCross
- **Python** (Runtime environment (3.8-3.12) in which PyExoCross is executed on the line list files)

## Examples

```
git clone https://github.com/Beryl-Jingxin/PyExoCross.git && python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The directory tree matches the required layout (e.g. ReadPath/MgH/24Mg-1H/Yadin/ containing def, states, and transitions files) so PyExoCross resolves all files from the .inp settings
- A converted/filtered file parses without error and retains all levels or transitions expected after UncFilter, Threshold, or QNsFilter are applied
- Downstream post-processing runs (e.g. Lifetimes=1 or SpecificHeats=1) complete and produce the expected output table (e.g. 24Mg-1H_Yadin_lifetimes.csv) under SavePath
- Temperatures or levels derived from the files span the ranges declared in the .def file (e.g. 1 K to Tmax in 1 K steps for partition-function work)

## Limitations

- Files must be placed under the standard ExoMol directory layout ReadPath/Molecule/Isotopologue/Dataset/; non-standard layouts are not resolved automatically
- HITRAN SpeciesID must be looked up externally from HITRANOnline isotopologue metadata; it is not inferred
- Support for VAMDC databases was still pending ('Databases from VAMDC' marked with an in-progress icon) in the README

## Evidence

- [readme] PyExoCross can convert data format between ExoMol and HITRAN.: [evidence span withheld — anchor status: external_doc]
- [methods] Definition files for ExoMol and ExoAtom databases available at exomol.com: [evidence span withheld — anchor status: not_found]
- [abstract] Batch download of ExoMol line list files: "extra code for users to automate the batch download of line list files from the ExoMol data base"
- [readme] PyExoCross designed for post-processing huge ExoMol molecular line lists and HITRAN/HITEMP: "PyExoCross is designed for postprocessing the huge molecular line lists generated by the ExoMol project and other similar initiatives such as the HITRAN and HITEMP."
