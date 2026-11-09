---
name: lte-spectroscopic-post-processing
description: Use when you have an ExoMol, ExoMolHR, ExoAtom, HITRAN, or HITEMP molecular/atomic
  line list and need LTE or non-LTE stick spectra, cross-sections, or derived properties
  such as partition functions or cooling functions.
when_to_use_negative:
- You need a smoothed/broadened spectrum rather than discrete line positions and intensities
  — use the cross-section computation with Doppler/Gaussian/Lorentzian/Voigt profiles
  instead of StickSpectra.
- The input is not a molecular/atomic line list (e.g. an already-generated spectrum
  or feature table) — PyExoCross post-processes line lists, so there is nothing to
  compute.
- 'High-precision work on macOS: MPS GPU mode only uses float32 which lacks precision,
  so GPU mode is inappropriate there.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0176
tools:
- name: PyExoCross
  role: 'Primary tool: reads line lists and generates LTE/non-LTE absorption and emission
    stick spectra and cross-sections via ''python run.py -p <input_filepath>'''
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Fortran predecessor whose functionality PyExoCross adapts for spectroscopic
    post-processing
  repo: https://github.com/ExoMol/ExoCross
- name: Python
  role: Runtime environment for PyExoCross (tested on Python 3.8–3.12, installed via
    pip or git clone with requirements.txt)
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:c5805fe5a8d45d472c09acc094bc3f16edc1dfe0e08b6cdcc62fdfa22a3815a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoCross
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0176
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# lte-spectroscopic-post-processing

## Summary

Generate LTE (and non-LTE) absorption and emission stick spectra from huge molecular line lists using PyExoCross, a Python adaptation of the ExoCross Fortran code, with temperature-weighted line intensities computed via the partition function.

## When to use

Use when you have an ExoMol, ExoMolHR, ExoAtom, HITRAN, or HITEMP molecular/atomic line list and need LTE or non-LTE stick spectra, cross-sections, or derived properties such as partition functions or cooling functions.

## When NOT to use

- You need a smoothed/broadened spectrum rather than discrete line positions and intensities — use the cross-section computation with Doppler/Gaussian/Lorentzian/Voigt profiles instead of StickSpectra.
- The input is not a molecular/atomic line list (e.g. an already-generated spectrum or feature table) — PyExoCross post-processes line lists, so there is nothing to compute.
- High-precision work on macOS: MPS GPU mode only uses float32 which lacks precision, so GPU mode is inappropriate there.

## Inputs

- ExoMol line list files (states, transitions, .pf, .def) e.g. POKAZATEL for 1H2-16O
- HITRAN/HITEMP line list files
- PyExoCross input file (.inp) with Database, Molecule, Isotopologue, Dataset, SpeciesID, ReadPath, SavePath, LogFilePath, Functions, and temperature settings
- Desired temperature in K for LTE stick-spectrum calculation

## Outputs

- Stick-spectrum tables (stick_spectra.csv) with wavenumber and intensity columns for absorption and emission
- Cross-sections (Doppler, Gaussian, Lorentzian, Voigt profiles; sampling and binned methods)
- Other properties: partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths

## How to apply

Download the relevant line list files from exomol.com (e.g. the POKAZATEL line list for 1H2-16O: states, transitions, .pf, and .def files) and arrange them under ReadPath in the Molecule/Isotopologue/Dataset folder structure. Prepare a PyExoCross input file setting Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, SpeciesID=11, with ReadPath, SavePath, and LogFilePath configured; in the Functions section set StickSpectra=1 (all other functions 0), choose CPU mode (default NCPUtrans/NCPUfiles/ChunkSize, or GPU mode via device='GPU' with bounded GPUBatchLines/GPUBatchGrid), and specify the desired temperature in K for the LTE stick-spectrum calculation. Optional filters (UncFilter, Threshold, QNsFilter, ConvUncFilter, ConvThreshold) can restrict transitions by maximum uncertainty in cm⁻¹ or minimum intensity in cm/molecule before computing. Run 'python run.py -p <input_filepath>'; the stick-spectra module reads the line list, computes temperature-weighted line intensities via the partition function, and writes stick-spectrum tables to SavePath. Verify that the output stick_spectra.csv exists and contains wavenumber and intensity columns for absorption and emission matching the spectra reported in the PyExoCross paper.

## Related tools

- **PyExoCross** (Primary tool: reads line lists and generates LTE/non-LTE absorption and emission stick spectra and cross-sections via 'python run.py -p <input_filepath>') — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Fortran predecessor whose functionality PyExoCross adapts for spectroscopic post-processing) — https://github.com/ExoMol/ExoCross
- **Python** (Runtime environment for PyExoCross (tested on Python 3.8–3.12, installed via pip or git clone with requirements.txt)) — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The output stick_spectra.csv exists in SavePath and contains wavenumber and intensity columns for both absorption and emission.
- Computed stick spectra match the stick spectra reported in the PyExoCross paper for the same molecule/dataset/temperature.
- Line intensities are temperature-weighted consistently with the partition function read from the .pf file.
- Log file at LogFilePath shows the line list was fully read with the expected number of transitions (after any UncFilter/Threshold/QNsFilter).
- If GPU mode is requested, the run either uses CUDA (PyTorch/CuPy) or explicitly falls back to CPU without numerical discrepancies.

## Limitations

- macOS MPS GPU mode only supports float32 which lacks precision; use an NVIDIA GPU (CUDA) or CPU mode for high-precision calculations.
- GPU mode falls back to CPU if CUDA runtime packages or devices are unavailable.
- No changelog is documented for the repository, so version-to-version behavioral changes may be hard to track.
- VAMDC databases are only partially supported (marked as pending) compared with ExoMol, ExoMolHR, ExoAtom, HITRAN, and HITEMP.

## Evidence

- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on molecular line lists.: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths)"
- [intro] A Python framework for LTE and non-LTE spectroscopic post-processing of atomic and molecular line lists.: "A Python framework for LTE and non-LTE spectroscopic post-processing of atomic and molecular line lists"
- [methods] ``UncFilter(Y/N)``, ``Threshold(Y/N)``, and ``QNsFilter(Y/N)`` are filters: "``UncFilter(Y/N)``, ``Threshold(Y/N)``, and ``QNsFilter(Y/N)`` are filters"
- [readme] python run.py -p input_filepath: "python run.py -p input_filepath"
- [readme] GPU mode is optional and automatically falls back to CPU if CUDA runtime packages or devices are unavailable.: [evidence span withheld — anchor status: external_doc]
