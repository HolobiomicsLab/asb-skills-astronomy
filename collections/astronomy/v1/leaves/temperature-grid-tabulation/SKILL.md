---
name: temperature-grid-tabulation
description: Use when you have an ExoMol (or ExoAtom/HITRAN/HITEMP) states file with
  energy levels and degeneracies and need Q(T) values on a fine temperature grid,
  e.g. as input to spectra or thermodynamic property computations.
when_to_use_negative:
- You only need spectra or cross-sections and not partition functions or other derived
  properties — configure stick spectra/cross-section function flags instead
- The temperature grid resolution you need is coarser than 1 K steps or exceeds the
  dataset's recommended maximum temperature, where higher energy levels may be incomplete
- You have no states file (energy levels/degeneracies) — Q(T) cannot be summed without
  it
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3318
tools:
- name: PyExoCross
  role: Parses the ExoMol states file and computes the Boltzmann-weighted partition
    function sum on the temperature grid, saving the .pf file
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment in which PyExoCross (a Python adaptation of the ExoCross
    Fortran application) is executed via run.py or the pip-installed package
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:18538c8bcdeae3a656b553e61108cf31c520dcda9ee94fbbf9e55b534a5e53a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# temperature-grid-tabulation

## Summary

Tabulates molecular partition functions Q(T) on a regular temperature grid from an ExoMol line list using PyExoCross, as a post-processing step that Boltzmann-sums energy levels and degeneracies from the states file.

## When to use

Use when you have an ExoMol (or ExoAtom/HITRAN/HITEMP) states file with energy levels and degeneracies and need Q(T) values on a fine temperature grid, e.g. as input to spectra or thermodynamic property computations.

## When NOT to use

- You only need spectra or cross-sections and not partition functions or other derived properties — configure stick spectra/cross-section function flags instead
- The temperature grid resolution you need is coarser than 1 K steps or exceeds the dataset's recommended maximum temperature, where higher energy levels may be incomplete
- You have no states file (energy levels/degeneracies) — Q(T) cannot be summed without it

## Inputs

- ExoMol dataset definition file (.def / .def.) specifying molecule, isotopologue, dataset, and maximum recommended temperature
- States file (<Isotopologue>__<Dataset>.states.bz2) with energy levels and total degeneracies
- PyExoCross input file (.inp) with Database=ExoMol, identifiers, ReadPath/SavePath, PartitionFunctions=1, Ntemp=1 K, Tmax (e.g. 10000 K)

## Outputs

- Partition function table <Isotopologue>__<Dataset>.pf: temperature grid from 1 K to Tmax K in 1 K steps versus Q(T)

## How to apply

Read the ExoMol dataset definition metadata (.def/.def.) to confirm molecule, isotopologue, dataset name and the dataset's recommended maximum temperature (e.g. 10000 K). Parse the states file (<Isotopologue>__<Dataset>.states.bz2) with PyExoCross to obtain energy levels E_n and total degeneracies g_n. In the PyExoCross input file set Database=ExoMol with the molecule/isotopologue/dataset identifiers and ReadPath/SavePath, set Functions flag PartitionFunctions=1 with all other function flags 0, Ntemp to 1 K steps and Tmax to the dataset's recommended maximum. PyExoCross computes Q(T) = sum_n g_n * exp(-c2 * E_n / T) at each temperature on CPU (or GPU mode, though CPU is preferred for high precision since macOS MPS GPU is float32 only). Save the temperature-versus-Q(T) table as <Isotopologue>__<Dataset>.pf in the SavePath folder and cross-check magnitudes against the reference .pf file distributed with the ExoMol dataset.

## Related tools

- **PyExoCross** (Parses the ExoMol states file and computes the Boltzmann-weighted partition function sum on the temperature grid, saving the .pf file) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment in which PyExoCross (a Python adaptation of the ExoCross Fortran application) is executed via run.py or the pip-installed package) — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The .pf output file exists in SavePath and is named <Isotopologue>__<Dataset>.pf
- The temperature column starts at 1 K, ends at Tmax K (e.g. 10000 K), with exactly 1 K steps (Ntemp=1)
- Computed Q(T) values closely match the reference .pf file distributed with the ExoMol dataset at overlapping temperatures
- Q(T) is monotonically non-decreasing in T (a Boltzmann sum over positive levels cannot decrease with temperature)
- Q(T) magnitudes judged physically plausible by expert review (e.g. Q(1 K) ≈ minimum total degeneracy of the molecule)

## Limitations

- Accuracy is limited above the dataset's recommended maximum temperature (e.g. Tmax=10000 K) because the line list/states file may not contain enough high-lying levels
- macOS MPS GPU mode only uses float32 and lacks precision; use Nvidia GPU (CUDA) or CPU mode for high-precision partition function calculations
- Distributed as GPL-3.0 code pinned to numpy <2.0 and specific package version ranges (Python 3.8–3.12)

## Evidence

- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on molecular line lists.: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths)"
- [methods] PyExoCross also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE.: "*PyExoCross* also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths"
- [methods] The definition files `.def`, `.def.`, and `.adef.` of ExoMol and ExoAtom databases (available at exomol.com) are read by PyExoCross.: [evidence span withheld — anchor status: not_found]
- [readme] PyExoCross is designed for postprocessing the huge molecular line lists generated by the ExoMol project and other similar initiatives such as the HITRAN and HITEMP.: "designed for postprocessing the huge molecular line lists generated by the ExoMol project and other similar initiatives such as the HITRAN and HITEMP"
- [readme] macOS MPS GPU can use GPU mode but it only uses float32 which lacks precision, so please use Nvidia GPU (CUDA) or use macOS CPU mode instead for high precision calculations.: [evidence span withheld — anchor status: external_doc]
