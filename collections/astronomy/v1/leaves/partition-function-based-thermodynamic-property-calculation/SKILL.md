---
name: partition-function-based-thermodynamic-property-calculation
description: Use when you need temperature-dependent specific heats or partition functions
  for a molecule (e.g. 1H2-16O with the POKAZATEL line list) and have access to an
  ExoMol/HITRAN line-list folder with states and .def files.
when_to_use_negative:
- You need pressure-broadened spectra or cross-sections rather than thermodynamic
  properties — enable StickSpectra/CrossSections instead of SpecificHeats
- The molecule's .def file Tmax is below your required temperature range, since the
  temperature grid must stay consistent with the ExoMol definition file
- Input is atomic line-list data outside the supported ExoMol/ExoMolHR/ExoAtom/HITRAN/HITEMP
  formats
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3318
tools:
- name: PyExoCross
  role: Executes the specific-heats computation from molecular line lists via a .inp
    input file and 'python run.py -p input_filepath'
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoMol
  role: Source database supplying the line list, states files, and .def definition
    file (e.g. 1H2-16O POKAZATEL) from which specific heats are derived
  repo: https://www.exomol.com/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:e200200d885f7f861295020e6c0af0c577c03acbbf4b60aec83f4926a6cbbda6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoMol
  - PyExoCross
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# partition-function-based-thermodynamic-property-calculation

## Summary

Computes specific heats (and related properties such as partition functions and cooling functions) directly from molecular line lists using PyExoCross, which derives temperature-dependent thermodynamic quantities from the deposited states and partition-function files of databases like ExoMol.

## When to use

Use when you need temperature-dependent specific heats or partition functions for a molecule (e.g. 1H2-16O with the POKAZATEL line list) and have access to an ExoMol/HITRAN line-list folder with states and .def files.

## When NOT to use

- You need pressure-broadened spectra or cross-sections rather than thermodynamic properties — enable StickSpectra/CrossSections instead of SpecificHeats
- The molecule's .def file Tmax is below your required temperature range, since the temperature grid must stay consistent with the ExoMol definition file
- Input is atomic line-list data outside the supported ExoMol/ExoMolHR/ExoAtom/HITRAN/HITEMP formats

## Inputs

- PyExoCross .inp input file with Database, Molecule, Isotopologue, Dataset, SpeciesID fields and SpecificHeats=1
- ExoMol line-list folder containing .states and .def definition files (ReadPath)
- Temperature grid settings (Ntemp step, Tmax) consistent with the .def file

## Outputs

- Saved specific-heats table spanning 1 K to Tmax in Ntemp (e.g. 1 K) steps
- Optional partition-function table if PartitionFunctions is also enabled

## How to apply

Prepare a PyExoCross .inp input file specifying the ExoMol data source (Database, Molecule, Isotopologue, Dataset, SpeciesID, e.g. H2O / 1H2-16O / POKAZATEL) with ReadPath pointing to the ExoMol line-list folder and SavePath/LogFilePath set for outputs. Enable the specific-heats function by setting SpecificHeats to 1 and all other function flags (Conversion, PartitionFunctions, CoolingFunctions, Lifetimes, OscillatorStrengths, StickSpectra, CrossSections) to 0. In the '# Calculate partition, specific heats or cooling functions #' section, set the temperature step Ntemp to 1 K and choose Tmax (e.g. 10000 K) consistent with the maximum temperature given in the ExoMol .def definition file, so temperatures run from 1 K to Tmax. Execute the program with 'python run.py -p input_filepath' in the default CPU mode; the computation reads the deposited states and partition-function files. The rationale is that specific heats are derived from the same line-list states that underlie the partition function, ensuring consistency across the thermodynamic properties.

## Related tools

- **PyExoCross** (Executes the specific-heats computation from molecular line lists via a .inp input file and 'python run.py -p input_filepath') — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoMol** (Source database supplying the line list, states files, and .def definition file (e.g. 1H2-16O POKAZATEL) from which specific heats are derived) — https://www.exomol.com/

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The saved specific-heats table spans 1 K to Tmax in 1 K steps as configured via Ntemp
- Specific-heat values at a sample of temperatures match values independently derivable from the deposited partition function and states files
- Log file confirms SpecificHeats=1 while all other function flags (Conversion, PartitionFunctions, CoolingFunctions, Lifetimes, OscillatorStrengths, StickSpectra, CrossSections) were 0
- Chosen Tmax does not exceed the maximum temperature given in the ExoMol .def definition file

## Limitations

- Tmax must be consistent with the maximum temperature in the ExoMol .def definition file; extrapolation beyond it is not supported by this workflow
- GPU mode with macOS MPS only uses float32 which lacks precision, so CPU mode is recommended for high-precision calculations
- Requires a complete ExoMol line-list download (states and .def files) at ReadPath; no batch download is part of this specific-heat function

## Evidence

- [methods] PyExoCross also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths: "*PyExoCross* also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths"
- [readme] PyExoCross generates absorption and emission spectra and other properties (partition functions, specific heats, cooling functions, lifetimes, oscillator strengths, and stick spectra) based on molecular line lists: [evidence span withheld — anchor status: not_found]
- [methods] The definition files .def, .def., and .adef. of ExoMol and ExoAtom databases are needed to configure runs: [evidence span withheld — anchor status: not_found]
- [readme] In the terminal, use the following commands to run PyExoCross: [evidence span withheld — anchor status: external_doc]
- [readme] macOS MPS GPU can use GPU mode but it only uses float32 which lacks precision, so please use Nvidia GPU (CUDA) or use macOS CPU mode instead for high precision: [evidence span withheld — anchor status: external_doc]
