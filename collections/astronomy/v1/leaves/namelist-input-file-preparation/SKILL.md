---
name: namelist-input-file-preparation
description: Use when setting up a new MOLSCAT/BOUND/FIELD run (e.g. atom + linear
  rigid rotor at 1250 cm-1, a two-channel LJ Feshbach scan, or a FIELD threshold search
  over magnetic field) and you must author or adapt a namelist input file.
when_to_use_negative:
- The calculation requires a collision type or external field not covered by the built-in
  interaction types or plug-in basis-set suites — a plug-in routine/basis-set suite
  must be coded instead of only editing namelists
- The task is post-processing of an existing S-matrix file (differential cross sections
  via DCS, SBE transport cross sections, or RESFIT resonance fitting) rather than
  setting up a new calculation
- The goal is compiling the code (adapting GNUmakefile LIBS/LIBUTILS and Compiler
  variables) rather than defining a calculation's physics
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_3318
tools:
- name: molscat
  role: Target executable that consumes the namelist input file on unit 5 for non-reactive
    quantum scattering (coupled-channel) calculations
  repo: https://github.com/molscat/molscat
- name: bound
  role: Target executable consuming namelist input for bound-state energy calculations
    with bound-state boundary conditions
  repo: https://github.com/molscat/molscat
- name: field
  role: Target executable consuming namelist input to locate external fields (e.g.
    magnetic Z field in Gauss) at which a bound state crosses a specified threshold
    energy
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Language of the namelist input format and the programs themselves (near-standard
    Fortran 77 with some Fortran 90 features)
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_001
  source_content_hash: sha256:133a694dfb4ae98b1cc43f607c47ff4dcd7213dd9bf4e94eed9b7f45fa62a2f4
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - Fortran
  - GNUmakefile
  - GNUmakefile (GNU make)
  - LAPACK
  - bound
  - field
  - molscat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# namelist-input-file-preparation

## Summary

Prepares the Fortran namelist input files that drive MOLSCAT, BOUND and FIELD close-coupling calculations: the file read on unit 5 consists of several blocks of namelist data (&INPUT, &BASIS, &POTL) that specify interaction type, basis set, interaction potential, reduced mass, propagator, collision energies and external fields.

## When to use

Use when setting up a new MOLSCAT/BOUND/FIELD run (e.g. atom + linear rigid rotor at 1250 cm-1, a two-channel LJ Feshbach scan, or a FIELD threshold search over magnetic field) and you must author or adapt a namelist input file.

## When NOT to use

- The calculation requires a collision type or external field not covered by the built-in interaction types or plug-in basis-set suites — a plug-in routine/basis-set suite must be coded instead of only editing namelists
- The task is post-processing of an existing S-matrix file (differential cross sections via DCS, SBE transport cross sections, or RESFIT resonance fitting) rather than setting up a new calculation
- The goal is compiling the code (adapting GNUmakefile LIBS/LIBUTILS and Compiler variables) rather than defining a calculation's physics

## Inputs

- Namelist input file (e.g. molscat-basic1.input, molscat-2chan-LJ.input, field-basic_Rb2.input from examples/input/) with &INPUT, &BASIS and &POTL blocks
- Physical parameters: reduced mass (URED), collision energy (e.g. 1250 cm-1), JTOT range, threshold energy and external magnetic field search ranges
- Potential specification: built-in forms (e.g. Lennard-Jones 12-6 + R^-6) or data files (e.g. Mg + NH, H2 + H2 potential data in data/)

## Outputs

- Complete namelist input file ready to be piped to a molscat-/bound-/field- executable on unit 5
- Corresponding program output (e.g. molscat-basic1.output, field-basic_Rb2.out) writable to examples/output/ for comparison with reference files

## How to apply

Author the input as several blocks of namelist data read on unit 5 (standard input): &INPUT for run control (interaction type such as ITYP=1 atom + linear molecule, reduced mass URED, propagator, collision energy e.g. 1250 cm-1, JTOT range such as 10 to 20 step 10, or the threshold/field search settings like BCYOMX for FIELD), &BASIS for the basis-set specification, and &POTL for the interaction potential (e.g. Lennard-Jones 12-6 plus R^-6 terms, or two LJ channels separated by 100 cm-1 with a coupling 10x weaker than the channel potentials). Choose values consistent with the executable variant being run (molscat-, bound- or field- prefixed, e.g. molscat-H2_H2 built via 'make molscat-H2_H2'). Validate by running the executable and diffing the generated output against the reference file in examples/output/, which is the acceptance criterion used in the documentation (e.g. sections 3.8.1, 14.1.12).

## Related tools

- **molscat** (Target executable that consumes the namelist input file on unit 5 for non-reactive quantum scattering (coupled-channel) calculations) — https://github.com/molscat/molscat
- **bound** (Target executable consuming namelist input for bound-state energy calculations with bound-state boundary conditions) — https://github.com/molscat/molscat
- **field** (Target executable consuming namelist input to locate external fields (e.g. magnetic Z field in Gauss) at which a bound state crosses a specified threshold energy) — https://github.com/molscat/molscat
- **Fortran** (Language of the namelist input format and the programs themselves (near-standard Fortran 77 with some Fortran 90 features))

## Examples

```
./molscat-basic < examples/input/molscat-basic1.input > molscat-basic1.output
```

## Evaluation signals

- The program runs to completion without namelist read errors and reports convergence (e.g. 'CONVERGED ON STATE NUMBER 1469' at 851.8761789 Gauss for field-basic_Rb2)
- Generated output matches the associated reference file in examples/output/ to the printed precision (acceptance criterion in sections 14.1.12 / 14.3.2)
- For scattering runs, extracted cross sections (SIG(1,1)=1.81057 A^2, SIG(2,1)=3.722327E-02 A^2 at 1250 cm-1) match section 3.8.1 reference values
- For resonance scans, the eigenphase sum shows a Feshbach resonance just below 42.84 cm-1 for molscat-2chan-LJ.input
- All required namelist blocks (&INPUT, &BASIS, &POTL) are present and the specified interaction type, reduced mass, propagator, energies and fields are consistent with the chosen executable variant

## Limitations

- The namelist format and available keywords are tied to the program version; version 2025.0 changed internals (fixed storage array X replaced by allocatable arrays, constants updated to 2022 CODATA), so old inputs may need checking
- Simple interaction potentials may be specified in input data, but more complicated ones require plug-in routines beyond the namelist blocks
- Default optimisation level is 0 with traceback switched on in GNUmakefile; input files alone do not control compiler/library setup (LIBS vs LIBUTILS for BLAS/LAPACK)

## Evidence

- [methods] specify interaction type, basis set, interaction potential, reduced mass, propagator, energies and fields via namelist input blocks: [evidence span withheld — anchor status: not_found]
