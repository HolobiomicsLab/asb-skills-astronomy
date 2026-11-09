---
name: hot-jupiter-atmospheric-configuration
description: Use when you need to reproduce the ExoLyn baseline hot-Jupiter cloud-structure
  solve, or measure wall-clock runtime of a single relaxation run to validate the
  few-seconds performance claim.
when_to_use_negative:
- The goal is spectral retrieval or opacity computation — that requires coupling ExoLyn
  to optool or petitRADTRANS rather than a single relaxation solve.
- The target planet is a sub-Neptune or self-luminous planet with a different cloud
  regime (e.g. SiO2-dominated or Fe clouds), not the hot-Jupiter baseline configuration.
- A new reaction network is being added — parameters.txt and gibbs.txt must be edited
  first (Gibbs data from janaf.nist.gov) before a runtime benchmark is meaningful.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ExoLyn
  role: 1D cloud model that solves cloud-particle and vapor transport with self-consistent
    condensation rates; produces the hot-Jupiter layered cloud structure
  repo: github.com/helonghuangastro/exolyn
- name: Fortran (sol.f90)
  role: Custom linear-system solver compiled via build.sh; much faster than the numpy
    solver, enabling the few-second runtime
- name: Python
  role: Runs relaxation.py and the surrounding scripts (parameters.py, read.py, init.py);
    used to time the solve with time.perf_counter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:864b45c88990c2827605fe05b3ff59c510f6d301fd58a13449abbf452ec0ac99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoLyn
  - Fortran (sol.f90)
  - Python
  - bash
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hot-jupiter-atmospheric-configuration

## Summary

Benchmark configuration and runtime check for reproducing ExoLyn's hot-Jupiter layered cloud structure (magnesiumsilicate cloud over an iron cloud) and verifying the paper's claim that a full relaxation solve completes in only a few seconds.

## When to use

Use when you need to reproduce the ExoLyn baseline hot-Jupiter cloud-structure solve, or measure wall-clock runtime of a single relaxation run to validate the few-seconds performance claim.

## When NOT to use

- The goal is spectral retrieval or opacity computation — that requires coupling ExoLyn to optool or petitRADTRANS rather than a single relaxation solve.
- The target planet is a sub-Neptune or self-luminous planet with a different cloud regime (e.g. SiO2-dominated or Fe clouds), not the hot-Jupiter baseline configuration.
- A new reaction network is being added — parameters.txt and gibbs.txt must be edited first (Gibbs data from janaf.nist.gov) before a runtime benchmark is meaningful.

## Inputs

- ExoLyn repository (github.com/helonghuangastro/exolyn) with parameters.txt, reactions.txt, gibbs.txt
- Hot-Jupiter atmospheric configuration (layered magnesiumsilicate cloud over an iron cloud) in parameters.txt
- Fortran source sol.f90 and build script build.sh

## Outputs

- runtime.json containing runtime_seconds (measured wall-clock duration of one full relaxation solve)
- Converged cloud-structure solution showing magnesiumsilicate-dominated clouds layered on top of an iron cloud

## How to apply

Clone the ExoLyn repository and compile the custom Fortran linear-system solver (sol.f90) with 'bash build.sh'; this solver replaces the slower numpy solver for the relaxation step. Load the hot-Jupiter configuration by having parameters.py read parameters.txt, and ingest the chemical data (reactions.txt and gibbs.txt) via read.py; Gibbs formation energies come from janaf.nist.gov. Initialize the atmospheric domain size and initial equilibrium concentrations with init.py, then run one full relaxation solve with 'python3 relaxation.py' while wrapping the execution in a wall-clock timer (e.g. Python time.perf_counter or /usr/bin/time) so only the solve duration is captured. Write the measured elapsed time into runtime.json as runtime_seconds and inspect the output layered structure for a magnesiumsilicate-dominated cloud on top of an iron cloud. The rationale: ExoLyn solves the transport equation of cloud particles and vapor under self-consistently calculated condensation rates, so a converged run on the hot-Jupiter case should both match the layered composition finding and finish on the order of a few seconds.

## Related tools

- **ExoLyn** (1D cloud model that solves cloud-particle and vapor transport with self-consistent condensation rates; produces the hot-Jupiter layered cloud structure) — github.com/helonghuangastro/exolyn
- **Fortran (sol.f90)** (Custom linear-system solver compiled via build.sh; much faster than the numpy solver, enabling the few-second runtime)
- **Python** (Runs relaxation.py and the surrounding scripts (parameters.py, read.py, init.py); used to time the solve with time.perf_counter)

## Examples

```
bash build.sh; python3 -c "import time;t=time.perf_counter();exec(open('relaxation.py').read());print(time.perf_counter()-t)" > runtime_seconds
```

## Evaluation signals

- runtime_seconds recorded in runtime.json is on the order of a few seconds, matching the paper's claim that results 'can be obtained with ExoLyn in only a few seconds'
- The converged solution shows a layered structure with a magnesiumsilicate-dominated cloud on top of an iron cloud
- The Fortran solver was compiled successfully via 'bash build.sh' (a numpy-solver fallback would violate the runtime expectation)
- The solve was timed so that only the relaxation.py execution was captured, excluding compilation and file-loading overhead

## Limitations

- Runtime benchmark is specific to the hot-Jupiter configuration; other planet types (metal-rich, C-rich, sub-Neptune, self-luminous) may have different solve times and cloud structures.
- The custom sol.f90 solver must be compiled before running; the code will not achieve the few-second runtime using the slower numpy solver.
- Changing reactions requires edits to both parameters.txt and gibbs.txt, with Gibbs formation energy data sourced from janaf.nist.gov; the README notes '#' is a comment character in parameters.txt.

## Evidence

- [abstract] runtime claim: "This finding is consistent with more complex cloud formation models but can be obtained with ExoLyn in only a few seconds"
- [abstract] layered structure: "the compositional structure of clouds in hot Jupiter planets’ atmospheres is layered with a cloud dominated by magnesiumsilicates on top of an iron cloud"
- [methods] compile: "After downloading the code, you can compile the fortran part by running `bash build.sh`"
- [methods] run: "After compilation, you can run the code by `python3 relaxation.py`"
- [methods] solver speed: "sol.f90** is a custom fortran program to numerically solve linear systems. It's much faster than the numpy solver"
- [readme] initialization: "**init.py** initializes the code, finding appropriate domain size and then calculating the initial equilibrium concentrations."
