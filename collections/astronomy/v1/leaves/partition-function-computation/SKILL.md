---
name: partition-function-computation
description: Use when a workflow requires internal partition functions Q(T) for a
  molecule/isotopologue over a temperature grid (e.g. 1 K steps up to the dataset's
  Tmax) and a states file with energy levels and degeneracies is available.
when_to_use_negative:
- The goal is to reuse an existing reference partition function — the ExoMol dataset
  already distributes a reference .pf file that should be used directly for validation
  instead of recomputation
- The task is cross-section or stick-spectrum generation — PyExoCross supports those
  via separate function flags, not PartitionFunctions
- Only spectral line profiles (Doppler, Gaussian, Lorentzian, Voigt) are needed —
  that is the cross-section pipeline, not partition function computation
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3318
tools:
- name: PyExoCross
  role: Reads the states file and computes the Boltzmann-weighted partition function
    sum, writing the .pf output table
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment for PyExoCross (tested on Python 3.8–3.12)
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:6cd5cc7f7d5b4dd55e57d0bcede28b3c39e786a9003c0c048da26cf10091d781
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# partition-function-computation

## Summary

Compute temperature-dependent molecular partition functions Q(T) from ExoMol (or HITRAN/HITEMP) line lists using PyExoCross, as one of its post-processing functions alongside specific heats, cooling functions, lifetimes, and oscillator strengths.

## When to use

Use when a workflow requires internal partition functions Q(T) for a molecule/isotopologue over a temperature grid (e.g. 1 K steps up to the dataset's Tmax) and a states file with energy levels and degeneracies is available.

## When NOT to use

- The goal is to reuse an existing reference partition function — the ExoMol dataset already distributes a reference .pf file that should be used directly for validation instead of recomputation
- The task is cross-section or stick-spectrum generation — PyExoCross supports those via separate function flags, not PartitionFunctions
- Only spectral line profiles (Doppler, Gaussian, Lorentzian, Voigt) are needed — that is the cross-section pipeline, not partition function computation

## Inputs

- ExoMol dataset definition metadata (.def / .def. / .adef.)
- States file <Isotopologue>__<Dataset>.states.bz2 containing energy levels and total degeneracies
- PyExoCross input file (.inp) with Database, molecule/isotopologue/dataset identifiers, ReadPath/SavePath, and function flags

## Outputs

- Partition function table <Isotopologue>__<Dataset>.pf with temperature-versus-Q(T) values from 1 K to Tmax in 1 K steps

## How to apply

Read the ExoMol dataset definition metadata (.def/.def.) to confirm the molecule, isotopologue, dataset names and the maximum recommended temperature. Configure PyExoCross with Database=ExoMol, the molecule/isotopologue/dataset identifiers, ReadPath/SavePath, and set the Functions flag PartitionFunctions=1 with all other functions set to 0. Set Ntemp to 1 K temperature steps and Tmax to the dataset's recommended maximum (e.g. 10000 K), then compute Q(T) as the Boltzmann-weighted sum over all levels, Q(T) = sum_n g_n * exp(-c2 * E_n / T), using the division convention used by PyExoCross. Save the temperature-versus-Q(T) table as <Isotopologue>__<Dataset>.pf in the SavePath folder.

## Related tools

- **PyExoCross** (Reads the states file and computes the Boltzmann-weighted partition function sum, writing the .pf output table) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment for PyExoCross (tested on Python 3.8–3.12)) — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp  # with PartitionFunctions=1, Ntemp=1 K steps, Tmax=10000 K in the input file
```

## Evaluation signals

- The .pf output file exists in the SavePath folder and is named <Isotopologue>__<Dataset>.pf
- The temperature column starts at 1 K and ends at Tmax K with exactly 1 K steps
- Computed Q(T) values are in close agreement with the reference .pf file distributed with the ExoMol dataset
- Q(T) is monotonically increasing with T and physically plausible in magnitude (expert review)
- Q(T) at the lowest temperatures is close to the degeneracy of the ground state (consistency check on the Boltzmann sum)

## Limitations

- Q(T) is only valid up to the dataset's recommended maximum temperature (Tmax, e.g. 10000 K); extrapolation beyond it is unsupported
- Accuracy depends on the completeness of the states file energy levels; missing high-energy levels underestimate Q(T) at high T
- On macOS, MPS GPU mode uses float32 which lacks precision; use CPU mode (or Nvidia CUDA) for high-precision calculations

## Evidence

- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on molecular line lists.: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator"
- [methods] Partition functions are computed among other post-processing functions: "*PyExoCross* also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths"
- [readme] Partition functions listed among README-described output properties: [evidence span withheld — anchor status: external_doc]
- [intro] PyExoCross is a Python adaptation of ExoCross: "PyExoCross is a Python adaptation of the ExoCross Fortran application"
