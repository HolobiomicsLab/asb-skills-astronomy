---
name: line-list-format-conversion
description: Use when an ExoMol-format line list (e.g. .states.bz2/.trans.bz2 files
  like 24Mg-1H__XAB) must be emitted as fixed-width HITRAN2004 160-character records,
  or vice versa, for cross-database interoperability.
when_to_use_negative:
- The line list is already in the target format and only direct import is needed —
  PyExoCross supports direct import/export of ExoMol and HITRAN/HITEMP formats without
  conversion
- The goal is computing spectra, cross-sections, or derived properties (partition
  functions, lifetimes, oscillator strengths) rather than reformatting the line list
- Required quantum number labels cannot be sourced from the dataset .def metadata
  and QNsFilter is enabled, so the conversion mapping would be incomplete
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: PyExoCross
  role: Primary tool performing the ExoMol-to-HITRAN (and reverse) data format conversion,
    driven by an .inp input file with Database, ConversionFormat, ConversionFrequncyRange,
    ConvUncFilter, and ConvThreshold settings
  repo: https://github.com/Beryl-Jingxin/PyExoCross.git
- name: Python
  role: Runtime environment in which PyExoCross executes; PyExoCross is a Python adaptation
    of the ExoCross Fortran application and is runnable as `python run.py -p input_filepath`
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Fortran predecessor whose functionality PyExoCross adapts; conceptual reference
    for the conversion routine
  repo: https://github.com/ExoMol/ExoCross
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:c119022d03ae503fb406a4faf355c22cf48c958a33e38a9882d81d1b3f7ea862
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross.git
  tools:
  - ExoCross
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# line-list-format-conversion

## Summary

Convert molecular spectroscopy line lists between the ExoMol (ExoMol, ExoMolHR, ExoAtom databases) and HITRAN (HITRAN, HITEMP databases) data formats using PyExoCross, with wavenumber-range, uncertainty, and intensity-threshold filtering to control output size and quality.

## When to use

Use when an ExoMol-format line list (e.g. .states.bz2/.trans.bz2 files like 24Mg-1H__XAB) must be emitted as fixed-width HITRAN2004 160-character records, or vice versa, for cross-database interoperability.

## When NOT to use

- The line list is already in the target format and only direct import is needed — PyExoCross supports direct import/export of ExoMol and HITRAN/HITEMP formats without conversion
- The goal is computing spectra, cross-sections, or derived properties (partition functions, lifetimes, oscillator strengths) rather than reformatting the line list
- Required quantum number labels cannot be sourced from the dataset .def metadata and QNsFilter is enabled, so the conversion mapping would be incomplete

## Inputs

- ExoMol line list files (.states.bz2 and .trans.bz2, e.g. 24Mg-1H__XAB.states.bz2 / 24Mg-1H__XAB.trans.bz2) under a Molecule/Isotopologue/Dataset ReadPath
- ExoMol definition file (.def, .def., or .adef.) providing quantum number labels and formats
- PyExoCross input file (.inp) with Database, ConversionFormat, ConversionFrequncyRange, and conversion filter settings
- HITRAN/HITEMP line list files (for the reverse HITRAN-to-ExoMol conversion direction)

## Outputs

- Line list converted to the target format: fixed-width HITRAN2004 160-character records over the requested wavenumber range (ExoMol-to-HITRAN), or ExoMol-format states/transitions files (HITRAN-to-ExoMol)
- Log file recording the conversion run (optional, via nohup output redirection)

## How to apply

Locate the ExoMol input files under ReadPath using the Molecule/Isotopologue/Dataset layout (e.g. states file 24Mg-1H__XAB.states.bz2 and transitions file 24Mg-1H__XAB.trans.bz2). In the PyExoCross input file set Database=ExoMol, ConversionFormat=HITRAN, and ConversionFrequncyRange to the target wavenumber range (e.g. 0–30000 cm-1). Optionally enable conversion filters: ConvUncFilter(Y/N)=Y with a maximum uncertainty in cm-1 (e.g. 0.01) and ConvThreshold(Y/N)=Y with a minimum intensity in cm/molecule (e.g. 1e-30); the QNsFilter requires supplying quantum number labels and formats (e.g. global ElecState/v/Omega and local J/e-f) drawn from the dataset .def metadata. Run the conversion routine (convert_exomol_to_hitran / conversion) via `python run.py -p <input_filepath>` and verify the output records against HITRAN2004 fixed-width layout. The rationale is that quantum number metadata availability differs per database (ExoMol .def/.def./.adef. files, ExoMolHR qn metadata), so QN labels must come from the source dataset's own definition files.

## Related tools

- **PyExoCross** (Primary tool performing the ExoMol-to-HITRAN (and reverse) data format conversion, driven by an .inp input file with Database, ConversionFormat, ConversionFrequncyRange, ConvUncFilter, and ConvThreshold settings) — https://github.com/Beryl-Jingxin/PyExoCross.git
- **Python** (Runtime environment in which PyExoCross executes; PyExoCross is a Python adaptation of the ExoCross Fortran application and is runnable as `python run.py -p input_filepath`) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Fortran predecessor whose functionality PyExoCross adapts; conceptual reference for the conversion routine) — https://github.com/ExoMol/ExoCross

## Examples

```
python run.py -p ./input/24Mg-1H__XAB_exomol_to_hitran.inp
```

## Evaluation signals

- Output file exists at the expected path after the conversion run completes
- Each output record is a fixed-width HITRAN2004 160-character line
- All output lines fall within the requested ConversionFrequncyRange (e.g. 0–30000 cm-1)
- Every converted line satisfies the enabled ConvUncFilter maximum uncertainty (e.g. 0.01 cm-1) and ConvThreshold minimum intensity (e.g. 1e-30 cm/molecule)
- Quantum number columns in the output match the labels/formats supplied from the dataset .def metadata (e.g. global ElecState, v, Omega; local J, e/f)

## Limitations

- Conversion requires dataset-specific quantum number metadata: ExoMol .def/.def./.adef. files from exomol.com, or ExoMolHR qn metadata from exomol.com/exomolhr/qn/; HITRAN/HITEMP conversion additionally needs SpeciesID from HITRANOnline Isotopologue Metadata
- Support for databases beyond ExoMol, ExoMolHR, ExoAtom, HITRAN, and HITEMP is incomplete — VAMDC databases are marked as pending in the README
- If GPU mode is used for related computations, macOS MPS only uses float32 which lacks precision; Nvidia CUDA GPU or CPU mode is recommended for high-precision work

## Evidence

- [abstract] ExoMol↔HITRAN conversion capability: "PyExoCross supports importing and exporting line lists in the ExoMol and HITRAN/HITEMP formats. PyExoCross also provides conversion between the ExoMol and HITRAN data formats"
- [methods] Conversion scope across databases: "*PyExoCross* can convert data format between the ExoMol (ExoMol, ExoMolHR, and ExoAtom databases) and the HITRAN (HITRAN and HITEMP database) formats."
- [methods] ConvUncFilter semantics: "`ConvUncFilter(Y/N)`: If it is yes, the value is the maximum uncertainty you require, in unit of cm⁻¹."
- [methods] ConvThreshold semantics: "`ConvThreshold(Y/N)`: If it is yes, the value is the minimum intensity you require, in unit of cm/molecule."
- [methods] Quantum number metadata sources: [evidence span withheld — anchor status: not_found]
- [readme] Program invocation: "python run.py -p input_filepath"
- [readme] README conversion statement: [evidence span withheld — anchor status: external_doc]
