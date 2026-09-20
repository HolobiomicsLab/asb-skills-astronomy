---
name: partition-function-intensity-weighting
description: Use when a molecular line list (e.g. ExoMol POKAZATEL .states/.trans/.pf/.def
  files for 1H2-16O) must be converted into temperature-dependent absorption/emission
  stick intensities, i.e. when per-line LTE intensity weighting at a specified temperature
  in K is the analysis goal.
when_to_use_negative:
- The goal is broadened, resampled cross-sections rather than unbroadened per-line
  stick intensities (use the cross-section functions with Doppler/Gaussian/Lorentzian/Voigt
  profiles instead)
- The dataset folder lacks required files (no .states/.trans/.pf/.def), so partition-function
  and intensity weighting cannot be performed
- The task is data-format conversion between ExoMol and HITRAN/HITEMP line list formats
  rather than spectral property computation
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PyExoCross
  role: Reads the line list and computes LTE/non-LTE stick spectra by temperature-weighting
    line intensities via the partition function; run via 'python run.py -p input_filepath'
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Fortran predecessor whose functionality PyExoCross adapts; useful as a reference
    implementation for validating stick-spectrum intensities
  repo: https://github.com/ExoMol/ExoCross
- name: Python
  role: Runtime environment for PyExoCross (tested on Python 3.8–3.12, numpy >=1.20
    <2.0); PyExoCross is also installable as the 'pyexocross' PyPI package
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:7de1f366466ea6a492bb81c236af04e58f1203a3221ea9c3b540aa5bfe0afe1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoCross
  - PyExoCross
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# partition-function-intensity-weighting

## Summary

Computes LTE (and non-LTE) stick spectra by temperature-weighting transition intensities from a molecular line list through the partition function, as implemented in PyExoCross, a Python adaptation of the ExoCross Fortran code.

## When to use

Use when a molecular line list (e.g. ExoMol POKAZATEL .states/.trans/.pf/.def files for 1H2-16O) must be converted into temperature-dependent absorption/emission stick intensities, i.e. when per-line LTE intensity weighting at a specified temperature in K is the analysis goal.

## When NOT to use

- The goal is broadened, resampled cross-sections rather than unbroadened per-line stick intensities (use the cross-section functions with Doppler/Gaussian/Lorentzian/Voigt profiles instead)
- The dataset folder lacks required files (no .states/.trans/.pf/.def), so partition-function and intensity weighting cannot be performed
- The task is data-format conversion between ExoMol and HITRAN/HITEMP line list formats rather than spectral property computation

## Inputs

- ExoMol line list files: .states, .trans, .pf, and .def files (e.g. 1H2-16O POKAZATEL from exomol.com)
- PyExoCross input file (.inp) specifying Database, Molecule, Isotopologue, Dataset, SpeciesID, ReadPath, SavePath, LogFilePath
- Temperature in K for the LTE stick-spectrum calculation

## Outputs

- Stick-spectrum tables written to SavePath (e.g. stick_spectra.csv with wavenumber and intensity columns for absorption and emission)
- Optional log file at LogFilePath recording the run

## How to apply

Assemble the complete ExoMol dataset under ReadPath in the Molecule/Isotopologue/Dataset folder structure (states, transitions, .pf, and .def files) and author an input file with Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, SpeciesID=11 plus ReadPath/SavePath/LogFilePath. In the Functions section set StickSpectra=1 and all other functions 0, choose CPU mode with default NCPUtrans/NCPUfiles/ChunkSize, and specify the desired temperature in K for the LTE calculation. Run with 'python run.py -p <input_filepath>'; the stick-spectra module reads the line list and derives temperature-weighted line intensities via the partition function (Q(T) from the .pf file), applying Boltzmann population weighting to lower-state energies before summing absorption and emission intensities. Optionally use UncFilter/Threshold/ConvUncFilter/ConvThreshold filters (in cm⁻¹ and cm/molecule) to restrict lines before weighting. Rationale: stick spectra are the unbroadened, per-transition counterparts of cross-sections, so no line profile (Doppler, Gaussian, Lorentzian, or Voigt) sampling is needed at this stage.

## Related tools

- **PyExoCross** (Reads the line list and computes LTE/non-LTE stick spectra by temperature-weighting line intensities via the partition function; run via 'python run.py -p input_filepath') — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Fortran predecessor whose functionality PyExoCross adapts; useful as a reference implementation for validating stick-spectrum intensities) — https://github.com/ExoMol/ExoCross
- **Python** (Runtime environment for PyExoCross (tested on Python 3.8–3.12, numpy >=1.20 <2.0); PyExoCross is also installable as the 'pyexocross' PyPI package) — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- Output stick_spectra.csv exists in SavePath and contains wavenumber and intensity columns for both absorption and emission
- Stick-spectrum pattern matches the stick spectra reported in the PyExoCross paper for the same molecule/isotopologue/dataset and temperature
- Intensity weighting reflects the specified temperature in K: relative line intensities change consistently when temperature is varied, consistent with Boltzmann populations and Q(T) from the .pf file
- Log file shows all functions other than StickSpectra were disabled (StickSpectra=1, others 0) and the run completed over the expected number of transition files
- Filtered outputs (if UncFilter/Threshold used) contain only lines meeting the requested uncertainty (cm⁻¹) or intensity (cm/molecule) thresholds

## Limitations

- Requires the full ExoMol folder structure under ReadPath (states, transitions, .pf, .def); incomplete downloads will fail
- LTE stick intensities depend on the provided partition function (.pf file); non-LTE calculations need additional non-LTE treatment
- macOS MPS GPU mode uses float32 and lacks precision, so use Nvidia GPU (CUDA) or CPU mode for high-precision calculations; stick-spectra runs default to CPU mode with NCPUtrans/NCPUfiles/ChunkSize settings
- Python version and dependency bounds are constrained (Python 3.8–3.12, numpy >=1.20 <2.0, pandas >=1.4 <3.0), and no changelog is available for the repository

## Evidence

- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections: "PyExoCross generates absorption and emission stick spectra, cross-sections"
- [methods] partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE stick spectra and cross sections: "partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE stick spectra and cross sections"
- [readme] In the terminal, use the following commands to run PyExoCross:: [evidence span withheld — anchor status: external_doc]
- [readme] designed for postprocessing the huge molecular line lists generated by the ExoMol project: "designed for postprocessing the huge molecular line lists generated by the ExoMol project"
