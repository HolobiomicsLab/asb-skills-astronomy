---
name: lte-spectral-simulation
description: Use when a task requires LTE absorption cross-sections from an ExoMol/HITRAN
  line list with a Voigt profile in sampling mode, e.g. reproducing H2O 1H2-16O POKAZATEL
  cross-sections for comparison against exomol.com reference data.
when_to_use_negative:
- The goal is stick spectra, partition functions, cooling functions, lifetimes, or
  oscillator strengths rather than cross-sections — those require different function
  flags in the input file.
- The target is a binned (rather than sampled) cross-section computation, which selects
  a different computation method in the Cross sections section.
- Only a Doppler, Gaussian, or Lorentzian profile is required; the Voigt-profile configuration
  (SciPyVoigt, Cutoff, DopplerHWHM/LorentzianHWHM) is unnecessary.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3473
tools:
- name: PyExoCross
  role: Python program that computes the LTE cross-sections by convolving the ExoMol
    line list with a Voigt profile over the sampling grid via its cross_sections routine
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime language used to execute PyExoCross with 'python run.py -p ./input/H2O_exomol.inp'
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:c1de9a65249cb7ce7c5e74b062a24e3adc08d83f84fb5bbd189203ff55f4eb68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# lte-spectral-simulation

## Summary

Compute temperature- and pressure-dependent molecular cross-sections in LTE from an ExoMol line list (e.g. H2O POKAZATEL) using PyExoCross's sampling method with a Voigt line profile, and validate the output against reference cross sections from exomol.com.

## When to use

Use when a task requires LTE absorption cross-sections from an ExoMol/HITRAN line list with a Voigt profile in sampling mode, e.g. reproducing H2O 1H2-16O POKAZATEL cross-sections for comparison against exomol.com reference data.

## When NOT to use

- The goal is stick spectra, partition functions, cooling functions, lifetimes, or oscillator strengths rather than cross-sections — those require different function flags in the input file.
- The target is a binned (rather than sampled) cross-section computation, which selects a different computation method in the Cross sections section.
- Only a Doppler, Gaussian, or Lorentzian profile is required; the Voigt-profile configuration (SciPyVoigt, Cutoff, DopplerHWHM/LorentzianHWHM) is unnecessary.

## Inputs

- ExoMol POKAZATEL states file (1H2-16O__POKAZATEL.states.bz2)
- ExoMol POKAZATEL transitions files (1H2-16O__POKAZATEL.trans.bz2) in the standard ExoMol directory layout
- PyExoCross input file (e.g. input/H2O_exomol.inp) with Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, SpeciesID=11 and CrossSections=1
- Reference H2O cross sections from exomol.com for validation

## Outputs

- Cross-section file (.xsec) with the wavenumber grid (cm-1) in the first column and computed LTE absorption cross-section values
- Program log from the cross_sections routine run

## How to apply

Clone the PyExoCross repository and start from the packaged example input file input/H2O_exomol.inp, keeping its Data source block (Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, SpeciesID=11). Place the 1H2-16O__POKAZATEL.states.bz2 and .trans.bz2 files under ReadPath in the standard ExoMol directory layout and set CrossSections=1 (all other function flags 0). In the Cross sections section, configure LTE absorption with WnWlUnit in cm-1, a wavenumber Range, an Npoints sampling grid, default Broadeners (Ratios 1.0), and a Voigt profile option (e.g. SciPyVoigt) with Cutoff(Y/N) and DopplerHWHM/LorentzianHWHM so temperature- and pressure-dependent widths are computed. Parallelise reading and processing of the .trans files via the Cores and chunks block (NCPUtrans, NCPUfiles, ChunkSize) with Device=CPU, then run 'python run.py -p ./input/H2O_exomol.inp' and collect the .xsec file from SavePath.

## Related tools

- **PyExoCross** (Python program that computes the LTE cross-sections by convolving the ExoMol line list with a Voigt profile over the sampling grid via its cross_sections routine) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime language used to execute PyExoCross with 'python run.py -p ./input/H2O_exomol.inp') — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The output .xsec file exists at SavePath and its first column contains the wavenumber grid (cm-1) with computed cross-section values in the second column.
- Profile peak positions (wavenumbers) in the computed .xsec file match the line positions of the H2O reference cross sections from exomol.com.
- Peak magnitudes of the computed cross-sections are consistent with the exomol.com reference cross sections.
- The input file has CrossSections=1 and all other function flags set to 0, so only the cross_sections routine ran.
- The wavenumber grid spans the configured Range with the configured Npoints sampling points.

## Limitations

- macOS MPS GPU mode only uses float32 which lacks precision, so CPU mode (or Nvidia CUDA) should be used for high-precision cross-section calculations.
- The Voigt-profile option 'predissociation lifetimes' (PredissocXsec) applies only when computing cross sections with Voigt profile and depends on availability of lifetimes data.
- Optional GPU acceleration (PyTorch-CUDA/CuPy) automatically falls back to CPU if CUDA runtime packages or devices are unavailable; performance expectations must account for that.

## Evidence

- [abstract] PyExoCross calculates cross-sections with four line profiles (Doppler, Gaussian, Lorentzian, and Voigt) and offers both sampling and binned computation methods.: "PyExoCross calculates cross-sections with four line profiles: Doppler, Gaussian, Lorentzian, and Voigt profiles in both sampling and binned methods"
- [methods] Cross sections section configures WnWlUnit in cm-1, Range, Npoints, Broadeners Ratios 1.0, Voigt Profile option with Cutoff and HWHMs.: "`Cutoff(Y/N)`: If `Cutoff(Y/N)` is yes, you can provide wing cutoff here in unit of cm⁻¹."
- [methods] Predissociation lifetimes are used in Voigt-profile cross-section calculations when enabled.: "If `PredissocXsec(Y/N)` is yes, predissociation lifetimes will be used or calculated when calculating cross sections with Voigt profile."
- [readme] README documents the exact CLI invocation used to run the cross-section workflow.: "python run.py -p ./input/H2O_exomol.inp"
- [readme] README confirms ExoMol directory data and PyExoCross program download/execution path.: "Download PyExoCross program by command:

bash
git clone https://github.com/Beryl-Jingxin/PyExoCross.git"
