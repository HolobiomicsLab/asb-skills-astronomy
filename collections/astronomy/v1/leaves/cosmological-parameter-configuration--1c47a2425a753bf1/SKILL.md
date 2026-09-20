---
name: cosmological-parameter-configuration
description: 'Use when a CBL modelling task requires a non-default cosmology: the
  example constructors apply Planck-like defaults, but mass function, bias, growth-rate
  (fσ8) or 2PCF computations need explicitly set parameters.'
when_to_use_negative:
- When the Planck-like default parameters set at object creation already match the
  desired cosmology and no custom constructor is needed
- When the task does not depend on cosmological parameters (e.g. purely geometric
  catalogue handling or pair counting with fixed input catalogues)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: Cosmo Bologna Lib
  role: Provides the cosmobl::Cosmology class whose constructor and set_* setters
    define the cosmological parameters for all downstream computations
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the Cosmology object is instantiated and its setter methods
    called
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: Python
  role: High-level alternative for configuring the Cosmology object via the CBL wrapping
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: CAMB
  role: External boltzmann solver dispatched by the fsigma8 method once the cosmology
    is configured
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:ca5184ae2653cfdb1711e5e7fb70df9c877aabb7a5f23af2635ae09ea3dbb84e
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
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cosmological-parameter-configuration

## Summary

Configure the cosmobl::Cosmology object in CosmoBolognaLib with explicit cosmological parameters (e.g. Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67) using class setters or a custom constructor, so that downstream computations (fσ8, mass functions, power spectra, 2PCF models) run in a controlled, reproducible cosmology.

## When to use

Use when a CBL modelling task requires a non-default cosmology: the example constructors apply Planck-like defaults, but mass function, bias, growth-rate (fσ8) or 2PCF computations need explicitly set parameters.

## When NOT to use

- When the Planck-like default parameters set at object creation already match the desired cosmology and no custom constructor is needed
- When the task does not depend on cosmological parameters (e.g. purely geometric catalogue handling or pair counting with fixed input catalogues)

## Inputs

- Cosmological parameter values (Omega_B, Omega_M, Omega_DE, sigma8, h, or a full parameter set passed to an alternative constructor)
- Target redshift(s) for derived quantities (e.g. z=1 for fσ8)
- Choice of external boltzmann solver dispatch (e.g. CAMB) when required by the method argument

## Outputs

- A configured cosmobl::Cosmology object usable by CBL modelling classes (mass functions, bias, P(k,µ), 2PCF)
- Derived quantities evaluated in the fixed cosmology (e.g. fσ8(z=1) ≈ 0.4–0.6 for the Planck-like default cosmology)

## How to apply

Instantiate a cosmobl::Cosmology object in C++ (or via the SWIG Python wrapper); by default the object at creation uses Planck-like default cosmological parameters. To impose a specific cosmology, either use a different constructor or call the class setters individually, e.g. cosm.set_OmegaB(0.049); cosm.set_OmegaM(0.319); cosm.set_OmegaDE(0.681); cosm.set_sigma8(0.83); cosm.set_hh(0.67), as shown in the paper's HOD example. These setters must be applied BEFORE calling dependent methods such as fsigma8(z, method) (which can dispatch to the external CAMB boltzmann solver), the sigma(M) variance computation feeding the nine halo mass function models, or the redshift-space P(k,µ) construction. Rationale: parameter-setting details are deferred to external example codes, so leaving defaults in place silently changes all derived clustering predictions; fixing the cosmology once at object level guarantees consistency across the transfer-function machinery, growth rate and bias parameterisations.

## Related tools

- **Cosmo Bologna Lib** (Provides the cosmobl::Cosmology class whose constructor and set_* setters define the cosmological parameters for all downstream computations) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the Cosmology object is instantiated and its setter methods called) — https://github.com/federicomarulli/CosmoBolognaLib
- **Python** (High-level alternative for configuring the Cosmology object via the CBL wrapping) — https://github.com/federicomarulli/CosmoBolognaLib
- **CAMB** (External boltzmann solver dispatched by the fsigma8 method once the cosmology is configured)

## Examples

```
cosmobl::Cosmology cosm; cosm.set_OmegaB(0.049); cosm.set_OmegaM(0.319); cosm.set_OmegaDE(0.681); cosm.set_sigma8(0.83); cosm.set_hh(0.67);
```

## Evaluation signals

- Verify the setters were called before any derived computation: re-run with different Omega_M/sigma8 values and confirm outputs (e.g. dn/dM, b(z), fσ8) change accordingly, proving defaults were overridden
- For the fσ8(z=1) example, check the output parses as a single float in the physically expected range ~0.4–0.6 for the default Planck cosmology
- Cross-check configured parameters against the reference set Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67 used in the paper's HOD example
- Confirm the script runs without error and outputs match reference CBL built-in model outputs for the fixed cosmology (output_matches_reference)

## Limitations

- The Cosmology object defaults to Planck-like parameters at creation; cosmological parameter-setting details are deferred to external example codes on the CBL webpage, so users may unknowingly run with defaults
- Only a subset of CBL has a Python wrapper; some parameter-dependent methods may only be reachable from C++
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) not described or hosted, complicating end-to-end reproduction

## Evidence

- [discussion] Default cosmological parameters are applied at object creation; alternatives via constructor or setters: [evidence span withheld — anchor status: not_found]
