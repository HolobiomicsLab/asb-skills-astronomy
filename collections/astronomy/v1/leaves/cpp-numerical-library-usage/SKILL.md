---
name: cpp-numerical-library-usage
description: Use when the analysis requires CBL's optimised C++ routines (e.g. parallel
  chain-mesh pair/triplet counting for 2PCF/3PCF) and the task involves handling real
  or simulated astronomical catalogues.
when_to_use_negative:
- Python wrapper coverage matters and the needed CBL function is not wrapped — the
  article states only a subset of CBL currently has a Python wrapper
- The task is unrelated to cosmological catalogue statistics (e.g. an Einstein-Boltzmann
  solver modification) where a purpose-built tool like CAMB/MGCAMB is more appropriate
- Only lightweight generic numerics are needed and a full cosmology library's GSL/FFTW/OpenMP
  dependency chain is unjustified overhead
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0102
tools:
- name: CosmoBolognaLib
  role: The C++ library being compiled and called; provides catalogue handling, chain-mesh
    pair/triplet counting, and 2PCF/3PCF measurement classes
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language in which CBL is written and in which full-coverage driver
    codes must be written
- name: Python
  role: Alternative entry point via compiled SWIG wrappers ('make python'), covering
    a subset of CBL functions
- name: GSL
  role: External numerical dependency used by CBL; Numerical Recipes functions were
    fully substituted by GSL from V3.0
- name: wurlitzer
  role: Captures C-level stdout/stderr (os.dup2) when calling C++ code from Python/Jupyter
    so library messages are visible
  repo: https://github.com/minrk/wurlitzer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:6e30f0609ad00c6cb03eeebead53974f3b73a7dfe442560f3612a69df52b1dfe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - CosmoBolognaLib
  - GSL
  - Python
  - wurlitzer
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cpp-numerical-library-usage

## Summary

Compile and invoke a large C++ numerical library (CosmoBolognaLib) — optionally from Python via wrappers — to run cosmological calculations such as catalogue handling and two-/three-point correlation function measurements.

## When to use

Use when the analysis requires CBL's optimised C++ routines (e.g. parallel chain-mesh pair/triplet counting for 2PCF/3PCF) and the task involves handling real or simulated astronomical catalogues.

## When NOT to use

- Python wrapper coverage matters and the needed CBL function is not wrapped — the article states only a subset of CBL currently has a Python wrapper
- The task is unrelated to cosmological catalogue statistics (e.g. an Einstein-Boltzmann solver modification) where a purpose-built tool like CAMB/MGCAMB is more appropriate
- Only lightweight generic numerics are needed and a full cosmology library's GSL/FFTW/OpenMP dependency chain is unjustified overhead

## Inputs

- Galaxy/halo catalogue files (e.g. cat.dat used in CBL examples)
- Random catalogue constructed with CBL random-catalogue methods
- Parameter file for catalogue cuts (e.g. minimum tracer mass threshold Mmin)

## Outputs

- Compiled CBL shared libraries (libFUNC.so etc.) and Python wrapper modules
- Correlation-function measurement tables (pair/triplet counts, 2PCF/3PCF outputs)
- Cosmological quantities such as fσ8(z, method) from a cosmobl::Cosmology object

## How to apply

Install the external dependencies GSL, FFTW, OpenMP and (for older versions) Numerical Recipes 3rd ed., then download the CosmoBolognaLib.tar archive, unpack it into a Cosmo_Bologna_Lib/ folder, and build with 'make' (or 'make lib*' for individual libraries and 'make python' for the SWIG Python wrappers). Write driver code in C++ (or Python, noting only a subset of CBL has a Python wrapper) that instantiates the relevant classes — e.g. cosmobl::Cosmology to estimate fσ8, or catalogue/two-point/three-point correlation classes fed by vectors of Galaxy objects plus a random catalogue. Decide between the C++ and Python entry points based on coverage: Python wrappers are convenient but incomplete, so statistical cores like the Szapudi-Szalay 3PCF estimator may need direct C++ use. On macOS, compile with 'make SYS=`MAC`'. Validate the build by running an example from CosmoBolognaLib/Examples/ and checking that outputs (e.g. correlation function tables) are produced in the expected format.

## Related tools

- **CosmoBolognaLib** (The C++ library being compiled and called; provides catalogue handling, chain-mesh pair/triplet counting, and 2PCF/3PCF measurement classes) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language in which CBL is written and in which full-coverage driver codes must be written)
- **Python** (Alternative entry point via compiled SWIG wrappers ('make python'), covering a subset of CBL functions)
- **GSL** (External numerical dependency used by CBL; Numerical Recipes functions were fully substituted by GSL from V3.0)
- **wurlitzer** (Captures C-level stdout/stderr (os.dup2) when calling C++ code from Python/Jupyter so library messages are visible) — https://github.com/minrk/wurlitzer

## Examples

```
cd Cosmo_Bologna_Lib/ && make && make python
```

## Evaluation signals

- Build succeeds: 'make' produces the library set and 'make lib*' compiles individual libraries such as libFUNC.so without errors
- Python interoperability check: after 'make python', example codes in CosmoBolognaLib/Examples/ run in both C++ and Python
- Output sanity: a 2PCF example run on the example catalogue produces pair-count and correlation outputs in the expected output/ directory structure
- Statistical plausibility: 3PCF-based measurements yield reduced Q values of order unity and correlation functions with finite values
- Numerical consistency: GSL-backed routines (V3.0+, where Numerical Recipes was fully substituted) reproduce expected fσ8 estimates from cosmobl::Cosmology at a given redshift

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage was deferred to a future version, so some functionality requires C++
- No fixed version/release identifier or archival DOI for the .tar download in the paper; it was hosted on a personal institutional webpage, complicating reproducibility
- Example codes depend on local input files (cat.dat, random/, pairs/, output/ directories) that are not described or hosted with the paper
- Default cosmological parameters are used unless a different constructor is chosen; parameter-setting details live in external example codes on the CBL webpage
- Compilation requires external libraries (GSL, FFTW, OpenMP, formerly Numerical Recipes) which may not be present on the system

## Evidence
