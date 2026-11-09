---
name: voigt-profile-cross-section-computation
description: Use when temperature- and pressure-dependent LTE absorption cross-sections
  are needed from an ExoMol/HITRAN line list (e.g. H2O POKAZATEL) with a Voigt line
  profile over a specified wavenumber grid.
when_to_use_negative:
- If stick spectra, partition functions, specific heats, cooling functions, lifetimes,
  or oscillator strengths are the target — PyExoCross exposes these as separate function
  flags, so CrossSections=1 is the wrong configuration
- If binned (rather than sampled) cross-sections are required — use the binned computation
  method instead of the sampling grid with Npoints
- If the line list is not in ExoMol or HITRAN/HITEMP format (e.g. VAMDC databases,
  which are still pending support)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3892
tools:
- name: PyExoCross
  role: Python program that convolves the ExoMol line list with the Voigt profile
    in sampling mode to produce cross-sections; run via run.py with an .inp input
    file
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment (Python 3.8–3.12) in which PyExoCross is installed and
    executed
  repo: https://github.com/Beryl-Jingxin/PyExoCross
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:4046724cd4ac1e1857ede1ed2a8033a166f1166ce6afaef8956b9eb898a50527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3892
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# voigt-profile-cross-section-computation

## Summary

Compute molecular absorption cross-sections from the ExoMol H2O POKAZATEL line list using PyExoCross's Voigt line profile in sampling mode, producing a .xsec file that can be validated against reference cross sections on exomol.com.

## When to use

Use when temperature- and pressure-dependent LTE absorption cross-sections are needed from an ExoMol/HITRAN line list (e.g. H2O POKAZATEL) with a Voigt line profile over a specified wavenumber grid.

## When NOT to use

- If stick spectra, partition functions, specific heats, cooling functions, lifetimes, or oscillator strengths are the target — PyExoCross exposes these as separate function flags, so CrossSections=1 is the wrong configuration
- If binned (rather than sampled) cross-sections are required — use the binned computation method instead of the sampling grid with Npoints
- If the line list is not in ExoMol or HITRAN/HITEMP format (e.g. VAMDC databases, which are still pending support)

## Inputs

- ExoMol POKAZATEL states file (1H2-16O__POKAZATEL.states.bz2)
- ExoMol POKAZATEL transitions files (1H2-16O__POKAZATEL.trans.bz2)
- ExoMol .def definition file from exomol.com
- PyExoCross input file (e.g. input/H2O_exomol.inp) with CrossSections=1, LTE settings, wavenumber Range/Npoints grid, Voigt Profile and HWHM settings

## Outputs

- .xsec cross-section file (wavenumber grid in first column with computed cross-section values in the second)
- log file when run with nohup (e.g. ./output/H2O_exomol.out)

## How to apply

Clone PyExoCross and start from the packaged example input/H2O_exomol.inp, keeping the Data source block (Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, SpeciesID=11). Place the 1H2-16O__POKAZATEL.states.bz2 and .trans.bz2 files under ReadPath in the standard ExoMol directory layout, and in the Functions section set CrossSections=1 with all other function flags 0. In the Cross sections section, configure LTE absorption with WnWlUnit=cm-1, a wavenumber Range, an Npoints sampling grid, default Broadeners with Ratios 1.0, and a Voigt Profile option (e.g. SciPyVoigt) with Cutoff(Y/N) plus DopplerHWHM/LorentzianHWHM so temperature- and pressure-dependent widths are computed. Set the Cores and chunks block (NCPUtrans, NCPUfiles, ChunkSize) with Device=CPU to parallelise reading of the .trans files, then run 'python run.py -p ./input/H2O_exomol.inp' so the cross_sections routine convolves the line list with the Voigt profile over the sampling grid. Retrieve the .xsec file from SavePath and compare its wavenumber grid, peak positions, and magnitudes against reference H2O cross sections from exomol.com.

## Related tools

- **PyExoCross** (Python program that convolves the ExoMol line list with the Voigt profile in sampling mode to produce cross-sections; run via run.py with an .inp input file) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment (Python 3.8–3.12) in which PyExoCross is installed and executed) — https://github.com/Beryl-Jingxin/PyExoCross

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The .xsec file exists in SavePath and contains the wavenumber grid in the first column with computed cross-section values
- Peak positions (wavenumbers) of the computed Voigt profile match the reference H2O POKAZATEL cross sections deposited on exomol.com
- Peak magnitudes of the computed cross-sections are consistent with the exomol.com reference cross sections
- The run completes the cross_sections routine with the Cores and chunks settings (NCPUtrans, NCPUfiles, ChunkSize, Device=CPU) without errors, optionally logged via nohup

## Limitations

- macOS MPS GPU mode only uses float32 and lacks precision; Nvidia GPU (CUDA) or CPU mode is recommended for high-precision calculations
- Voigt profile computation requires Cutoff(Y/N) and DopplerHWHM/LorentzianHWHM to be set so widths are computed; wing cutoff is specified in cm-1 when Cutoff is enabled
- PyExoCross 2.0 citation is 'to be submitted' with an empty DOI, and no changelog was found in the discussion content
- Only Doppler, Gaussian, Lorentzian, and Voigt profiles are available; other line shapes are not supported

## Evidence

- [abstract] Four line profiles for cross sections: "PyExoCross calculates cross-sections with four line profiles: Doppler, Gaussian, Lorentzian, and Voigt profiles in both sampling and binned methods; a number of options are available for computing"
- [methods] Cutoff parameter meaning: "`Cutoff(Y/N)`: If `Cutoff(Y/N)` is yes, you can provide wing cutoff here in unit of cm⁻¹."
- [methods] Predissociation with Voigt: "`PredissocXsec(Y/N)`: If `PredissocXsec(Y/N)` is yes, predissociation lifetimes will be used or calculated when calculating cross sections with Voigt profile."
- [readme] Run command from README: "python run.py -p ./input/H2O_exomol.inp"
- [readme] Download instructions from README: [evidence span withheld — anchor status: not_found]
- [readme] CPU/GPU device selection: [evidence span withheld — anchor status: external_doc]
