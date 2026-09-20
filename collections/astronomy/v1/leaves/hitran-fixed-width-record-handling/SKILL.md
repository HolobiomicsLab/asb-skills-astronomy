---
name: hitran-fixed-width-record-handling
description: Use when an ExoMol (or ExoMolHR/ExoAtom) line list must be emitted in
  HITRAN/HITEMP format over a defined wavenumber range, e.g. ConversionFormat=HITRAN
  with ConversionFrequncyRange 0–30000 cm-1.
when_to_use_negative:
- The line list is already in HITRAN/HITEMP format — use import/export directly instead
  of conversion.
- The target database is ExoMol-family (ExoMol, ExoMolHR, ExoAtom) output rather than
  HITRAN format; conversion is defined between ExoMol and HITRAN formats only.
- You need spectra or cross-sections (Doppler/Gaussian/Lorentzian/Voigt profiles)
  rather than a format-converted line list.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: PyExoCross
  role: Runs the ExoMol-to-HITRAN format conversion, applying ConvUncFilter and ConvThreshold
    filters and quantum number labels from the .def file
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Runtime environment for executing PyExoCross (tested on Python 3.8–3.12, dependencies
    in requirements.txt)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:79f4b2626abcdfcbabaaf8e4b8ebf5e7680742acca19dafec712e4819c5735dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Beryl-Jingxin/PyExoCross
  tools:
  - PyExoCross
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hitran-fixed-width-record-handling

## Summary

Converts ExoMol-format molecular line lists (e.g. 24Mg-1H__XAB .states.bz2/.trans.bz2) into HITRAN-format output with fixed-width HITRAN2004 160-character records, using PyExoCross's conversion function with uncertainty and intensity filters.

## When to use

Use when an ExoMol (or ExoMolHR/ExoAtom) line list must be emitted in HITRAN/HITEMP format over a defined wavenumber range, e.g. ConversionFormat=HITRAN with ConversionFrequncyRange 0–30000 cm-1.

## When NOT to use

- The line list is already in HITRAN/HITEMP format — use import/export directly instead of conversion.
- The target database is ExoMol-family (ExoMol, ExoMolHR, ExoAtom) output rather than HITRAN format; conversion is defined between ExoMol and HITRAN formats only.
- You need spectra or cross-sections (Doppler/Gaussian/Lorentzian/Voigt profiles) rather than a format-converted line list.

## Inputs

- ExoMol states file (e.g. 24Mg-1H__XAB.states.bz2) located under ReadPath in Molecule/Isotopologue/Dataset layout
- ExoMol transitions file (e.g. 24Mg-1H__XAB.trans.bz2)
- ExoMol .def definition file providing quantum number labels and formats (global: ElecState, v, Omega; local: J, e/f)
- PyExoCross input file (.inp) with Database=ExoMol, ConversionFormat=HITRAN, ConversionFrequncyRange, ConvUncFilter, ConvThreshold

## Outputs

- HITRAN-format line list file with fixed-width HITRAN2004 160-character records
- Records restricted to the requested wavenumber range and passing the 0.01 cm-1 uncertainty and 1e-30 cm/molecule intensity thresholds

## How to apply

Locate the ExoMol input files under ReadPath following the Molecule/Isotopologue/Dataset layout (e.g. 24Mg-1H__XAB.states.bz2 and 24Mg-1H__XAB.trans.bz2) and set Database=ExoMol with ConversionFormat=HITRAN in the PyExoCross input file. Set the ConversionFrequncyRange (e.g. 0–30000 cm-1) and enable the conversion filters: ConvUncFilter(Y/N)=Y with a maximum uncertainty of 0.01 cm-1, and ConvThreshold(Y/N)=Y with a minimum intensity of 1e-30 cm/molecule. Supply the quantum number labels and formats (e.g. ElecState, v, Omega as global; J, e/f as local) drawn from the dataset's ExoMol .def metadata, since HITRAN records require these quantum assignments. Run the conversion (convert_exomol_to_hitran / conversion function) via `python run.py -p input_filepath`. Validate that the output file exists, contains fixed-width HITRAN2004 160-character records over the requested range, and that every line satisfies the uncertainty and intensity thresholds.

## Related tools

- **PyExoCross** (Runs the ExoMol-to-HITRAN format conversion, applying ConvUncFilter and ConvThreshold filters and quantum number labels from the .def file) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Runtime environment for executing PyExoCross (tested on Python 3.8–3.12, dependencies in requirements.txt))

## Examples

```
python run.py -p ./input/H2O_exomol.inp
```

## Evaluation signals

- Output file exists and each record is exactly 160 characters in fixed-width HITRAN2004 layout.
- All output lines fall within the requested ConversionFrequncyRange (e.g. 0–30000 cm-1).
- Every emitted line has uncertainty ≤ 0.01 cm-1 (ConvUncFilter) and intensity ≥ 1e-30 cm/molecule (ConvThreshold).
- Quantum number labels in the output (global: ElecState, v, Omega; local: J, e/f) match the dataset's .def metadata.

## Limitations

- Conversion filters must be explicitly enabled: ConvUncFilter(Y/N) and ConvThreshold(Y/N) only apply when set to Y with the respective maximum uncertainty (cm-1) and minimum intensity (cm/molecule) values.
- Quantum number metadata differs per database — ExoMol .def/.def./.adef. files are needed, while ExoMolHR quantum number metadata comes from exomol.com/exomolhr/qn/ and HITRAN SpeciesID comes from HITRANOnline isotopologue metadata.
- No changelog is available in the repository's discussion section, so version-specific conversion behavior changes cannot be traced from the repo.

## Evidence

- [methods] PyExoCross conversion filters: "`ConvUncFilter(Y/N)`: If it is yes, the value is the maximum uncertainty you require, in unit of cm⁻¹."
- [methods] PyExoCross conversion threshold: "`ConvThreshold(Y/N)`: If it is yes, the value is the minimum intensity you require, in unit of cm/molecule."
- [methods] Conversion scope: "*PyExoCross* can convert data format between the ExoMol (ExoMol, ExoMolHR, and ExoAtom databases) and the HITRAN (HITRAN and HITEMP database) formats."
- [readme] README conversion claim: [evidence span withheld — anchor status: external_doc]
