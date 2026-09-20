---
name: einstein-a-coefficient-summation
description: Use when you have an ExoMol (or similar) molecular line list with states
  and transitions files containing Einstein A coefficients and need per-level radiative
  lifetimes rather than spectra.
when_to_use_negative:
- You need spectra or cross-sections rather than level-resolved lifetimes — use StickSpectra
  or CrossSections functions instead (set Lifetimes=0).
- The line list transitions files lack Einstein A coefficients (or only oscillator
  strengths are available) — the lifetime sum cannot be formed.
- You need predissociation lifetimes for Voigt-profile cross sections — that is the
  separate PredissocXsec option, not the lifetimes function.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3314
tools:
- name: PyExoCross
  role: Runs the lifetimes post-processing module that sums Einstein A coefficients
    per upper state; invoked with run.py and an input file with Lifetimes=1
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment for PyExoCross (tested on Python 3.8–3.12; install via
    pip install pyexocross or requirements.txt)
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoMol
  role: Source database providing the MgH 24Mg-1H Yadin def/states/transitions line
    list files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:2ef1a7df444db740027120a5d986d41cbeba9cc485a614c342caf9ca862b5d9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoMol
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Einstein A coefficient summation

## Summary

Computes per-level radiative lifetimes by summing Einstein A coefficients of all transitions terminating on each upper state, as implemented in PyExoCross's lifetimes post-processing function for ExoMol line lists such as 24Mg-1H Yadin.

## When to use

Use when you have an ExoMol (or similar) molecular line list with states and transitions files containing Einstein A coefficients and need per-level radiative lifetimes rather than spectra.

## When NOT to use

- You need spectra or cross-sections rather than level-resolved lifetimes — use StickSpectra or CrossSections functions instead (set Lifetimes=0).
- The line list transitions files lack Einstein A coefficients (or only oscillator strengths are available) — the lifetime sum cannot be formed.
- You need predissociation lifetimes for Voigt-profile cross sections — that is the separate PredissocXsec option, not the lifetimes function.

## Inputs

- ExoMol line list definition file (.def) for the isotopologue/dataset (e.g. 24Mg-1H Yadin)
- ExoMol states file (.states) listing energy levels
- ExoMol transitions files (.trans) containing Einstein A coefficients
- PyExoCross input file (.inp) configured with Database/Molecule/Isotopologue/Dataset and paths

## Outputs

- Lifetimes table saved as 24Mg-1H_Yadin_lifetimes.csv in SavePath, with one radiative lifetime per energy level

## How to apply

Obtain the target line list files (.def, .states, .trans) from exomol.com and arrange them in the standard layout ReadPath/MgH/24Mg-1H/Yadin/. Configure a PyExoCross input file with Database=ExoMol, Molecule=MgH, Isotopologue=24Mg-1H, Dataset=Yadin, set ReadPath, SavePath, and LogFilePath, and enable only the lifetimes function (Lifetimes=1) while setting Conversion, PartitionFunctions, SpecificHeats, CoolingFunctions, OscillatorStrengths, StickSpectra, and CrossSections to 0. Run via `python run.py -p input_filepath` with Device=CPU (default) and tune the parallel parameters NCPUtrans, NCPUfiles, and ChunkSize for the transitions-file reading. Rationale: PyExoCross sums the Einstein A coefficients from the transitions files for each upper state, and the lifetime is the reciprocal of that sum, so completeness of the transition set for each level directly determines correctness. Output is written to SavePath as 24Mg-1H_Yadin_lifetimes.csv with one lifetime entry per energy level.

## Related tools

- **PyExoCross** (Runs the lifetimes post-processing module that sums Einstein A coefficients per upper state; invoked with run.py and an input file with Lifetimes=1) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment for PyExoCross (tested on Python 3.8–3.12; install via pip install pyexocross or requirements.txt)) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoMol** (Source database providing the MgH 24Mg-1H Yadin def/states/transitions line list files)

## Examples

```
python run.py -p ./input/MgH_exomol_lifetimes.inp  # input has Database=ExoMol, Molecule=MgH, Isotopologue=24Mg-1H, Dataset=Yadin, Lifetimes=1
```

## Evaluation signals

- Output file 24Mg-1H_Yadin_lifetimes.csv exists in SavePath after the run.
- The CSV contains exactly one lifetime entry per energy level of the line list's states file.
- Each lifetime equals the reciprocal of the summed Einstein A coefficients of transitions terminating on that level.
- All lifetime values are positive and finite.
- The log at LogFilePath reports that only the lifetimes module ran (all other functions set to 0).

## Limitations

- Lifetime accuracy depends on the transitions files capturing every decay channel of each upper state; missing A coefficients inflate the lifetime.
- GPU (CUDA) mode is optional and falls back to CPU when unavailable; macOS MPS GPU uses float32 which lacks precision, so CPU mode is recommended for high-precision calculations.
- Parallel performance depends on correctly setting NCPUtrans, NCPUfiles, and ChunkSize for the transitions-file reading workload.

## Evidence

- [methods] Radiative lifetimes computation is a PyExoCross post-processing function alongside partition functions, specific heats, cooling functions, and oscillator strengths.: "*PyExoCross* also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE"
- [abstract] Lifetimes are among the properties PyExoCross generates from molecular line lists.: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on"
- [readme] The program is run from an input file path, e.g. for an ExoMol input.: "python run.py -p ./input/H2O_exomol.inp"
- [readme] Device defaults to CPU; GPU mode falls back to CPU when CUDA is unavailable.: "GPU mode is optional and automatically falls back to CPU if CUDA runtime
packages or devices are unavailable."
