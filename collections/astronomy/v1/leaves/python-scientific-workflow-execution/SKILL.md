---
name: python-scientific-workflow-execution
description: Use when a task requires running a documented Python program (e.g. PyExoCross)
  over large spectroscopic line-list data (ExoMol, HITRAN, HITEMP formats) by configuring
  an input file and validating its generated output tables.
when_to_use_negative:
- The goal is spectral synthesis (stick spectra or cross-sections with Doppler/Gaussian/Lorentzian/Voigt
  profiles) rather than scalar property computation — enable the spectra/cross-section
  functions instead of lifetimes.
- The data are in HITRAN/HITEMP format and only format conversion between ExoMol and
  HITRAN is needed — use the Conversion function, not the lifetimes module.
- Line-list files are not present locally and must be fetched — use PyExoCross's batch-download
  code for ExoMol files before running post-processing.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0092
tools:
- name: PyExoCross
  role: Python program whose input-file-driven workflow (run.py) computes radiative
    lifetimes and other post-processing properties from ExoMol line lists
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment (3.8–3.12) in which PyExoCross and its dependencies (numpy,
    pandas, scipy, dask, etc.) are installed via requirements.txt or pip
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoMol
  role: Source database providing the .def, .states, and .trans files consumed by
    the workflow
  repo: https://www.exomol.com/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:1e3f708ed3de5422aeda64e6ced711280237e9164e27d86221d0ab0eedbc2216
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoMol
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# python-scientific-workflow-execution

## Summary

Executes a configured Python scientific post-processing program (PyExoCross) over a molecular line list by writing a structured input file, running `python run.py`, and validating the tabular outputs. The workflow covers directory layout, function flags, parallelism settings, and output checks for tasks such as computing radiative lifetimes from the 24Mg-1H Yadin ExoMol line list.

## When to use

Use when a task requires running a documented Python program (e.g. PyExoCross) over large spectroscopic line-list data (ExoMol, HITRAN, HITEMP formats) by configuring an input file and validating its generated output tables.

## When NOT to use

- The goal is spectral synthesis (stick spectra or cross-sections with Doppler/Gaussian/Lorentzian/Voigt profiles) rather than scalar property computation — enable the spectra/cross-section functions instead of lifetimes.
- The data are in HITRAN/HITEMP format and only format conversion between ExoMol and HITRAN is needed — use the Conversion function, not the lifetimes module.
- Line-list files are not present locally and must be fetched — use PyExoCross's batch-download code for ExoMol files before running post-processing.

## Inputs

- ExoMol line-list files (.def definition file, .states file, .trans transitions file) for 24Mg-1H Yadin from exomol.com
- PyExoCross input file (.inp) specifying Database, Molecule, Isotopologue, Dataset, ReadPath, SavePath, LogFilePath, function flags, and parallelism parameters

## Outputs

- Radiative lifetimes table saved to SavePath as 24Mg-1H_Yadin_lifetimes.csv, one lifetime per energy level
- Log file at LogFilePath recording the run

## How to apply

Obtain the line-list files (def, states, transitions) from exomol.com and arrange them in the standard directory layout ReadPath/MgH/24Mg-1H/Yadin/ for the MgH Yadin dataset. Write the PyExoCross input file with Database=ExoMol, Molecule=MgH, Isotopologue=24Mg-1H, Dataset=Yadin, plus ReadPath, SavePath, and LogFilePath, keeping the first-column strings unchanged. Enable exactly the target function (e.g. Lifetimes=1) and set all others (Conversion, PartitionFunctions, SpecificHeats, CoolingFunctions, OscillatorStrengths, StickSpectra, CrossSections) to 0 so the run computes only radiative lifetimes, summing Einstein A coefficients per upper state. Set parallel-processing parameters (NCPUtrans, NCPUfiles, ChunkSize) and leave Device=CPU (default; GPU is an optional fallback-capable mode). Run the program and validate that the output CSV exists, has one entry per energy level, and that each lifetime equals the reciprocal of the summed A coefficients for that level.

## Related tools

- **PyExoCross** (Python program whose input-file-driven workflow (run.py) computes radiative lifetimes and other post-processing properties from ExoMol line lists) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment (3.8–3.12) in which PyExoCross and its dependencies (numpy, pandas, scipy, dask, etc.) are installed via requirements.txt or pip) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoMol** (Source database providing the .def, .states, and .trans files consumed by the workflow) — https://www.exomol.com/

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- Output file 24Mg-1H_Yadin_lifetimes.csv exists at SavePath after the run completes.
- The CSV contains one lifetime entry per energy level of the line list (row count matches the .states file).
- Each lifetime equals the reciprocal of the summed Einstein A coefficients of transitions terminating on that level.
- All lifetime values are positive and finite.
- The log file at LogFilePath shows only the lifetimes function executed (Lifetimes=1, all other function flags 0).

## Limitations

- GPU mode on macOS MPS uses float32, which lacks precision; use Nvidia GPU (CUDA) or CPU mode for high-precision calculations.
- The program is designed for post-processing huge ExoMol/HITRAN/HITEMP line lists — it is not a general-purpose spectroscopy tool, and unsupported databases (e.g. VAMDC) are pending.
- Only the functions explicitly enabled in the input file run; leaving unrelated flags on produces unwanted extra outputs.

## Evidence

- [readme] PyExoCross generates absorption and emission spectra and other properties (partition functions, specific heats, cooling functions, lifetimes, oscillator strengths, and stick spectra) based on molecular line lists.: [evidence span withheld — anchor status: not_found]
- [methods] partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE stick spectra and cross sections: "partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE stick spectra and cross sections"
- [readme] Device CPU (default) / Device GPU (uses CUDA backend when available) ... GPU mode is optional and automatically falls back to CPU if CUDA runtime packages or devices are unavailable.: [evidence span withheld — anchor status: not_found]
- [readme] All information can be written in the input file. Just change the information you will use.You don't need to change any other unnecessary information.Please do not change the first column strings.: "All information can be written in the input file. Just change the information you will use.You don't need to change any other unnecessary information.Please do not change the first column strings."
- [abstract] extra code for users to automate the batch download of line list files from the ExoMol data base: "extra code for users to automate the batch download of line list files from the ExoMol data base"
