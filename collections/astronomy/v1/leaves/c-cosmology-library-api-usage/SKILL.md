---
name: c-cosmology-library-api-usage
description: Use when a task requires computing a cosmological quantity (e.g. fσ8
  at z=1) by calling the cosmobl::Cosmology class from a compiled C++ program that
  dispatches to an external Boltzmann solver like CAMB.
when_to_use_negative:
- The task is measuring or modelling the two-point correlation function from a galaxy
  catalogue (cat.dat, random catalogues, pairs/) — that workflow uses the Catalogue
  and Two Point Correlation classes, not the Cosmology API
- You need to run CAMB directly with modified-growth (mu/gamma, Q/R) parametrizations
  — that requires the MGCAMB patch to CAMB, not the CBL Cosmology dispatch
- You need a Python-only interface — only a subset of CBL functions has a Python wrapper,
  so a full Cosmology-class dispatch may require the C++ path
edam_operation: http://edamontology.org/operation_0279
tools:
- name: CosmoBolognaLib
  role: Provides the cosmobl::Cosmology class whose fsigma8(z, method) method is the
    API entry point for the computation
  repo: github.com/federicomarulli/CosmoBolognaLib
- name: CAMB
  role: External Einstein-Boltzmann solver dispatched by the fsigma8 method argument
    to compute the growth quantity
- name: GSL
  role: Required external library that must be installed before compiling the CBL
    libraries
- name: C++
  role: Language in which the example program is written and compiled against the
    CBL libraries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:b1ae49db1d20e485d46c62357ccec46271f4a542f6e94d46697449d45ea95359
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C++
  - CAMB
  - Cosmo Bologna Lib
  - CosmoBolognaLib
  - GSL
  edam_operation: http://edamontology.org/operation_0279
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# c++-cosmology-library-api-usage

## Summary

Pattern for including the CosmoBolognaLib C++ headers in a compiled program, constructing a cosmobl::Cosmology object, and calling its methods (e.g. fsigma8(z, method)) that dispatch computations to external Boltzmann solvers such as CAMB.

## When to use

Use when a task requires computing a cosmological quantity (e.g. fσ8 at z=1) by calling the cosmobl::Cosmology class from a compiled C++ program that dispatches to an external Boltzmann solver like CAMB.

## When NOT to use

- The task is measuring or modelling the two-point correlation function from a galaxy catalogue (cat.dat, random catalogues, pairs/) — that workflow uses the Catalogue and Two Point Correlation classes, not the Cosmology API
- You need to run CAMB directly with modified-growth (mu/gamma, Q/R) parametrizations — that requires the MGCAMB patch to CAMB, not the CBL Cosmology dispatch
- You need a Python-only interface — only a subset of CBL functions has a Python wrapper, so a full Cosmology-class dispatch may require the C++ path

## Inputs

- Compiled CosmoBolognaLib shared libraries (built with 'make' after installing GSL, FFTW, OpenMP)
- C++ source program including cosmobl::Cosmology headers (modeled on Listing 1)
- Redshift value (z=1) and method string selecting the external Boltzmann solver (CAMB)
- Optional cosmological parameters set via setters (Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67) or Planck-like defaults at object creation

## Outputs

- Printed f*sigma8 value at z=1 from the example executable
- Single-float output file (fsigma8_z1_camb.txt) containing the computed fσ8(z=1)

## How to apply

First build the CBL libraries from source: install the external dependencies (GSL, FFTW, OpenMP), download the CosmoBolognaLib source from github.com/federicomarulli/CosmoBolognaLib, and run 'make' in the top folder (or 'make libFUNC' to compile individual libraries). Then write a C++ example program modeled on Listing 1: construct the cosmobl::Cosmology object — by default it uses Planck-like default cosmological parameters at creation, or you can call setters such as cosm.set_OmegaB(0.049), cosm.set_OmegaM(0.319), cosm.set_OmegaDE(0.681), cosm.set_sigma8(0.83), cosm.set_hh(0.67) before the computation. Call the fsigma8 method at redshift z=1 with the method argument set to dispatch to CAMB, capture the printed value, and write it as a single float to an output file (e.g. fsigma8_z1_camb.txt). The rationale is that the paper explicitly demonstrates the Cosmology class as the API entry point for cosmological computations, with external Boltzmann codes handled through the method argument rather than reimplemented in CBL.

## Related tools

- **CosmoBolognaLib** (Provides the cosmobl::Cosmology class whose fsigma8(z, method) method is the API entry point for the computation) — github.com/federicomarulli/CosmoBolognaLib
- **CAMB** (External Einstein-Boltzmann solver dispatched by the fsigma8 method argument to compute the growth quantity)
- **GSL** (Required external library that must be installed before compiling the CBL libraries)
- **C++** (Language in which the example program is written and compiled against the CBL libraries)

## Examples

```
cosmobl::Cosmology cosm; cosm.set_sigma8(0.83); cosm.set_hh(0.67); double fs8 = cosm.fsigma8(1., "CAMB");
```

## Evaluation signals

- The output file fsigma8_z1_camb.txt exists and parses as a single float
- The computed fσ8(z=1) falls in the physically expected range ~0.4–0.6 for the default Planck cosmology
- The value is consistent with the example executable's printed output (reproducible across runs)
- The CBL build completed without missing-library errors (GSL/FFTW/OpenMP present), confirming the library path is correct
- When non-default parameters are used, the setters (e.g. Omega_M=0.319, sigma8=0.83, hh=0.67) were applied before the fsigma8 call, not after

## Limitations

- When the Cosmology object is created, default cosmological parameters are used; parameter-setting details are deferred to external example codes on the CBL webpage, so non-default runs require reading the alternative constructor examples
- No fixed version/release identifier is given for the CBL download; only a .tar archive (formerly at a personal institutional URL, now github), so results may vary across library versions
- Only a subset of the CBL has a Python wrapper, with full coverage deferred to a future version — the C++ path is required for full API access

## Evidence

- [discussion] Cosmology class fsigma8 dispatch to CAMB: "The following code shows how to use the class cosmobl::Cosmology to estimate the value of fσ 8 at z = 1."
- [discussion] Default cosmological parameters at creation: "When the object is created, at line 13, default cosmological pa- rameters are used. Alternatively, the cosmological parameters can be set directly by using a different constructor"
- [results] Parameter setters used in examples: "cosm.set_OmegaB(0.049)
cosm.set_OmegaM(0.319)
cosm.set_OmegaDE(0.681)
cosm.set_sigma8(0.83)
cosm.set_hh(0.67)"
- [readme] Example codes demonstrating C++ or Python usage: "The example codes in CosmoBolognaLib/Examples/ show how to use the CosmoBolognaLib in either C++ or Python codes."
- [discussion] External library prerequisites for building: "if not already present in the system, install the follow- ing external libraries: GSL (GNU Scientific Library)6, FFTW7, OpenMP8 and Numerical Recipes (Third Edi- tion)9"
- [discussion] Python wrapper coverage is partial: "Only a subset of the CBL has currently a Python wrapper. We plan to extend this to all the CBL functions in a future version of the libraries."
