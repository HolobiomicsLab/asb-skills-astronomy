---
name: exomol-line-list-processing
description: Use when the input is an ExoMol molecular line list (e.g. 1H2-16O POKAZATEL
  .states.bz2/.trans.bz2/.pf/.def files) and derived spectroscopic properties such
  as LTE stick spectra, partition functions, lifetimes, cooling functions, or oscillator
  strengths must be computed.
when_to_use_negative:
- The input is not a molecular or atomic line list (e.g. raw observational spectra
  or images) — PyExoCross is designed for postprocessing line lists, not direct observation
  fitting
- The database is not one of the supported formats (ExoMol, ExoMolHR, ExoAtom/NIST/Kurucz,
  HITRAN, HITEMP, VAMDC) — format conversion or a different tool is needed first
- High-precision GPU work on macOS MPS is attempted — the README notes MPS only uses
  float32 which lacks precision, so use Nvidia CUDA or CPU mode instead
tools:
- name: PyExoCross
  role: Primary tool that reads ExoMol/HITRAN line lists and computes stick spectra,
    cross-sections, partition functions, specific heats, cooling functions, lifetimes,
    and oscillator strengths
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Fortran predecessor whose functionality PyExoCross adapts; the workflow conventions
    (input file sections, partition-function convention) follow ExoCross
  repo: https://github.com/ExoMol/ExoCross
- name: ExoMol
  role: Source database providing the molecular line list files (states, transitions,
    .pf, .def) downloaded from exomol.com
  repo: https://www.exomol.com/
- name: Python
  role: Runtime environment for PyExoCross (tested on Python 3.8–3.12); program invoked
    with 'python run.py -p <input_filepath>'
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:19ac785b72764736eac438a5cbd43b0664a60f08029860165128bf74cdf7c965
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoCross
  - ExoMol
  - PyExoCross
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# exomol-line-list-processing

## Summary

Post-process ExoMol (and HITRAN/HITEMP) molecular line lists with PyExoCross, a Python adaptation of the ExoCross Fortran code, to generate stick spectra, cross-sections, partition functions, specific heats, cooling functions, radiative lifetimes, and oscillator strengths.

## When to use

Use when the input is an ExoMol molecular line list (e.g. 1H2-16O POKAZATEL .states.bz2/.trans.bz2/.pf/.def files) and derived spectroscopic properties such as LTE stick spectra, partition functions, lifetimes, cooling functions, or oscillator strengths must be computed.

## When NOT to use

- The input is not a molecular or atomic line list (e.g. raw observational spectra or images) — PyExoCross is designed for postprocessing line lists, not direct observation fitting
- The database is not one of the supported formats (ExoMol, ExoMolHR, ExoAtom/NIST/Kurucz, HITRAN, HITEMP, VAMDC) — format conversion or a different tool is needed first
- High-precision GPU work on macOS MPS is attempted — the README notes MPS only uses float32 which lacks precision, so use Nvidia CUDA or CPU mode instead

## Inputs

- ExoMol line list files: Isotopologue__Dataset.states.bz2 (energy levels, degeneracies), Isotopologue__Dataset.trans.bz2 (transitions with Einstein A coefficients), .pf partition function file, and .def definition file from exomol.com
- PyExoCross input file (.inp) with Database, Molecule, Isotopologue, Dataset, SpeciesID, ReadPath, SavePath, LogFilePath, Functions flags, and processing parameters

## Outputs

- Stick-spectrum table (stick_spectra.csv) with wavenumber and intensity columns for absorption and emission at a specified LTE temperature
- Partition-function table <Isotopologue>__<Dataset>.pf with Q(T) from 1 K to Tmax in 1 K steps
- Lifetimes table (e.g. 24Mg-1H_Yadin_lifetimes.csv) with one radiative lifetime per energy level
- Cooling-function table W(T) from 1 K to Tmax in 1 K steps
- Oscillator-strength table with entries per requested transition

