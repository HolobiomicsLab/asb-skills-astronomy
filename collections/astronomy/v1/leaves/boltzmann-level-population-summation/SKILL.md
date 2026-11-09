---
name: boltzmann-level-population-summation
description: Use when you have an ExoMol (or similar) molecular line list with a states
  file of energy levels and total degeneracies, and need a temperature-resolved partition
  function table Q(T).
when_to_use_negative:
- The task is computing cross-sections, stick spectra, cooling functions, or lifetimes
  — those require different Functions flags in the PyExoCross input file
- The line list has no states file with energy levels and degeneracies, so the direct
  Boltzmann sum over levels cannot be formed
- Temperatures above the dataset's recommended maximum T are required — the .pf values
  would be extrapolated beyond the line list's level completeness
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3314
tools:
- name: PyExoCross
  role: Parses ExoMol states files and runs the partition_functions routine (PartitionFunctions=1)
    to compute Q(T) over a 1 K temperature grid up to Tmax
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment for PyExoCross (tested on Python 3.8–3.12) used to execute
    run.py with the partition-function input file
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Original Fortran application that PyExoCross (the partition-function implementation)
    is a Python adaptation of
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:d59deb56514e7088f95e3272cd16f3e7453f5058327c14e6c323f4f57ef6d7f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoCross
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# boltzmann-level-population-summation

## Summary

Computes molecular partition functions Q(T) as the Boltzmann-weighted sum over all energy levels, Q(T) = sum_n g_n * exp(-c2 * E_n / T), using PyExoCross's PartitionFunctions routine on ExoMol states files.

## When to use

Use when you have an ExoMol (or similar) molecular line list with a states file of energy levels and total degeneracies, and need a temperature-resolved partition function table Q(T).

## When NOT to use

- The task is computing cross-sections, stick spectra, cooling functions, or lifetimes — those require different Functions flags in the PyExoCross input file
- The line list has no states file with energy levels and degeneracies, so the direct Boltzmann sum over levels cannot be formed
- Temperatures above the dataset's recommended maximum T are required — the .pf values would be extrapolated beyond the line list's level completeness

## Inputs

- ExoMol dataset definition file (.def / .def.) specifying molecule, isotopologue, and dataset
- States file (Isotopologue__Dataset.states.bz2) with energy levels and total degeneracies
- Reference partition function file (.pf) distributed with the ExoMol dataset (for validation)
- PyExoCross input file with Database=ExoMol, ReadPath/SavePath, PartitionFunctions=1, Ntemp=1 K, Tmax (e.g. 10000 K)

## Outputs

- <Isotopologue>__<Dataset>.pf file: temperature-versus-Q(T) table from 1 K to Tmax in 1 K steps

## How to apply

Read the ExoMol dataset definition metadata (.def) to confirm molecule, isotopologue, dataset names and the maximum recommended temperature (e.g. 10000 K). Use PyExoCross to parse the states file (Isotopologue__Dataset.states.bz2) to obtain energy levels E_n and total degeneracies g_n. Configure the input with Database=ExoMol, molecule/isotopologue/dataset identifiers, ReadPath/SavePath, Functions flag PartitionFunctions=1 with all other functions set to 0, Ntemp=1 K temperature steps, and Tmax equal to the dataset's recommended maximum; run partition_functions on CPU. At each temperature compute Q(T) as the Boltzmann-weighted sum over all levels, Q(T) = sum_n g_n * exp(-c2 * E_n / T), following PyExoCross's division-by-total-partition-function weight convention, and save the temperature-versus-Q(T) table as <Isotopologue>__<Dataset>.pf in the SavePath folder.

## Related tools

- **PyExoCross** (Parses ExoMol states files and runs the partition_functions routine (PartitionFunctions=1) to compute Q(T) over a 1 K temperature grid up to Tmax) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment for PyExoCross (tested on Python 3.8–3.12) used to execute run.py with the partition-function input file) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Original Fortran application that PyExoCross (the partition-function implementation) is a Python adaptation of)

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The output .pf file exists in SavePath and is named <Isotopologue>__<Dataset>.pf
- The .pf table starts at 1 K, ends at Tmax K, and contains 1 K temperature steps throughout
- Computed Q(T) values are in close agreement with the reference .pf file distributed with the ExoMol dataset
- Q(T) magnitudes are physically plausible upon expert review (monotonically increasing with T, consistent with level count and degeneracies)
- All other Functions flags are 0 so only partition functions were computed in the run

## Limitations

- Q(T) is only trustworthy up to the dataset's recommended maximum temperature; above Tmax the level list is incomplete
- GPU mode (CUDA via PyTorch/CuPy) is aimed at cross-section calculations; macOS MPS GPU uses float32 which lacks precision, so CPU mode should be used for high-precision calculations
- PyExoCross uses a specific division-by-total-partition-function weight convention that must be matched when comparing against other Q(T) implementations

## Evidence

- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on molecular line lists.: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on"
- [methods] PyExoCross also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE: "*PyExoCross* also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE"
- [other] Parse the states file to obtain energy levels and total degeneracies using PyExoCross; compute Q(T) = sum_n g_n * exp(-c2 * E_n / T) and save <Isotopologue>__<Dataset>.pf: [evidence span withheld — anchor status: not_found]
- [readme] macOS MPS GPU can use GPU mode but it only uses float32 which lacks precision, so please use Nvidia GPU (CUDA) or use macOS CPU mode instead for high precision calculations: [evidence span withheld — anchor status: external_doc]
