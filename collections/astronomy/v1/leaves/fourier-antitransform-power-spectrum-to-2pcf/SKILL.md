---
name: fourier-antitransform-power-spectrum-to-2pcf
description: Use when you need to model the angle-averaged redshift-space 2PCF xi(s)
  of cosmic tracers from a P(k,mu) model, including Gaussian redshift-error damping
  and de-wiggled BAO treatment.
when_to_use_negative:
- The analysis goal is measuring the 2PCF from an observed galaxy catalogue (pair
  counting) rather than modelling xi(s) from a power spectrum — use the CBL TwoPointCorrelation
  measurement classes instead
- The task requires only real-space clustering models without redshift-space-distortion
  or redshift-error damping terms
- Only a subset of CBL has a Python wrapper, so if the required modelling classes
  are unwrapped, work in C++ directly
edam_operation: http://edamontology.org/operation_2464
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: CosmoBolognaLib
  role: Provides the modelling classes for the redshift-space P(k,mu), the mu-integrated
    multipole terms, the de-wiggled BAO power spectrum, and the Fourier anti-transform
    to xi(s)
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: Numerical quadrature over mu and other numerical routines used in place of
    Numerical Recipes from V3.0 onward
- name: C++
  role: Language in which CBL is fully written; used to build and drive the 2PCF modelling
    workflow
- name: Python
  role: Optional high-level interface via SWIG wrappers (partial coverage of CBL functions)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:e08baacc15b2c0e354b74aec9f81f668704b4c98f0f77a4286d567b77a5e17a8
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
  edam_operation: http://edamontology.org/operation_2464
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fourier-antitransform-power-spectrum-to-2pcf

## Summary

Reconstructs the model redshift-space two-point correlation function xi(s) by Fourier anti-transforming the angle-dependent redshift-space power spectrum P(k,mu) of CosmoBolognaLib, using its mu-integrated monopole, quadrupole and hexadecapole terms and the de-wiggled (Sigma_NL-damped) BAO power spectrum.

## When to use

Use when you need to model the angle-averaged redshift-space 2PCF xi(s) of cosmic tracers from a P(k,mu) model, including Gaussian redshift-error damping and de-wiggled BAO treatment.

## When NOT to use

- The analysis goal is measuring the 2PCF from an observed galaxy catalogue (pair counting) rather than modelling xi(s) from a power spectrum — use the CBL TwoPointCorrelation measurement classes instead
- The task requires only real-space clustering models without redshift-space-distortion or redshift-error damping terms
- Only a subset of CBL has a Python wrapper, so if the required modelling classes are unwrapped, work in C++ directly

## Inputs

- CBL Cosmology object with test-cosmology background parameters set (Omega_M, Omega_DE, sigma8, h)
- Real-space linear/non-linear matter power spectrum P_DM(k)
- Linear growth rate of cosmic structures f
- Gaussian redshift-error damping scale sigma (and Sigma_NL for BAO de-wiggling)

## Outputs

- Angle-averaged redshift-space 2PCF model xi(s)
- Mu-integrated multipole terms (monopole, quadrupole, hexadecapole; Eqs. 20-22)
- De-wiggled BAO power spectrum (Eq. 23)
- model_2pcf.dat file with s, xi(s) columns over the chosen separation range

## How to apply

Instantiate a CBL Cosmology object and set the background parameters via the class setters (e.g. Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67) instead of constructor defaults. Evaluate the redshift-space power spectrum P(k,mu) (Eq. 16), combining the real-space matter power spectrum with the linear growth rate f and the Gaussian damping scale sigma for redshift errors. Compute the mu-integrated monopole, quadrupole and hexadecapole terms (Eqs. 20-22) by numerical integration over mu using the GSL quadrature routines linked since V3.0, when Numerical Recipes was fully substituted by GSL functions. Apply the Sigma_NL Gaussian damping to the BAO wiggles to construct the de-wiggled power spectrum (Eq. 23), then obtain xi(s) as the Fourier anti-transform of the de-wiggled, mu-integrated power spectrum using the CBL two-point-correlation modelling classes improved from V4.1 onward. Write the resulting curves to model_2pcf.dat and verify it contains the expected s, xi(s) columns over the chosen separation range.

## Related tools

- **CosmoBolognaLib** (Provides the modelling classes for the redshift-space P(k,mu), the mu-integrated multipole terms, the de-wiggled BAO power spectrum, and the Fourier anti-transform to xi(s)) — https://github.com/federicomarulli/CosmoBolognaLib
- **GSL** (Numerical quadrature over mu and other numerical routines used in place of Numerical Recipes from V3.0 onward)
- **C++** (Language in which CBL is fully written; used to build and drive the 2PCF modelling workflow)
- **Python** (Optional high-level interface via SWIG wrappers (partial coverage of CBL functions))

## Examples

```
make libFUNC && make python
```

## Evaluation signals

- model_2pcf.dat contains the expected two columns s and xi(s) over the chosen separation range
- The mu-integrated terms (Eqs. 20-22) integrate to the correct multipole moments under the chosen GSL quadrature scheme
- The de-wiggled power spectrum (Eq. 23) shows BAO wiggles smoothly damped by the Sigma_NL Gaussian factor compared to the wiggly model P(k)
- The computed xi(s) is consistent with the Fourier anti-transform of the input P(k,mu) and reflects the set cosmology (f and sigma8 dependence)
- Build succeeds via 'make libFUNC' and the modelling libraries, confirming GSL-linked routines are available

## Limitations

- Only a subset of the CBL has currently a Python wrapper; full coverage was deferred to a future version, so some modelling classes may require C++
- No fixed version/release identifier or archival DOI for the CBL download was given in the paper; version-dependent behaviour matters (V3.0 GSL substitution, V4.1 modelling improvements)
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) not described or hosted in the paper

## Evidence

- [methods] The redshift-space 2PCF model is built from a redshift-space power spectrum in which P_DM(k) is the dark matter power spectrum, f is the linear growth rate: [evidence span withheld — anchor status: not_found]
- [methods] the angle-averaged ξ(s) follows as the Fourier anti-transform of this P(k,µ): [evidence span withheld — anchor status: not_found]
