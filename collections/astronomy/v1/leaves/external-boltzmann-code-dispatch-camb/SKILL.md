---
name: external-boltzmann-code-dispatch-camb
description: Use when the analysis goal is to compute fσ8 (or other growth quantities)
  at a chosen redshift using CosmoBolognaLib's cosmobl::Cosmology with an external
  Boltzmann code (CAMB) rather than internal approximations.
when_to_use_negative:
- When a modified-gravity or dynamical dark-energy growth model is required — that
  is the scope of the MGCAMB patch to CAMB, not the default cosmobl::Cosmology dispatch
- When the target is measuring clustering statistics (2PCF/3PCF) from a galaxy catalogue
  rather than computing growth-rate quantities from a cosmological model
- When only Python wrappers are acceptable — only a subset of CBL functions has a
  Python wrapper, and the fsigma8 example shown is a C++ program
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CosmoBolognaLib
  role: Provides the cosmobl::Cosmology class whose fsigma8(z, method) method dispatches
    growth computations to external Boltzmann codes
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: CAMB
  role: External Einstein–Boltzmann solver invoked by the fsigma8 method argument
    to compute fσ8
- name: GSL
  role: Required external library for building CosmoBolognaLib C++ code (Numerical
    Recipes substituted by GSL from V3.0)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:9201aa126916c5df431e1846c4b8cd77468d4c3697ea8b9ee31ecd77b4faf132
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - CAMB
  - Cosmo Bologna Lib
  - CosmoBolognaLib
  - GSL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# external-boltzmann-code-dispatch-camb

## Summary

Instantiate a cosmobl::Cosmology object in a C++ example program and call its fsigma8 method with a method argument that dispatches the growth-rate computation to the external CAMB Boltzmann solver, reproducing the Appendix B.1 fσ8(z=1) example of CosmoBolognaLib.

## When to use

Use when the analysis goal is to compute fσ8 (or other growth quantities) at a chosen redshift using CosmoBolognaLib's cosmobl::Cosmology with an external Boltzmann code (CAMB) rather than internal approximations.

## When NOT to use

- When a modified-gravity or dynamical dark-energy growth model is required — that is the scope of the MGCAMB patch to CAMB, not the default cosmobl::Cosmology dispatch
- When the target is measuring clustering statistics (2PCF/3PCF) from a galaxy catalogue rather than computing growth-rate quantities from a cosmological model
- When only Python wrappers are acceptable — only a subset of CBL functions has a Python wrapper, and the fsigma8 example shown is a C++ program

## Inputs

- CosmoBolognaLib source tree downloaded and unpacked into a Cosmo_Bologna_Lib/ folder
- C++ example program (Listing 1 style) constructing a cosmobl::Cosmology object
- External Boltzmann solver CAMB installed and locatable by CosmoBolognaLib
- GSL (and FFTW, OpenMP) libraries available on the system

## Outputs

- Printed f*sigma8 value at z=1 from the example program
- fsigma8_z1_camb.txt containing a single float (expected ~0.4–0.6 for default Planck-like cosmology)

## How to apply

Build the CosmoBolognaLib C++ libraries from source with 'make' in the Cosmo_Bologna_Lib/ folder, ensuring external libraries (GSL, FFTW, OpenMP) are available as required since V3.0 substitutes Numerical Recipes with GSL functions. Write the C++ example program (Listing 1) that constructs a cosmobl::Cosmology object; at creation, default cosmological parameters are used, or parameters can be set with setters such as set_OmegaM, set_sigma8, set_hh. Call the fsigma8 method at z=1 passing the method argument that routes the computation to the external CAMB Boltzmann code, and capture the printed f*sigma8 value. Write the numeric result to fsigma8_z1_camb.txt and validate it as a single float in the physically expected ~0.4–0.6 range for the default Planck-like cosmology.

## Related tools

- **CosmoBolognaLib** (Provides the cosmobl::Cosmology class whose fsigma8(z, method) method dispatches growth computations to external Boltzmann codes) — https://github.com/federicomarulli/CosmoBolognaLib
- **CAMB** (External Einstein–Boltzmann solver invoked by the fsigma8 method argument to compute fσ8)
- **GSL** (Required external library for building CosmoBolognaLib C++ code (Numerical Recipes substituted by GSL from V3.0))

## Examples

```
cosm.set_OmegaM(0.319); cosm.set_sigma8(0.83); cosm.set_hh(0.67); double fs8 = cosm.fsigma8(1., "CAMB");
```

## Evaluation signals

- The output file fsigma8_z1_camb.txt exists and parses as a single float
- The f*sigma8 value at z=1 lies in the physically expected range ~0.4–0.6 for the default Planck-like cosmology
- The value matches the output printed by the paper's Appendix B.1 example program when re-run
- The compilation log shows the CosmoBolognaLib libraries (e.g. make libFUNC) and the example binary built without missing-GSL errors
- The method argument passed to fsigma8 resolves to the CAMB dispatch path (not internal approximations) in the executed program

## Limitations

- When the Cosmology object is created with the default constructor, default cosmological parameters are used; parameter-setting details are deferred to external example codes on the CBL webpage
- No fixed version/release identifier is given for the CBL download in the paper; the archive was hosted on a personal institutional URL without a DOI
- Only a subset of CBL functions has a Python wrapper, so reproducing this C++ example via Python may not be possible
- The CAMB dispatch depends on the external Boltzmann code being correctly installed and findable; installation of CAMB itself is not covered by the CBL instructions

## Evidence
