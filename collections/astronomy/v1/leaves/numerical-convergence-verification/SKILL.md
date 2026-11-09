---
name: numerical-convergence-verification
description: Reference for validating that a FIELD (or BOUND) close-coupling run converged
  on a bound state at threshold and that reported external-field values are correct
  before reusing them downstream.
when_to_use_negative:
- The goal is scattering observables (S matrices, cross sections) rather than bound-state
  energies or Feshbach resonance positions — use molscat instead of field
- The run has not finished or the solver did not print a convergence message; extrapolating
  field values from an unconverged search is invalid
- The physical system or basis-set suite differs from the one the executable was built
  for (e.g. using a non-alkali-metal dimer field variant), so documented reference
  values do not apply
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3315
tools:
- name: field 2025.0
  role: Runs the close-coupling bound-state search over magnetic field whose convergence
    messages and threshold-crossing values are verified
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines (e.g. dsyevr) whose correct linkage affects the numerical
    accuracy of the bound-state energies being checked
- name: BLAS
  role: Basic linear algebra subroutines (dgemm, dgemv, dsymm, etc.) that must be
    supplied in LIBS or LIBUTILS for reproducible convergence results
- name: GNUmakefile (GNU make)
  role: Build system used to compile the field- variant executable (e.g. 'make field-Rb2')
    before the verification run
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:5a9774dd4d586aaec8e10f4e7a7667185ed68f0fc3e4f4691892c860667e442d
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - GNUmakefile (GNU make)
  - LAPACK
  - field 2025.0
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# numerical-convergence-verification

## Summary

Verify that coupled-channel bound-state/field calculations have genuinely converged by checking 'CONVERGED ON STATE NUMBER' messages in FIELD output and cross-checking threshold-crossing field values against documented reference results (e.g. the 85Rb2 Feshbach resonances at 851.8761789, 819.5625939 and 810.9471815 Gauss).

## When to use

Reference for validating that a FIELD (or BOUND) close-coupling run converged on a bound state at threshold and that reported external-field values are correct before reusing them downstream.

## When NOT to use

- The goal is scattering observables (S matrices, cross sections) rather than bound-state energies or Feshbach resonance positions — use molscat instead of field
- The run has not finished or the solver did not print a convergence message; extrapolating field values from an unconverged search is invalid
- The physical system or basis-set suite differs from the one the executable was built for (e.g. using a non-alkali-metal dimer field variant), so documented reference values do not apply

## Inputs

- FIELD executable variant with special-purpose basis-set suite (e.g. field-Rb2 built via GNUmakefile)
- Namelist input file (e.g. field-basic_Rb2.input) with &INPUT, &BASIS, &POTL blocks, threshold energy 0 and magnetic-field search range
- BLAS/LAPACK libraries (optimised in LIBS, or Netlib object files in LIBUTILS)
- Reference documentation values (sections 3.10.2 and 14.3.2) for expected crossing fields

## Outputs

- Output file in examples/output/ (e.g. field-basic_Rb2.out) containing 'CONVERGED ON STATE NUMBER' messages
- List of converged threshold-crossing field values in Gauss (851.8761789, 819.5625939, 810.9471815 for 85Rb2)
- Convergence status confirming the bound-state search terminated successfully

## How to apply

Build the required FIELD variant executable (e.g. 'make field-Rb2') after adapting GNUmakefile to supply optimised BLAS/LAPACK in LIBS or Netlib object files in LIBUTILS. Run the executable on the example input (e.g. field-basic_Rb2.input), which performs a close-coupling bound-state calculation at threshold energy 0 with BCYOMX at its default 0.0, searching over magnetic field as the external variable. Inspect the output in examples/output/ (field-basic_Rb2.out) for lines of the form 'CONVERGED ON STATE NUMBER 1469' / '1470' and extract the Gauss values at which bound states cross the threshold (851.8761789 and 819.5625939 Gauss in the sample run). Judge success by: (1) the solver reports convergence rather than exhaustion of states or non-convergence warnings, (2) all three crossings match 851.8761789, 819.5625939 and 810.9471815 Gauss as given in sections 3.10.2 and 14.3.2 of the documentation, and (3) BLAS/LAPACK were linked correctly so numerical results are reproducible.

## Related tools

- **field 2025.0** (Runs the close-coupling bound-state search over magnetic field whose convergence messages and threshold-crossing values are verified) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines (e.g. dsyevr) whose correct linkage affects the numerical accuracy of the bound-state energies being checked)
- **BLAS** (Basic linear algebra subroutines (dgemm, dgemv, dsymm, etc.) that must be supplied in LIBS or LIBUTILS for reproducible convergence results)
- **GNUmakefile (GNU make)** (Build system used to compile the field- variant executable (e.g. 'make field-Rb2') before the verification run) — https://github.com/molscat/molscat

## Examples

```
make field-Rb2 && ./field-Rb2 < examples/input/field-basic_Rb2.input > examples/output/field-basic_Rb2.out; grep 'CONVERGED ON STATE NUMBER' examples/output/field-basic_Rb2.out
```

## Evaluation signals

- Output contains explicit 'CONVERGED ON STATE NUMBER' lines (e.g. state 1469 and 1470) rather than a non-convergence condition
- Reported crossing fields match documentation values exactly: 851.8761789, 819.5625939 and 810.9471815 Gauss for 85Rb2
- BCYOMX is at its default 0.0 so the search is at threshold energy 0 as documented
- Build completed with the required LAPACK routines (dgesv, dlamch, dsyevr, dsyr2k, dsytrf, dsytri, ilaenv, lsame) resolved from LIBS or LIBUTILS
- Re-running the same input reproduces the same field values to full printed precision

## Limitations

- Verification is tied to documented example systems (85Rb2 with the alkali-metal atom pair basis suite); other systems need their own reference values
- The default GNUmakefile compiles gfortran at optimisation level 0 with traceback on; different compilers or optimisation could alter reproducibility unless validated
- Convergence messages only certify that the specified bound state was found at the requested energy; physical accuracy still depends on the interaction potential and basis set supplied via &POTL and &BASIS

## Evidence
