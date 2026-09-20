---
name: fortran-python-wrapped-package-installation
description: Use when a task requires installing a Python scientific package with
  compiled Fortran 90 core routines (rabacus-style f2py wrapping), where the build
  needs a Fortran compiler and OpenMP flags before any Python-level import will succeed.
when_to_use_negative:
- When the target package is pure Python with no compiled Fortran core — no F90 setup
  or f2py compiler detection is needed.
- When the Fortran build must use a compiler other than gfortran or ifort; the article
  states other compilers require the Manual Install route, not the single pip command.
- When the task requires metal line cooling or non-equilibrium solvers — the article
  notes these are planned future improvements, not present in the installed package.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: rabacus
  role: Package being installed; Fortran 90 core wrapped in Python via f2py, OpenMP-enabled
    build
  repo: https://bitbucket.org/galtay/rabacus
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines in Python during installation; used with
    --help-fcompiler to list detected compilers
  repo: http://pythonhosted.org//rabacus
- name: Quantities
  role: Required prerequisite providing units for all rabacus variables
  repo: https://www.github.com/python-quantities/python-quantities
- name: gfortran
  role: Recommended Fortran compiler set via the F90 environment variable for the
    OpenMP build
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:9a8b1707805d6943f0dcee2c3947dd0d3a098928abaf41697fd2286a865bbe8f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Numpy f2py
  - Python
  - Quantities
  - gfortran
  - rabacus
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fortran-python-wrapped-package-installation

## Summary

Installs a Python package whose core is Fortran 90 compiled and wrapped via Numpy f2py (e.g. rabacus), by setting the F90 environment variable to gfortran or ifort and using a single pip call to obtain an OpenMP-enabled build, then verifying the import.

## When to use

Use when a task requires installing a Python scientific package with compiled Fortran 90 core routines (rabacus-style f2py wrapping), where the build needs a Fortran compiler and OpenMP flags before any Python-level import will succeed.

## When NOT to use

- When the target package is pure Python with no compiled Fortran core — no F90 setup or f2py compiler detection is needed.
- When the Fortran build must use a compiler other than gfortran or ifort; the article states other compilers require the Manual Install route, not the single pip command.
- When the task requires metal line cooling or non-equilibrium solvers — the article notes these are planned future improvements, not present in the installed package.

## Inputs

- Python environment with pip
- Fortran 90 compiler in path (gfortran or ifort)
- rabacus tar.gz from PyPI (https://pypi.python.org/pypi/rabacus) for manual install

## Outputs

- OpenMP-enabled rabacus installation importable as 'rabacus' (import rabacus as ra)
- rabacus_install_files.txt listing installed files for easy uninstall (manual install)

## How to apply

First install the Python prerequisites (scipy, numpy >= 1.7, quantities) via pip or APT, and ensure a Fortran compiler such as gfortran is in the executable path. Set the environment variable F90 (export F90=gfortran or export F90=ifort) so the build system knows which Fortran 90 compiler to use for the OpenMP-enabled f2py compilation. Then run a single 'pip install rabacus' (or 'pip install --user rabacus' without root) and confirm success by the 'Successfully installed rabacus' message. If the single-command install fails, fall back to the manual route: list compilers with 'f2py -c --help-fcompiler', edit setup.py's f90_flags and omp_lib variables, and run 'python setup.py build --fcompiler=intelem' followed by 'python setup.py install --record rabacus_install_files.txt'. Finally verify the install by importing the package (import rabacus as ra) in a Python/IPython session and exercising a documented call such as computing a He I photo-heating rate from the metagalactic background.

## Related tools

- **rabacus** (Package being installed; Fortran 90 core wrapped in Python via f2py, OpenMP-enabled build) — https://bitbucket.org/galtay/rabacus
- **Numpy f2py** (Wraps the Fortran 90 core routines in Python during installation; used with --help-fcompiler to list detected compilers) — http://pythonhosted.org//rabacus
- **Quantities** (Required prerequisite providing units for all rabacus variables) — https://www.github.com/python-quantities/python-quantities
- **gfortran** (Recommended Fortran compiler set via the F90 environment variable for the OpenMP build)

## Examples

```
export F90=gfortran; sudo pip install rabacus; python -c "import rabacus as ra"
```

## Evaluation signals

- pip reports 'Successfully installed rabacus' followed by 'Cleaning up...'.
- 'import rabacus as ra' succeeds in a fresh Python/IPython session without build or import errors.
- The F90 environment variable was set before install and the build used gfortran or ifort, yielding an OpenMP-enabled binary.
- After install, a documented call (e.g. computing the He I photo-heating rate at redshift 3.0 from the metagalactic background) executes and returns a rate with units from the Quantities package.
- Prerequisite imports (scipy, numpy as np, quantities as pq) all succeed before attempting the rabacus install.

## Limitations

- The single pip install path is only tested with the intel (ifort) and gnu gfortran compilers; other compilers must follow the Manual Install route with edited f90_flags and omp_lib.
- Quantities package test coverage is incomplete and it is 'not suggested for mission-critical applications' per its README.
- Numpy version 1.7 or later is required.
- Metal line cooling and non-equilibrium solvers are not yet included in the package (planned future improvements).
- Without root access, use 'pip install --user' which installs into ~/.local.

## Evidence

- [intro] we implemented the core functionality of rabacus in Fortran 90 and wrapped the resulting code in Python: [evidence span withheld — anchor status: not_found]
