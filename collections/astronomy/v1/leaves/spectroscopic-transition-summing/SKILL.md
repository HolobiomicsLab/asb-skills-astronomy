---
name: spectroscopic-transition-summing
description: Use when you have an ExoMol molecular line list (e.g. H2O 1H2-16O POKAZATEL
  .trans/.states/.def files) and need a cooling function W(T) on a temperature grid
  rather than spectra or cross-sections.
when_to_use_negative:
- You need stick spectra or cross-sections with line profiles (Doppler/Gaussian/Lorentzian/Voigt)
  rather than the temperature-resolved cooling function
- Your data are in HITRAN/HITEMP format and you primarily need format conversion rather
  than cooling functions
- You need radiative lifetimes or oscillator strengths, which are separate PyExoCross
  function flags
edam_topics:
- http://edamontology.org/topic_0820
- http://edamontology.org/topic_3318
tools:
- name: PyExoCross
  role: Computes the cooling function by Boltzmann-weighted summing over line-list
    transitions using the CoolingFunctions flag; Python adaptation of the ExoCross
    Fortran application
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment for PyExoCross (tested on Python 3.8–3.12)
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:33009cc5897277308babc222a390b1ddf6660d1d0d29eb6ea705c66d2af7cbba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - PyExoCross
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0820
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectroscopic-transition-summing

## Summary

Compute a molecule's cooling function W(T) by thermally weighted Boltzmann summing over all transitions (Einstein A coefficients, wavenumbers, lower-state energies) of a molecular line list, using PyExoCross on ExoMol-format data.

## When to use

Use when you have an ExoMol molecular line list (e.g. H2O 1H2-16O POKAZATEL .trans/.states/.def files) and need a cooling function W(T) on a temperature grid rather than spectra or cross-sections.

## When NOT to use

- You need stick spectra or cross-sections with line profiles (Doppler/Gaussian/Lorentzian/Voigt) rather than the temperature-resolved cooling function
- Your data are in HITRAN/HITEMP format and you primarily need format conversion rather than cooling functions
- You need radiative lifetimes or oscillator strengths, which are separate PyExoCross function flags

## Inputs

- ExoMol line list files (.states, .trans, .def, e.g. 1H2-16O POKAZATEL) available from exomol.com
- PyExoCross input file (.inp) specifying Database, Molecule, Isotopologue, Dataset, ReadPath, SavePath, CoolingFunctions flag and temperature grid

## Outputs

- Cooling-function table W(T) listing values from 1 K up to Tmax K in 1 K steps, written to SavePath

## How to apply

Configure a PyExoCross input file with Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, plus the matching ReadPath and SavePath. Enable only CoolingFunctions=1 and set Conversion, PartitionFunctions, SpecificHeats, Lifetimes, OscillatorStrengths, StickSpectra and CrossSections to 0. Set the temperature grid (Ntemp with 1 K step, Tmax=10000 K; the .def file gives the molecule's maximum temperature as a reference) and set Device=CPU, since CPU mode is recommended for cooling-function calculations. PyExoCross then evaluates W(T) = (1/(4*pi*Q(T))) * sum over transitions of A*h*c*vwn*g'*exp(-c2*E'/T), where the partition function Q(T) = sum_n g_n^tot * exp(-c2*E_n/T). Run via `python run.py -p <input_filepath>`.

## Related tools

- **PyExoCross** (Computes the cooling function by Boltzmann-weighted summing over line-list transitions using the CoolingFunctions flag; Python adaptation of the ExoCross Fortran application) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment for PyExoCross (tested on Python 3.8–3.12)) — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- Output table covers the full requested grid, listing cooling-function values from 1 K up to Tmax K in 1 K steps
- Only the CoolingFunctions flag is set to 1; all other function flags (Conversion, PartitionFunctions, SpecificHeats, Lifetimes, OscillatorStrengths, StickSpectra, CrossSections) remain 0
- Output files exist and are intact in SavePath (file-integrity check, since the paper reports no quantitative threshold)
- W(T) behaves physically: near-zero at very low T (Boltzmann factor exp(-c2*E'/T) suppresses transitions) and rising monotonically toward Tmax
- Run completes under Device=CPU without GPU fallback warnings; the .def file's maximum temperature is consistent with the chosen Tmax

## Limitations

- No quantitative accuracy threshold is reported in the paper; acceptance is based on temperature-grid coverage and file integrity
- CPU mode is recommended for cooling-function calculations; GPU/MPS mode uses float32 on macOS and lacks precision for high-precision calculations
- The temperature grid upper bound should be chosen with reference to the molecule's maximum temperature from the .def definition file

## Evidence

- [methods] Cooling function via transition summing: [evidence span withheld — anchor status: not_found]
- [abstract] PyExoCross property generation scope: "PyExoCross generates other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on molecular line lists."
- [readme] README description of generated properties: [evidence span withheld — anchor status: not_found]
- [readme] CPU vs GPU precision caveat: [evidence span withheld — anchor status: external_doc]
