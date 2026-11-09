---
name: radiative-lifetime-calculation
description: Use when you have an ExoMol line list (states and transitions files with
  Einstein A coefficients) and need radiative lifetimes per energy level as a post-processing
  product rather than spectra or cross-sections.
when_to_use_negative:
- You need spectra, cross-sections, or cooling functions — enable StickSpectra/CrossSections/CoolingFunctions
  instead; lifetimes mode computes only per-level lifetimes
- The dataset's transitions files lack Einstein A coefficients, since lifetimes are
  the reciprocal of summed A coefficients
- You are working with a HITRAN/HITEMP line list without the transition probability
  data PyExoCross's lifetimes module expects from ExoMol-format transitions files
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3314
tools:
- name: PyExoCross
  role: Executes the lifetimes post-processing function, summing Einstein A coefficients
    from the ExoMol transitions files to produce per-level radiative lifetimes
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoMol
  role: Source database supplying the 24Mg-1H Yadin line list files (.def, .states,
    .trans) consumed by the lifetimes computation
  repo: https://www.exomol.com/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:648469fa42335165963fbc66dcb91a0b67560f3ab3db85425372f0c321481c7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoMol
  - PyExoCross
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# radiative-lifetime-calculation

## Summary

Compute per-level radiative lifetimes from an ExoMol molecular line list (e.g. 24Mg-1H Yadin) by summing Einstein A coefficients over transitions terminating on each upper state, using PyExoCross's lifetimes post-processing function.

## When to use

Use when you have an ExoMol line list (states and transitions files with Einstein A coefficients) and need radiative lifetimes per energy level as a post-processing product rather than spectra or cross-sections.

## When NOT to use

- You need spectra, cross-sections, or cooling functions — enable StickSpectra/CrossSections/CoolingFunctions instead; lifetimes mode computes only per-level lifetimes
- The dataset's transitions files lack Einstein A coefficients, since lifetimes are the reciprocal of summed A coefficients
- You are working with a HITRAN/HITEMP line list without the transition probability data PyExoCross's lifetimes module expects from ExoMol-format transitions files

## Inputs

- ExoMol line list definition file (e.g. 24Mg-1H Yadin .def)
- ExoMol states file (.states) with per-level energies and quantum numbers
- ExoMol transitions file(s) (.trans) with Einstein A coefficients
- PyExoCross input file (.inp) configured with Database/Molecule/Isotopologue/Dataset and paths

## Outputs

- CSV table of radiative lifetimes, one entry per upper energy level (e.g. 24Mg-1H_Yadin_lifetimes.csv)
- Log file recording the lifetimes run at LogFilePath

## How to apply

Obtain the target line list files (e.g. the 24Mg-1H Yadin .def, .states, and .trans files) from exomol.com and arrange them under ReadPath in the standard ExoMol directory layout ReadPath/MgH/24Mg-1H/Yadin/. Configure a PyExoCross input file with Database=ExoMol, Molecule=MgH, Isotopologue=24Mg-1H, Dataset=Yadin, and set ReadPath, SavePath, and LogFilePath. Enable only the lifetimes function (Lifetimes=1) and set Conversion, PartitionFunctions, SpecificHeats, CoolingFunctions, OscillatorStrengths, StickSpectra, and CrossSections to 0, since the lifetimes module only needs the transitions files' A coefficients. Set parallel-processing parameters (NCPUtrans, NCPUfiles, ChunkSize) and leave Device=CPU (default) unless GPU acceleration is desired; run via `python run.py -p input_filepath`. The lifetimes module sums the Einstein A coefficients of all transitions ending on each upper state to yield the lifetime of that level, saved as 24Mg-1H_Yadin_lifetimes.csv in SavePath.

## Related tools

- **PyExoCross** (Executes the lifetimes post-processing function, summing Einstein A coefficients from the ExoMol transitions files to produce per-level radiative lifetimes) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoMol** (Source database supplying the 24Mg-1H Yadin line list files (.def, .states, .trans) consumed by the lifetimes computation) — https://www.exomol.com/

## Examples

```
python run.py -p ./input/MgH_24Mg-1H_Yadin_lifetimes.inp
```

## Evaluation signals

- Output file 24Mg-1H_Yadin_lifetimes.csv exists at SavePath after the run
- The lifetimes table contains exactly one entry per energy level in the line list states file
- Each lifetime equals the reciprocal of the sum of Einstein A coefficients of all transitions terminating on that level
- All lifetime values are positive and finite
- Log file at LogFilePath shows the lifetimes module completed with only Lifetimes=1 enabled and all other function flags at 0

## Limitations

- GPU mode (Device GPU) on macOS MPS uses float32, which lacks precision; use Nvidia GPU (CUDA) or CPU mode for high-precision lifetimes
- numpy must be >=1.20,<2.0 in the tested environment (Python 3.8–3.12); other version bounds in requirements.txt apply
- Only Lifetimes should be enabled; other function flags (Conversion, PartitionFunctions, etc.) must be set to 0 to avoid unintended computation

## Evidence

- [abstract] PyExoCross implements radiative lifetimes as post-processing: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on"
- [methods] Other functions implemented alongside lifetimes: "partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE stick spectra and cross sections"
- [readme] Running the program via input file: "python run.py -p input_filepath"
- [readme] Device modes available: "`Device CPU` (default)
- `Device GPU` (uses CUDA backend when available)"
- [other] Workflow card: enable only lifetimes: [evidence span withheld — anchor status: not_found]
- [other] Workflow card: validation criterion: [evidence span withheld — anchor status: not_found]