## How to apply

Download the ExoMol dataset files (states, transitions, .pf, .def) from exomol.com and arrange them under ReadPath in the Molecule/Isotopologue/Dataset folder layout (e.g. ReadPath/H2O/1H2-16O/POKAZATEL/). Write a PyExoCross input file setting Database=ExoMol plus Molecule, Isotopologue, Dataset, and SpeciesID identifiers, configure ReadPath/SavePath/LogFilePath, and enable exactly one Functions flag (e.g. StickSpectra=1, PartitionFunctions, CoolingFunctions, Lifetimes, or OscillatorStrengths) with all others set to 0. Run in default Device=CPU mode with parallel parameters NCPUtrans/NCPUfiles and ChunkSize=1000000 for chunked reading of .states.bz2 and .trans.bz2 files via 'python run.py -p <input_filepath>'. Property calculations use the Boltzmann-weighted sums over levels: Q(T) = sum_n g_n * exp(-c2 * E_n / T) for partition functions (saved as <Isotopologue>__<Dataset>.pf in 1 K steps up to Tmax), radiative lifetimes from summed Einstein A coefficients per upper state, and cooling functions W(T) = (1/4*pi*Q(T)) * sum of A*h*c*vwn*g'*exp(-c2*E'/T). Optional line filters UncFilter(Y/N), Threshold(Y/N), and QNsFilter(Y/N) restrict processed transitions by uncertainty, intensity, and quantum numbers. For stick spectra, specify the desired LTE temperature in K and validate that the output stick_spectra.csv contains wavenumber and intensity columns for absorption and emission.

## Related tools

- **PyExoCross** (Primary tool that reads ExoMol/HITRAN line lists and computes stick spectra, cross-sections, partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Fortran predecessor whose functionality PyExoCross adapts; the workflow conventions (input file sections, partition-function convention) follow ExoCross) — https://github.com/ExoMol/ExoCross
- **ExoMol** (Source database providing the molecular line list files (states, transitions, .pf, .def) downloaded from exomol.com) — https://www.exomol.com/
- **Python** (Runtime environment for PyExoCross (tested on Python 3.8–3.12); program invoked with 'python run.py -p <input_filepath>') — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- Output files exist in SavePath with expected names: stick_spectra.csv with wavenumber and intensity columns for absorption and emission; <Isotopologue>__<Dataset>.pf starting at 1 K and ending at Tmax K with 1 K steps
- Computed partition functions Q(T) agree closely with the reference .pf file distributed with the ExoMol dataset
- Each computed radiative lifetime equals the reciprocal of the summed Einstein A coefficients of transitions terminating on that level, with one entry per energy level and positive finite values
- Stick spectra match those reported in the PyExoCross paper for the same dataset (e.g. 1H2-16O POKAZATEL)
- Run details are recorded to LogFilePath and the cooling-function table covers the full 1 K to Tmax grid without gaps

## Limitations

- macOS MPS GPU mode only uses float32, which lacks precision; use Nvidia CUDA GPU or CPU mode for high-precision calculations
- No changelog or substantive discussion section content was available for this article version (PyExoCross 2.0 citation is 'to be submitted')
- No quantitative accuracy thresholds are reported for some properties (e.g. cooling functions), so validation relies on coverage and file-integrity checks plus expert review for physical plausibility

## Evidence

- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties"
- [methods] partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths: "partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE stick spectra and cross sections"
- [readme] designed for postprocessing huge molecular line lists: "designed for postprocessing the huge molecular line lists generated by the ExoMol project and other similar initiatives such as the HITRAN and HITEMP"
- [readme] program invocation command: "python run.py -p ./input/H2O_exomol.inp"
- [methods] filter options for restricting processed lines: "`ConvUncFilter(Y/N)`: If it is yes, the value is the maximum uncertainty you require, in unit of cm⁻¹."
