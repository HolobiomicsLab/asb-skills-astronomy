---
name: exomol-database-file-handling
description: 'Use when preparing ExoMol line-list files (e.g. 1H2-16O__POKAZATEL.states.bz2
  and .trans.bz2) for PyExoCross: identifying the molecule/isotopologue/dataset/SpeciesID
  and arranging ReadPath/SavePath before running any function.'
when_to_use_negative:
- Input line lists are in HITRAN/HITEMP format rather than the ExoMol layout — those
  require HITRAN SpeciesID metadata and different database identifiers
- The goal is format conversion between ExoMol and HITRAN rather than staging files
  for a computational function
- Atomic/ion datasets are involved — the ExoAtom database requires Database, Atom,
  and Dataset names instead of molecule/isotopologue fields
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3382
tools:
- name: PyExoCross
  role: Program that parses the .inp Data source block, locates the ExoMol states/transitions
    files under ReadPath, and runs the requested function
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Fortran application of which PyExoCross is a Python adaptation, sharing the
    ExoMol input conventions
- name: Python
  role: Language in which PyExoCross is implemented and executed via run.py
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:97205bd3d029fa76f80b22fe08d2801dd813924b262b0768fb2e29911f74ec3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoCross
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# exomol-database-file-handling

## Summary

Configure and stage ExoMol database inputs (states, transitions, and .def definition files) in the standard directory layout so PyExoCross can locate and post-process a line list. This is the prerequisite data-handling step for all PyExoCross functions such as cross-sections, partition functions, and specific heats.

## When to use

Use when preparing ExoMol line-list files (e.g. 1H2-16O__POKAZATEL.states.bz2 and .trans.bz2) for PyExoCross: identifying the molecule/isotopologue/dataset/SpeciesID and arranging ReadPath/SavePath before running any function.

## When NOT to use

- Input line lists are in HITRAN/HITEMP format rather than the ExoMol layout — those require HITRAN SpeciesID metadata and different database identifiers
- The goal is format conversion between ExoMol and HITRAN rather than staging files for a computational function
- Atomic/ion datasets are involved — the ExoAtom database requires Database, Atom, and Dataset names instead of molecule/isotopologue fields

## Inputs

- ExoMol states file, e.g. 1H2-16O__POKAZATEL.states.bz2
- ExoMol transitions file, e.g. 1H2-16O__POKAZATEL.trans.bz2
- ExoMol definition files (.def, .def., .adef.) from exomol.com
- PyExoCross .inp input file with Data source block (Database, Molecule, Isotopologue, Dataset, SpeciesID) and path settings
- Quantum number metadata for ExoMolHR (from exomol.com/exomolhr/qn/) when applicable

## Outputs

- Correctly staged ExoMol directory readable by PyExoCross at ReadPath
- Validated .inp configuration ready to invoke cross_sections, partition_functions, or specific_heats routines
- Saved outputs in SavePath (e.g. .xsec cross-section file or <Isotopologue>__<Dataset>.pf partition function table)

## How to apply

Download the ExoMol dataset files and place the .states.bz2 and .trans.bz2 files under ReadPath in the standard ExoMol directory layout, then check the dataset .def/.def. definition file to confirm the molecule, isotopologue, dataset names and the maximum recommended temperature. In the PyExoCross .inp input file's Data source block, set Database=ExoMol, Molecule, Isotopologue (e.g. 1H2-16O), Dataset (e.g. POKAZATEL), and SpeciesID (e.g. 11 for 1H2-16O); SpeciesID is mandatory and can be looked up in HITRAN isotopologue metadata when working with HITRAN/HITEMP databases. Set ReadPath, SavePath, and LogFilePath for outputs, then enable exactly one Functions flag (e.g. CrossSections=1 or SpecificHeats=1 with all others 0) so the correct routine reads the staged files. The definition-file metadata (e.g. Tmax from the .def file) drives downstream parameter choices such as Tmax in partition-function and specific-heat computations.

## Related tools

- **PyExoCross** (Program that parses the .inp Data source block, locates the ExoMol states/transitions files under ReadPath, and runs the requested function) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Fortran application of which PyExoCross is a Python adaptation, sharing the ExoMol input conventions)
- **Python** (Language in which PyExoCross is implemented and executed via run.py)

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The Data source block in the .inp file matches the deposited ExoMol dataset (Database=ExoMol, correct Molecule/Isotopologue/Dataset/SpeciesID, e.g. 1H2-16O / POKAZATEL / 11)
- The states and transitions .bz2 files exist under ReadPath in the standard ExoMol directory layout
- Exactly one Functions flag is set to 1 (e.g. CrossSections=1 or SpecificHeats=1) with all others 0
- Tmax chosen for temperature-dependent computations is consistent with the maximum temperature in the ExoMol .def definition file
- The expected output file appears in SavePath (e.g. .xsec or <Isotopologue>__<Dataset>.pf) after running python run.py -p input_filepath

## Limitations

- SpeciesID must be supplied correctly and is database-specific; it is obtained from HITRANOnline isotopologue metadata for HITRAN/HITEMP datasets
- ExoMolHR requires separate quantum number metadata fetched from exomol.com/exomolhr/qn/
- Quantum-number handling and other options depend on the definition files (.def, .def., .adef.) being available from exomol.com
- No changelog was found in the discussion sources, so version-specific behavior changes are not traceable from this context

## Evidence

- [other] Data source block specifies Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, SpeciesID=11: [evidence span withheld — anchor status: not_found]
- [methods] Definition files available from exomol.com are required: [evidence span withheld — anchor status: not_found]
- [other] ExoMol states and transitions files must be available in the standard directory layout: [evidence span withheld — anchor status: not_found]
- [readme] README describes PyExoCross as post-processing ExoMol line lists: "designed for postprocessing the huge molecular line lists generated by the ExoMol project and other similar initiatives such as the HITRAN and HITEMP"
- [methods] SpeciesID can be found from HITRAN isotopologue metadata: "`SpeciesID` can be found from [HITRANOnline Isotopologue Metadata](https://hitran.org/docs/iso-meta/)"
