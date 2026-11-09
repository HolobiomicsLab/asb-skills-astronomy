---
name: namelist-input-file-configuration
description: Use when preparing an input file for molscat, bound or field variants
  (e.g. molscat-basic1.input, molscat-basic_Rb2.input, field-Mg_NH.input) that specifies
  basis, potential expansion, field ranges or energies via &INPUT/&BASIS/&POTL namelists.
when_to_use_negative:
- The collision type or potential is not expressible via the built-in namelist options
  and requires writing/altering a plug-in basis-set suite or VINIT/VSTAR routine rather
  than only input data
- The task is post-processing of existing S-matrix files (differential cross sections,
  RESFIT resonance fitting, SAVER accumulation) rather than configuring a new calculation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: molscat 2025.0
  role: Consumes the namelist input file on unit 5 to run coupled-channel scattering
    calculations
  repo: https://github.com/molscat/molscat
- name: bound 2025.0
  role: Reads the same namelist blocks to run bound-state energy calculations with
    bound-state boundary conditions
  repo: https://github.com/molscat/molscat
- name: field 2025.0
  role: Reads namelist input to locate external fields at which a bound state exists
    at a specified energy (e.g. field-Mg_NH.input)
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Namelist syntax of the input file and the language of the plug-in routines
    referenced by negative NTERM terms
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_005
  source_content_hash: sha256:9e3d7035191c4d47444a79e9b0115dcec095cb71cca7897d6b5bd2b52721cb9a
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - Fortran
  - LAPACK
  - bound 2025.0
  - field 2025.0
  - molscat
  - molscat 2025.0
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# namelist-input-file-configution

## Summary

Configure MOLSCAT/BOUND/FIELD 2025.0 runs by authoring Fortran namelist input blocks (&INPUT, &BASIS, &POTL) read from unit 5, specifying collision type, basis suite, interaction potential (including NTERM/NPOWER/A terms and negative-NTERM plug-in terms), reduced mass, propagator (e.g. LDMD/AIRY log-derivative with TOLHIL), energies and fields.

## When to use

Use when preparing an input file for molscat, bound or field variants (e.g. molscat-basic1.input, molscat-basic_Rb2.input, field-Mg_NH.input) that specifies basis, potential expansion, field ranges or energies via &INPUT/&BASIS/&POTL namelists.

## When NOT to use

- The collision type or potential is not expressible via the built-in namelist options and requires writing/altering a plug-in basis-set suite or VINIT/VSTAR routine rather than only input data
- The task is post-processing of existing S-matrix files (differential cross sections, RESFIT resonance fitting, SAVER accumulation) rather than configuring a new calculation

## Inputs

- Namelist input file (&INPUT, &BASIS, &POTL blocks) read on unit 5
- Example templates from examples/input/ (molscat-basic1.input, molscat-basic_Rb2.input, field-Mg_NH.input)
- Potential data files from the data/ directory (e.g. Mg + NH, H2 + H2) referenced by &POTL

## Outputs

- Executable run producing program output (e.g. field-Mg_NH.output in examples/output/)
- Printed POTENL potential expansion listing LAMBDA, NTERM, NPOWER and A values
- Per-field-point scattering observables (s-wave scattering length, effective range) or located bound-state crossing fields

## How to apply

Write the input file read on unit 5 as several blocks of namelist data: &INPUT for propagator, energies, fields and convergence (e.g. LDMD/AIRY log-derivative propagation with TOLHIL=0 for full convergence in field-Mg_NH), &BASIS for the interaction type (e.g. closed-shell atom + triplet-Sigma diatom in a magnetic field, or alkali-metal atom + alkali-metal atom with hyperfine structure), and &POTL for the potential expansion (e.g. MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6, A=1.0,-2.0,-0.2 for the molscat-basic1 model). For terms that should come from a user routine, set the NTERM element negative so POTENL calls VSTAR(R, I, P(I)) and the plug-in returns the radial coefficient. For field scans such as molscat-basic_Rb2, specify the magnetic-field sweep (800–900 G in 1 G steps at 100 nK collision energy) so the code reports s-wave scattering length and effective range per field point. Validate by running the executable and checking the printed POTENL expansion or located crossing fields against the documented reference values.

## Related tools

- **molscat 2025.0** (Consumes the namelist input file on unit 5 to run coupled-channel scattering calculations) — https://github.com/molscat/molscat
- **bound 2025.0** (Reads the same namelist blocks to run bound-state energy calculations with bound-state boundary conditions) — https://github.com/molscat/molscat
- **field 2025.0** (Reads namelist input to locate external fields at which a bound state exists at a specified energy (e.g. field-Mg_NH.input)) — https://github.com/molscat/molscat
- **Fortran** (Namelist syntax of the input file and the language of the plug-in routines referenced by negative NTERM terms) — https://github.com/molscat/molscat

## Examples

```
make field-Mg_NH && ./field-Mg_NH < examples/input/field-Mg_NH.input > examples/output/field-Mg_NH.output
```

## Evaluation signals

- The program parses and runs the input file without error and writes expected output (e.g. field-Mg_NH.output in examples/output/)
- The printed POTENL expansion matches the intended parameters exactly (e.g. MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6, A=1.0,-2.0,-0.2 as documented in section 3.8.1)
- For field-Mg_NH with TOLHIL=0, the located crossing fields reproduce the reference threshold crossing near 387.28 G
- For molscat-basic_Rb2, all 101 field points (800–900 G) complete and the scattering-length curve shows the pole near 850 G and narrow resonance near 820 G

## Limitations

- Simple potentials must be specified in input data; more complicated ones need plug-in routines — negative NTERM terms route to VSTAR and P(I) is zeroed in input
- Version 2025.0 uses 2022 CODATA fundamental constants and allocatable arrays; input files from older releases may need adjustment
- Executables require BLAS/LAPACK routines; if optimised libraries are absent, Netlib object files must be supplied via LIBUTILS before the namelist-driven run can execute

## Evidence

- [other] IF (NTERM(I).LT.0) CALL VSTAR(R, I, P(I)), i.e. VSTAR is called with the distance R and the term index I and returns the radial potential coefficient in P(I): [evidence span withheld — anchor status: not_found]
- [other] Run field-Mg_NH using LDMD/AIRY log-derivative propagation with TOLHIL=0 (full convergence): [evidence span withheld — anchor status: not_found]
