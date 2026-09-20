---
name: dewiggled-power-spectrum-bao-damping
description: Reference for modelling redshift-space 2PCF xi(s) from a de-wiggled BAO
  power spectrum in CosmoBolognaLib, when redshift errors, RSD (growth rate f), and
  BAO damping must be propagated into the monopole model.
when_to_use_negative:
- When you only need to measure, not model, the 2PCF from a galaxy catalogue — the
  measurement workflow (catalogue + random catalogue + TwoPointCorrelation class)
  is a separate CBL path
- When the analysis requires a fully non-linear BAO treatment without Gaussian damping
  approximations — this model explicitly uses Sigma_NL Gaussian damping of the wiggles
- When working in real space or ignoring redshift errors — the model's P(k,mu) explicitly
  includes the redshift-error damping term sigma and RSD terms
edam_operation: http://edamontology.org/operation_2426
tools:
- name: CosmoBolognaLib
  role: Provides the cosmology, modelling, and two-point-correlation classes used
    to build P(k,mu), the de-wiggled BAO spectrum, and the model xi(s)
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: Supplies the numerical quadrature and special functions used for mu-integration
    and the Fourier anti-transform, replacing Numerical Recipes from V3.0
- name: C++
  role: Language in which the CBL modelling libraries are written and compiled (e.g.
    make libFUNC)
- name: Python
  role: High-level interface via SWIG wrappers (full coverage of CBL functions only
    from V1.1 onward and still a subset in later versions)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:5a514f106b3d8f2156a339379d601b695b4b61ef55765c51d49e4937fc657637
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# dewiggled-power-spectrum-bao-damping

## Summary

Reconstruct the CosmoBolognaLib model for the angle-averaged two-point correlation function of cosmic tracers in redshift space, using the de-wiggled BAO power spectrum with Sigma_NL Gaussian damping and a Gaussian redshift-error damping term in P(k,mu).

## When to use

Reference for modelling redshift-space 2PCF xi(s) from a de-wiggled BAO power spectrum in CosmoBolognaLib, when redshift errors, RSD (growth rate f), and BAO damping must be propagated into the monopole model.

## When NOT to use

- When you only need to measure, not model, the 2PCF from a galaxy catalogue — the measurement workflow (catalogue + random catalogue + TwoPointCorrelation class) is a separate CBL path
- When the analysis requires a fully non-linear BAO treatment without Gaussian damping approximations — this model explicitly uses Sigma_NL Gaussian damping of the wiggles
- When working in real space or ignoring redshift errors — the model's P(k,mu) explicitly includes the redshift-error damping term sigma and RSD terms

## Inputs

- Cosmology object parameters (Omega_B, Omega_M, Omega_DE, sigma8, h)
- Real-space linear/non-linear dark matter power spectrum P_DM(k)
- Linear growth rate f of cosmic structures
- Gaussian redshift-error damping scale sigma and BAO damping scale Sigma_NL
- Separation range s for the model 2PCF

## Outputs

- Redshift-space power spectrum P(k,mu) (Eq. 16)
- mu-integrated monopole, quadrupole and hexadecapole terms (Eqs. 20-22)
- De-wiggled BAO power spectrum (Eq. 23)
- model_2pcf.dat with s and xi(s) columns for the test cosmology

## How to apply

Instantiate a CBL cosmology::Cosmology object and set background parameters (Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67) via setters rather than constructor defaults. Evaluate the redshift-space power spectrum P(k,mu) (Eq. 16), combining the real-space linear/non-linear matter power spectrum with the redshift-space-distortion terms involving the linear growth rate f and the Gaussian redshift-error damping scale sigma. Numerically integrate over mu to obtain the monopole, quadrupole and hexadecapole terms (Eqs. 20-22) using the GSL quadrature routines that replaced Numerical Recipes from V3.0 onward. Build the de-wiggled spectrum (Eq. 23) by applying the Sigma_NL Gaussian damping specifically to the BAO wiggles of the model P(k), then Fourier anti-transform the de-wiggled, mu-integrated power spectrum to obtain model xi(s), writing results to model_2pcf.dat with s, xi(s) columns.

## Related tools

- **CosmoBolognaLib** (Provides the cosmology, modelling, and two-point-correlation classes used to build P(k,mu), the de-wiggled BAO spectrum, and the model xi(s)) — https://github.com/federicomarulli/CosmoBolognaLib
- **GSL** (Supplies the numerical quadrature and special functions used for mu-integration and the Fourier anti-transform, replacing Numerical Recipes from V3.0)
- **C++** (Language in which the CBL modelling libraries are written and compiled (e.g. make libFUNC))
- **Python** (High-level interface via SWIG wrappers (full coverage of CBL functions only from V1.1 onward and still a subset in later versions))

## Examples

```
cd CosmoBolognaLib/ && make libFUNC && make python; cosm.set_OmegaM(0.319); cosm.set_OmegaDE(0.681); cosm.set_sigma8(0.83); cosm.set_hh(0.67)
```

## Evaluation signals

- model_2pcf.dat exists and contains two columns, s and xi(s), over the chosen separation range
- The mu-integrated monopole/quadrupole/hexadecapole integrals (Eqs. 20-22) converge under the linked GSL quadrature routines
- The de-wiggled spectrum (Eq. 23) shows damped BAO wiggles relative to the undamped model P(k), controlled by Sigma_NL
- Varying sigma (redshift-error damping) and f (growth rate) in P(k,mu) changes xi(s) in the expected direction (stronger damping broadens the BAO peak; larger f boosts RSD distortions)
- Compilation of the modelling libraries (make libFUNC / make python) completes without missing GSL symbols, confirming the V3.0+ GSL-based build

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage was deferred to a future version, so some modelling classes may require C++
- No fixed version/release identifier was given for the CBL download in the paper; only a .tar archive at a personal institutional URL, and the repository contains no changelog
- Cosmological parameters in the paper's examples rely on constructor defaults unless explicitly set via a different constructor or setters; parameter-setting details are deferred to external example codes on the CBL webpage
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) that are not described or hosted with the paper

## Evidence
