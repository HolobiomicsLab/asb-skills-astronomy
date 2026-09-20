---
name: molecular-line-list-processing
description: Use when a task requires LTE (or non-LTE) absorption/emission cross-sections
  or stick spectra computed from an ExoMol/HITRAN/HITEMP molecular line list, e.g.
  reproducing H2O POKAZATEL cross-sections with a Voigt profile in sampling mode.
when_to_use_negative:
- The goal is to convert line list data between ExoMol and HITRAN formats rather than
  compute spectra or cross-sections (a separate PyExoCross conversion function).
- Only raw partition functions, specific heats, cooling functions, radiative lifetimes,
  or oscillator strengths are needed — set the corresponding Functions flag instead
  of CrossSections=1.
- Input data are atomic line lists from a database unsupported by PyExoCross (e.g.
  VAMDC databases, still pending support).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3318
tools:
- name: PyExoCross
  role: Python program that reads ExoMol/HITRAN/HITEMP line lists and computes cross-sections
    with Doppler, Gaussian, Lorentzian, and Voigt profiles via sampling or binned
    methods, driven by an input file and run.py
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: ExoCross
  role: Fortran predecessor application of which PyExoCross is a Python adaptation
  repo: https://github.com/ExoMol/ExoCross
- name: Python
  role: Runtime environment in which PyExoCross is executed (python run.py -p ./input/H2O_exomol.inp);
    tested on Python 3.8–3.12
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:f519ecdd656d3374566d45e88ec23359aa6fa0652d270778b71cd6b0378316d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - ExoCross
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# molecular-line-list-processing

## Summary

Generate temperature- and pressure-dependent molecular cross-sections from large line lists (e.g. the ExoMol POKAZATEL H2O line list) by convolving lines with Doppler, Gaussian, Lorentzian, or Voigt profiles using PyExoCross's sampling or binned methods.

## When to use

Use when a task requires LTE (or non-LTE) absorption/emission cross-sections or stick spectra computed from an ExoMol/HITRAN/HITEMP molecular line list, e.g. reproducing H2O POKAZATEL cross-sections with a Voigt profile in sampling mode.

## When NOT to use

- The goal is to convert line list data between ExoMol and HITRAN formats rather than compute spectra or cross-sections (a separate PyExoCross conversion function).
- Only raw partition functions, specific heats, cooling functions, radiative lifetimes, or oscillator strengths are needed — set the corresponding Functions flag instead of CrossSections=1.
- Input data are atomic line lists from a database unsupported by PyExoCross (e.g. VAMDC databases, still pending support).

## Inputs

- ExoMol states file (e.g. 1H2-16O__POKAZATEL.states.bz2)
- ExoMol transitions files (e.g. 1H2-16O__POKAZATEL.trans.bz2)
- PyExoCross input file (e.g. input/H2O_exomol.inp) with Database/Molecule/Isotopologue/Dataset/SpeciesID block
- ExoMol .def definition file from exomol.com
- Optional reference cross-section file from exomol.com for validation

## Outputs

- Cross-section file (.xsec) with wavenumber grid in the first column and computed cross-section values
- LTE absorption cross-sections on a user-defined wavenumber Range with Npoints grid points

## How to apply

Clone PyExoCross and configure an input file (e.g. input/H2O_exomol.inp) with a Data source block specifying Database=ExoMol, Molecule=H2O, Isotopologue=1H2-16O, Dataset=POKAZATEL, SpeciesID=11, and place the .states.bz2 and .trans.bz2 files under ReadPath in the standard ExoMol directory layout. Set the Functions flag CrossSections=1 (all other function flags 0) and configure the Cross sections section: WnWlUnit in cm-1, a wavenumber Range, an Npoints sampling grid, default Broadeners with Ratios 1.0, and a Voigt Profile option (e.g. SciPyVoigt) with Cutoff(Y/N) and DopplerHWHM/LorentzianHWHM so temperature- and pressure-dependent widths are computed. Parallelise reading of the .trans files via the Cores and chunks block (NCPUtrans, NCPUfiles, ChunkSize) with Device=CPU, then execute 'python run.py -p ./input/H2O_exomol.inp' so the cross_sections routine convolves the line list over the sampling grid. Collect the saved .xsec file from SavePath and compare peak positions and magnitudes against reference cross sections deposited on exomol.com.

## Related tools

- **PyExoCross** (Python program that reads ExoMol/HITRAN/HITEMP line lists and computes cross-sections with Doppler, Gaussian, Lorentzian, and Voigt profiles via sampling or binned methods, driven by an input file and run.py) — https://github.com/Beryl-Jingxin/PyExoCross
- **ExoCross** (Fortran predecessor application of which PyExoCross is a Python adaptation) — https://github.com/ExoMol/ExoCross
- **Python** (Runtime environment in which PyExoCross is executed (python run.py -p ./input/H2O_exomol.inp); tested on Python 3.8–3.12)

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- The output .xsec file exists in SavePath and its first column contains the requested wavenumber grid (per WnWlUnit cm-1 and the Npoints/Range settings).
- Cross-section peak positions and magnitudes are consistent with the reference H2O cross sections deposited on exomol.com.
- The run completes without errors when executed as 'python run.py -p ./input/H2O_exomol.inp' with CrossSections=1 and all other function flags 0.
- Voigt profile settings (Cutoff(Y/N), DopplerHWHM, LorentzianHWHM) produced temperature- and pressure-dependent widths rather than fixed defaults.
- Parallel configuration (NCPUtrans, NCPUfiles, ChunkSize, Device=CPU) processed all POKAZATEL .trans chunks without missing wavenumber ranges.

## Limitations

- macOS MPS GPU mode only uses float32 and lacks precision; use Nvidia GPU (CUDA) or CPU mode for high-precision calculations.
- Databases from VAMDC are not yet supported (marked pending in the README).
- Cutoff(Y/N) and predissociation handling (PredissocXsec(Y/N)) introduce approximation choices — with Cutoff yes, a wing cutoff in cm-1 must be supplied, which truncates far wings of the Voigt profile.
- Users should not change the first column strings in the input file.

## Evidence

- [abstract] PyExoCross calculates cross-sections with four line profiles: Doppler, Gaussian, Lorentzian, and Voigt profiles in both sampling and binned methods: "PyExoCross calculates cross-sections with four line profiles: Doppler, Gaussian, Lorentzian, and Voigt profiles in both sampling and binned methods"
- [readme] run command: "python run.py -p ./input/H2O_exomol.inp"
- [methods] cutoff parameter: "`Cutoff(Y/N)`: If `Cutoff(Y/N)` is yes, you can provide wing cutoff here in unit of cm⁻¹."
- [methods] predissociation cross sections: "`PredissocXsec(Y/N)`: If `PredissocXsec(Y/N)` is yes, predissociation lifetimes will be used or calculated when calculating cross sections with Voigt profile."
- [readme] GPU precision note: [evidence span withheld — anchor status: external_doc]
