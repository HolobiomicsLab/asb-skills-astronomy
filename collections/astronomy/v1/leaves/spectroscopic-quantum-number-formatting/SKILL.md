---
name: spectroscopic-quantum-number-formatting
description: Use when converting ExoMol line lists (e.g. 24Mg-1H__XAB .states.bz2/.trans.bz2)
  to HITRAN format with PyExoCross and the output must carry molecule-specific quantum
  numbers instead of placeholder values.
when_to_use_negative:
- The target is already a HITRAN/HITEMP file and no reformatting is needed — use direct
  import/export instead of conversion
- You only need spectra or cross-sections computed from the line list; quantum-number
  formatting is specific to data format conversion
- The source is an ExoAtom or ExoMolHR dataset whose quantum number metadata comes
  from NIST/Kurucz or exomol.com/exomolhr/qn/ rather than a standard ExoMol .def file
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: PyExoCross
  role: Runs the ExoMol-to-HITRAN conversion, binding user-supplied quantum number
    labels/formats from .def metadata into fixed-width HITRAN records via the conversion
    (convert_exomol_to_hitran) function
  repo: https://github.com/Beryl-Jingxin/PyExoCross
- name: Python
  role: Language in which PyExoCross is implemented and in which the conversion is
    executed/configured
  repo: https://github.com/Beryl-Jingxin/PyExoCross.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/rasti/rzae016
    title: '<scp>PyExoCross</scp> : a Python program for generating spectra and cross-sections
      from molecular line lists'
  source_content_hash: sha256:1cc3f971775294aaf893e56c502c957863bb6dac327b5f934ba85fc7583ba207
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spectroscopic-quantum-number-formatting

## Summary

Method for supplying quantum number labels and fixed-width formats (e.g. global ElecState, v, Omega; local J, e/f) drawn from ExoMol .def metadata so that PyExoCross can correctly map ExoMol states into HITRAN2004 160-character records during format conversion.

## When to use

Use when converting ExoMol line lists (e.g. 24Mg-1H__XAB .states.bz2/.trans.bz2) to HITRAN format with PyExoCross and the output must carry molecule-specific quantum numbers instead of placeholder values.

## When NOT to use

- The target is already a HITRAN/HITEMP file and no reformatting is needed — use direct import/export instead of conversion
- You only need spectra or cross-sections computed from the line list; quantum-number formatting is specific to data format conversion
- The source is an ExoAtom or ExoMolHR dataset whose quantum number metadata comes from NIST/Kurucz or exomol.com/exomolhr/qn/ rather than a standard ExoMol .def file

## Inputs

- ExoMol states file (e.g. 24Mg-1H__XAB.states.bz2) and transitions file (24Mg-1H__XAB.trans.bz2) following the Molecule/Isotopologue/Dataset layout
- ExoMol definition file (.def, .def., or .adef.) from exomol.com specifying quantum number labels
- PyExoCross input file (.inp) with Database=ExoMol, ConversionFormat=HITRAN, ConversionFrequncyRange, and QNs labels/formats

## Outputs

- HITRAN-format line list with fixed-width HITRAN2004 160-character records covering the requested wavenumber range
- Converted records carrying the supplied global and local quantum numbers per line

## How to apply

First read the dataset's .def metadata from exomol.com to identify the global (e.g. ElecState, v, Omega) and local (e.g. J, e/f) quantum number labels appropriate to the isotopologue/dataset layout under ReadPath. In the PyExoCross input file set Database=ExoMol, ConversionFormat=HITRAN, and ConversionFrequncyRange (e.g. 0–30000 cm-1), then supply the quantum number labels and their formats so states-file columns are bound to the correct HITRAN record fields. Optionally enable ConvUncFilter(Y/N)=Y with a maximum uncertainty (e.g. 0.01 cm-1) and ConvThreshold(Y/N)=Y with a minimum intensity (e.g. 1e-30 cm/molecule) to restrict converted lines. Rationale: HITRAN2004 records are fixed-width 160-character lines, so each quantum number must have an explicit width/format or the conversion cannot serialize states correctly.

## Related tools

- **PyExoCross** (Runs the ExoMol-to-HITRAN conversion, binding user-supplied quantum number labels/formats from .def metadata into fixed-width HITRAN records via the conversion (convert_exomol_to_hitran) function) — https://github.com/Beryl-Jingxin/PyExoCross
- **Python** (Language in which PyExoCross is implemented and in which the conversion is executed/configured) — https://github.com/Beryl-Jingxin/PyExoCross.git

## Examples

```
python run.py -p ./input/24Mg-1H__XAB_exomol_to_hitran.inp
```

## Evaluation signals

- Output file exists and every line is a fixed-width HITRAN2004 160-character record
- All converted lines fall within the requested ConversionFrequncyRange (e.g. 0–30000 cm-1)
- Every output line satisfies the configured uncertainty filter (≤ 0.01 cm-1) and intensity threshold (≥ 1e-30 cm/molecule) when those filters are enabled
- Quantum number columns in the output match the labels/formats declared from the XAB .def metadata (global: ElecState, v, Omega; local: J, e/f)

## Limitations

- Quantum number labels are dataset-specific; using labels that do not match the .def metadata for the isotopologue will misalign record fields
- Molecule-specific quantum number metadata must be fetched externally (exomol.com .def files, or exomol.com/exomolhr/qn/ for ExoMolHR; HITRAN SpeciesID from HITRANOnline Isotopologue Metadata)
- No changelog was found for the repository, so version-dependent behavior of conversion options is not documented

## Evidence

- [methods] PyExoCross can convert data format between the ExoMol (ExoMol, ExoMolHR, and ExoAtom databases) and the HITRAN (HITRAN and HITEMP database) formats.: "PyExoCross can convert data format between the ExoMol (ExoMol, ExoMolHR, and ExoAtom databases) and the HITRAN (HITRAN and HITEMP database) formats."
- [methods] The definition files `.def`, `.def.`, and `.adef.` of ExoMol and ExoAtom databases (available at exomol.com): [evidence span withheld — anchor status: not_found]
- [methods] `ConvUncFilter(Y/N)`: If it is yes, the value is the maximum uncertainty you require, in unit of cm⁻¹.: "`ConvUncFilter(Y/N)`: If it is yes, the value is the maximum uncertainty you require, in unit of cm⁻¹."
- [methods] `ConvThreshold(Y/N)`: If it is yes, the value is the minimum intensity you require, in unit of cm/molecule.: "`ConvThreshold(Y/N)`: If it is yes, the value is the minimum intensity you require, in unit of cm/molecule."
- [abstract] PyExoCross supports importing and exporting line lists in the ExoMol and HITRAN/HITEMP formats. PyExoCross also provides conversion between the ExoMol and HITRAN data formats: "PyExoCross supports importing and exporting line lists in the ExoMol and HITRAN/HITEMP formats. PyExoCross also provides conversion between the ExoMol and HITRAN data formats"
- [methods] The quantum number metadata for ExoMolHR database can be accessed from https://www.exomol.com/exomolhr/qn/: "The quantum number metadata for ExoMolHR database can be accessed from [https://www.exomol.com/exomolhr/qn/](https://www.exomol.com/exomolhr/qn/)"
