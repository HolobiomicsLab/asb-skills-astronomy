---
name: molecular-cooling-function-calculation
description: Use when you have a molecular line list (e.g., ExoMol POKAZATEL for 1H2-16O)
  and need temperature-dependent cooling functions W(T) rather than spectra or cross-sections.
when_to_use_negative:
- The goal is cross-sections or stick spectra — these use the CrossSections/StickSpectra
  flags and line-profile machinery (Doppler, Gaussian, Lorentzian, Voigt), not the
  cooling-function routine.
- The target is partition functions, specific heats, radiative lifetimes, or oscillator
  strengths alone — enable those dedicated flags instead of CoolingFunctions.
- The input is an atomic line list requiring ExoAtom/NIST/Kurucz-specific settings
  where cooling-function workflow was not validated in the source card.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: PyExoCross
  role: Python program that computes cooling functions W(T) from molecular line lists
    via the CoolingFunctions input flag; Python adaptation of ExoCross
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment for executing PyExoCross (python run.py -p input_filepath);
    supports Python 3.8–3.12 with numpy/pandas/scipy dependencies
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:359841ff4e048dbb7ab6456ad20979bea04dacd9fbb9316132b2033c8853c945
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# molecular-cooling-function-calculation

## Summary

Computes molecular cooling functions W(T) from ExoMol/HITRAN/HITEMP line lists using PyExoCross's CoolingFunctions mode, summing Einstein-A weighted transition intensities over a temperature grid normalized by the partition function.

## When to use

Use when you have a molecular line list (e.g., ExoMol POKAZATEL for 1H2-16O) and need temperature-dependent cooling functions W(T) rather than spectra or cross-sections.

## When NOT to use

- The goal is cross-sections or stick spectra — these use the CrossSections/StickSpectra flags and line-profile machinery (Doppler, Gaussian, Lorentzian, Voigt), not the cooling-function routine.
- The target is partition functions, specific heats, radiative lifetimes, or oscillator strengths alone — enable those dedicated flags instead of CoolingFunctions.
- The input is an atomic line list requiring ExoAtom/NIST/Kurucz-specific settings where cooling-function workflow was not validated in the source card.

## Inputs

- ExoMol molecular line list files (e.g., .states and .trans for H2O 1H2-16O POKAZATEL dataset)
- ExoMol .def definition file (provides maximum temperature reference and dataset metadata)
- PyExoCross input file (.inp) with CoolingFunctions flag enabled and paths configured

## Outputs

- Output table of cooling-function values W(T) from 1 K to Tmax (10000 K) in 1 K steps

## How to apply

Configure a PyExoCross input file with Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, and the matching ReadPath/SavePath. Enable only CoolingFunctions=1 and set Conversion, PartitionFunctions, SpecificHeats, Lifetimes, OscillatorStrengths, StickSpectra, and CrossSections to 0. Set the temperature grid with Ntemp=1 K steps up to Tmax=10000 K (the ExoMol .def file gives the molecule's maximum temperature as reference) and use Device=CPU, since CPU mode is recommended for cooling-function calculations. PyExoCross then evaluates W(T) = (1/4*pi*Q(T)) * sum over transitions of A*h*c*vwn*g'*exp(-c2*E'/T), with Q(T) = sum_n g_n^tot * exp(-c2*E_n/T).

## Related tools

- **PyExoCross** (Python program that computes cooling functions W(T) from molecular line lists via the CoolingFunctions input flag; Python adaptation of ExoCross) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment for executing PyExoCross (python run.py -p input_filepath); supports Python 3.8–3.12 with numpy/pandas/scipy dependencies) — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- Output table contains cooling-function values for every integer temperature from 1 K through Tmax (10000 K) in 1 K steps — coverage is the acceptance criterion since no quantitative threshold is reported in the paper.
- Output file integrity check: the saved cooling-function file at SavePath is complete and parseable.
- All other function flags (Conversion, PartitionFunctions, SpecificHeats, Lifetimes, OscillatorStrengths, StickSpectra, CrossSections) were 0, confirming only the cooling-function path executed.
- Device was set to CPU (recommended mode for cooling functions), avoiding GPU precision caveats.
- Tmax choice is consistent with the molecule's maximum temperature reported in the ExoMol .def file.

## Limitations

- No quantitative validation threshold is reported in the paper; acceptance is a coverage and file-integrity check (values span 1 K to Tmax in 1 K steps).
- CPU mode is recommended for cooling-function calculations; macOS MPS GPU mode uses float32 which lacks precision for high-precision calculations.
- The .def file only provides the maximum temperature of the molecule as reference for Tmax — it does not guarantee cooling-function validity at that temperature.

## Evidence

- [abstract] PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on molecular line lists.: "PyExoCross generates absorption and emission stick spectra, cross-sections, and other properties (partition functions, specific heats, cooling functions, lifetimes, and oscillator strengths) based on"
- [methods] cooling functions computation listed among implemented computations: "*PyExoCross* also implements the computations of other useful functions including partition functions, specific heats, cooling functions, radiative lifetimes, oscillator strengths, LTE and non-LTE"
- [readme] download and run instructions: "python run.py -p input_filepath"
- [other] temperature grid and device rationale from workflow: [evidence span withheld — anchor status: not_found]
