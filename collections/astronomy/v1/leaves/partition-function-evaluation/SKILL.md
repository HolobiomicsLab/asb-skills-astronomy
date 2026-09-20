---
name: partition-function-evaluation
description: Use when you hold a molecular line list (e.g. ExoMol 1H2-16O POKAZATEL
  .states/.trans files) and need temperature-dependent partition function values Q(T)
  as input to or validation of spectroscopic property calculations.
when_to_use_negative:
- The goal is a temperature-dependent property other than Q(T) itself — cooling functions,
  specific heats, lifetimes, oscillator strengths, stick spectra, or cross sections
  have their own dedicated flags and workflows in PyExoCross.
- The line list is not in an ExoMol or HITRAN/HITEMP format (e.g. VAMDC databases
  are still pending support in the README) — convert or use a different tool first.
- You need LTE/non-LTE spectra rather than thermodynamic quantities — use the stick
  spectra or cross-sections workflows instead.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: PyExoCross
  role: Executes the PartitionFunctions computation, evaluating Q(T) from the line
    list states over the configured temperature grid
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Fortran reference implementation that PyExoCross is a Python adaptation of
  repo: https://github.com/ExoMol/ExoCross
- name: Python
  role: Runtime environment for PyExoCross (tested on Python 3.8-3.12; install via
    pip install pyexocross or requirements.txt)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:d96cc1218c05d8223023805a177bd89808936cc6dad2c5e34a6f2e88040652d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoCross
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# partition-function-evaluation

## Summary

Compute molecular partition functions Q(T) from ExoMol/HITRAN-style line lists with PyExoCross by enabling its PartitionFunctions flag, evaluating Q(T) = sum_n g_n^tot * exp(-c2*E_n/T) over the line list's state energies on a fine temperature grid.

## When to use

Use when you hold a molecular line list (e.g. ExoMol 1H2-16O POKAZATEL .states/.trans files) and need temperature-dependent partition function values Q(T) as input to or validation of spectroscopic property calculations.

## When NOT to use

- The goal is a temperature-dependent property other than Q(T) itself — cooling functions, specific heats, lifetimes, oscillator strengths, stick spectra, or cross sections have their own dedicated flags and workflows in PyExoCross.
- The line list is not in an ExoMol or HITRAN/HITEMP format (e.g. VAMDC databases are still pending support in the README) — convert or use a different tool first.
- You need LTE/non-LTE spectra rather than thermodynamic quantities — use the stick spectra or cross-sections workflows instead.

## Inputs

- ExoMol line list for a molecule/isotopologue/dataset (e.g. H2O 1H2-16O POKAZATEL .states and .trans files)
- ExoMol .def definition file (provides the maximum temperature reference)
- PyExoCross input file (.inp) with ReadPath and SavePath configured

## Outputs

- Partition function table Q(T) with values from 1 K up to Tmax K in 1 K steps
- Log file recording the run (e.g. H2O_exomol.out when run with nohup)

## How to apply

Configure a PyExoCross input file with Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, setting ReadPath/SavePath to the line list locations. Enable only the PartitionFunctions flag (set to 1) and set all other function flags (Conversion, CoolingFunctions, SpecificHeats, Lifetimes, OscillatorStrengths, StickSpectra, CrossSections) to 0 so the run isolates Q(T). Set the temperature grid with Ntemp=1 K steps up to Tmax=10000 K, using the molecule's .def definition file, which provides the maximum temperature as reference. Run with Device=CPU, since CPU mode is recommended for these property computations and the README warns that macOS MPS GPU mode only uses float32 and lacks precision. PyExoCross evaluates Q(T) = sum_n g_n^tot * exp(-c2*E_n/T) from the state energies and degeneracies in the line list, so correctness depends on a complete, unfiltered .states file.

## Related tools

- **PyExoCross** (Executes the PartitionFunctions computation, evaluating Q(T) from the line list states over the configured temperature grid) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Fortran reference implementation that PyExoCross is a Python adaptation of) — https://github.com/ExoMol/ExoCross
- **Python** (Runtime environment for PyExoCross (tested on Python 3.8-3.12; install via pip install pyexocross or requirements.txt))

## Examples

```
python run.py -p /home/username/PyExoCross/input/H2O_exomol.inp
```

## Evaluation signals

- Output table covers the full requested grid, from 1 K up to Tmax (e.g. 10000 K) in 1 K steps, with a file-integrity check as the acceptance criterion (no quantitative threshold is reported in the paper).
- Q(T) increases monotonically with temperature, as implied by Q(T) = sum_n g_n^tot * exp(-c2*E_n/T).
- Only the PartitionFunctions flag is set to 1; all other function flags (Conversion, CoolingFunctions, SpecificHeats, Lifetimes, OscillatorStrengths, StickSpectra, CrossSections) are 0.
- Tmax does not exceed the molecule's maximum temperature given in the .def definition file.
- Repeated runs with Device=CPU produce reproducible values; avoid macOS MPS GPU float32 results where precision matters.

## Limitations

- The paper reports no quantitative validation threshold for the Q(T) table; acceptance is coverage plus file-integrity checking.
- GPU acceleration is optional and CPU mode is recommended for these property calculations; macOS MPS GPU mode uses only float32 and lacks precision.
- Q(T) is computed from the states present in the line list, so results depend on the completeness of the input .states file; dataset choice (e.g. POKAZATEL) determines the state set.

## Evidence

- [methods] PyExoCross also implements computations of other useful functions including partition functions: "*PyExoCross* also implements the computations of other useful functions including partition functions, specific heats, cooling functions"
- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions"
- [readme] generates absorption and emission spectra and other properties (partition functions, specific heats, cooling functions, lifetimes, oscillator strengths: [evidence span withheld — anchor status: external_doc]
- [readme] A Python framework for LTE and non-LTE spectroscopic post-processing of atomic and molecular line lists: "A Python framework for LTE and non-LTE spectroscopic post-processing of atomic and molecular line lists"
