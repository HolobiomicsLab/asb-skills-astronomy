---
name: specific-heat-computation-from-molecular-line-lists
description: Use when you need temperature-dependent specific heats (heat capacities)
  for a molecule/isotopologue and already have access to an ExoMol (or HITRAN/HITEMP)
  line-list folder with its .def definition, states, and partition-function files.
when_to_use_negative:
- You need spectra rather than thermodynamics — stick spectra/cross-sections are separate
  function flags (StickSpectra, CrossSections) and should not be mixed with the SpecificHeats
  run.
- You only need a partition function or cooling function — enable PartitionFunctions
  or CoolingFunctions instead of SpecificHeats.
- You only need ExoMol-to-HITRAN format conversion — use the Conversion function rather
  than the thermodynamic-property path.
edam_operation: http://edamontology.org/operation_3663
tools:
- name: PyExoCross
  role: Executes the specific-heats computation from deposited states and partition-function
    files via a .inp input file run with 'python run.py -p input_filepath'
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment for PyExoCross (Python 3.8-3.12; packages in requirements.txt
    such as numpy, pandas, scipy)
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:e2215997f2db0822bec0cad74928c00eb841cd66bb3510e0a015c95740d0c140
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_3663
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# specific-heat-computation-from-molecular-line-lists

## Summary

Computes specific heats directly from molecular (or atomic) line lists using PyExoCross's thermodynamic-property mode, driven by an ExoMol-style .inp input file that reads the deposited states and partition-function files. This gives temperature-dependent heat capacities consistent with the source database without hand-deriving thermodynamics.

## When to use

Use when you need temperature-dependent specific heats (heat capacities) for a molecule/isotopologue and already have access to an ExoMol (or HITRAN/HITEMP) line-list folder with its .def definition, states, and partition-function files.

## When NOT to use

- You need spectra rather than thermodynamics — stick spectra/cross-sections are separate function flags (StickSpectra, CrossSections) and should not be mixed with the SpecificHeats run.
- You only need a partition function or cooling function — enable PartitionFunctions or CoolingFunctions instead of SpecificHeats.
- You only need ExoMol-to-HITRAN format conversion — use the Conversion function rather than the thermodynamic-property path.

## Inputs

- PyExoCross .inp input file (Database, Molecule, Isotopologue, Dataset, SpeciesID, ReadPath, SavePath, LogFilePath, SpecificHeats=1, Ntemp, Tmax)
- ExoMol line-list folder containing .def definition file, states file, and partition-function file (e.g. 1H2-16O / POKAZATEL)
- Optionally HITRAN/HITEMP line-list data with a HITRAN SpeciesID

## Outputs

- Temperature-dependent specific-heats table spanning 1 K to Tmax in Ntemp (e.g. 1 K) steps, written to the SavePath directory
- Program log file at LogFilePath

## How to apply

Prepare a PyExoCross .inp file specifying the data source (Database, Molecule, Isotopologue, Dataset, SpeciesID, e.g. H2O / 1H2-16O / POKAZATEL) with ReadPath pointing at the line-list folder and SavePath/LogFilePath set. Enable the specific-heats function by setting SpecificHeats=1 and all other function flags (Conversion, PartitionFunctions, CoolingFunctions, Lifetimes, OscillatorStrengths, StickSpectra, CrossSections) to 0. In the '# Calculate partition, specific heats or cooling functions #' section set Ntemp to a 1 K temperature step and choose Tmax (e.g. 10000 K) consistent with the maximum temperature in the ExoMol .def file, so temperatures run from 1 K to Tmax. Run 'python run.py -p input_filepath' in default CPU mode; PyExoCross computes specific heats from the deposited states and partition-function files. PyExoCross can be installed as a package via 'pip install pyexocross' or cloned from GitHub.

## Related tools

- **PyExoCross** (Executes the specific-heats computation from deposited states and partition-function files via a .inp input file run with 'python run.py -p input_filepath') — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment for PyExoCross (Python 3.8-3.12; packages in requirements.txt such as numpy, pandas, scipy)) — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The saved specific-heats table spans 1 K to Tmax in exact 1 K (Ntemp) steps with no missing temperatures.
- Values at a sample of temperatures match specific heats independently derivable from the deposited partition function and states files.
- Tmax choice is consistent with the maximum temperature given in the ExoMol .def definition file.
- Log file at LogFilePath confirms SpecificHeats was the only enabled function (all other flags = 0).
- Only one function flag (SpecificHeats=1) is active, so outputs contain no unintended spectra/cross-section files.

## Limitations

- Tmax must be consistent with the maximum temperature supported by the source database's .def file; exceeding it is out of scope.
- GPU mode uses float32 on macOS MPS which lacks precision, so CPU mode (default) or an NVIDIA CUDA GPU is preferred for high-precision calculations.
- The computation depends on the deposited partition-function and states files being present and complete in ReadPath.

## Evidence

- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths): "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths)"
- [methods] PyExoCross also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes: "PyExoCross also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes"
- [readme] PyExoCross generates absorption and emission spectra and other properties (partition functions, specific heats, cooling functions, lifetimes, oscillator strengths, and stick spectra): [evidence span withheld — anchor status: not_found]
- [methods] The definition files .def, .def., and .adef. of ExoMol and ExoAtom databases are available at exomol.com: [evidence span withheld — anchor status: not_found]
