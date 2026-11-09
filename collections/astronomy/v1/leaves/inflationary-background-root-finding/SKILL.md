---
name: inflationary-background-root-finding
description: Use when integrating an inflationary background with pyoscode and you
  must locate the moment epsilon = 1 (end of inflation) before setting Bunch-Davies
  initial conditions and evaluating Mukhanov-Sasaki modes at horizon exit.
when_to_use_negative:
- The background is given analytically or epsilon = 1 occurs outside the integration
  interval [ti, tf], so no root exists for the event function to find
- The ODE is a generic oscillatory problem (e.g. Airy or burst examples) with no cosmological
  slow-roll variable — use pyoscode.solve/solve_fn directly instead
tools:
- name: pyoscode
  role: Integrates the inflationary background equations and performs root finding
    on the endinfl event function to detect the end of inflation; also solves the
    subsequent Mukhanov-Sasaki mode equations
  repo: github.com/fruzsinaagocs/oscode
- name: numpy
  role: Array evaluation of the background variables and mode solutions over the k
    range when assembling the power spectrum
  repo: https://pypi.org/project/numpy/
- name: matplotlib
  role: Plots the computed scalar primordial power spectrum resulting from the horizon-exit
    evaluation
  repo: https://pypi.org/project/matplotlib/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:2b97e73aeb0c9aecf615cb27dac09e383067acef3619d97201fcdae7c32e4531
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matplotlib
  - numpy
  - pyoscode
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# inflationary-background-root-finding

## Summary

Detects the end of an inflationary cosmological background (computed with pyoscode) by root finding on the slow-roll condition epsilon = 1, supplying the horizon-crossing evaluation point and timing needed to assemble the scalar primordial power spectrum.

## When to use

Use when integrating an inflationary background with pyoscode and you must locate the moment epsilon = 1 (end of inflation) before setting Bunch-Davies initial conditions and evaluating Mukhanov-Sasaki modes at horizon exit.

## When NOT to use

- The background is given analytically or epsilon = 1 occurs outside the integration interval [ti, tf], so no root exists for the event function to find
- The ODE is a generic oscillatory problem (e.g. Airy or burst examples) with no cosmological slow-roll variable — use pyoscode.solve/solve_fn directly instead

## Inputs

- Inflationary background ODE system (phi(t), dphi(t)) defined as functions for pyoscode.solve_fn
- Slow-roll parameter epsilon(t) computed from the background variables
- Event function endinfl(t, y) that crosses zero at epsilon = 1
- Perturbation mode wavenumbers k over the desired spectrum range

## Outputs

- End-of-inflation time (root of the epsilon = 1 condition) and total e-folds N
- Background solution (phi, dphi) evaluated up to the end of inflation
- Per-mode Bunch-Davies initial conditions bdic(k, phi, dphi, ddphi, N) anchored to the detected horizon exit

## How to apply

Integrate the inflationary background equations (the phi/dphi system of the cosmology example) with pyoscode's solver, using pyoscode.solve_fn when the frequency and friction terms are supplied as functions. Define an event function endinfl(t, y) that crosses zero when the slow-roll parameter epsilon reaches 1; the solver's root finding locates the end of inflation. Use the detected horizon-crossing time to evaluate each perturbation mode k (initialized from the Bunch-Davies vacuum via bdic(k, phi, dphi, ddphi, N)) at horizon exit, then assemble the scalar primordial power spectrum over the k range. The rationale is that the epsilon = 1 crossing fixes the e-folds N and the time at which modes exit the horizon, so any error in the root location propagates directly into the spectrum.

## Related tools

- **pyoscode** (Integrates the inflationary background equations and performs root finding on the endinfl event function to detect the end of inflation; also solves the subsequent Mukhanov-Sasaki mode equations) — github.com/fruzsinaagocs/oscode
- **numpy** (Array evaluation of the background variables and mode solutions over the k range when assembling the power spectrum) — https://pypi.org/project/numpy/
- **matplotlib** (Plots the computed scalar primordial power spectrum resulting from the horizon-exit evaluation) — https://pypi.org/project/matplotlib/

## Examples

```
def endinfl(t, y):
    """ Crosses zero when inflation ends """
```

## Evaluation signals

- The detected end-of-inflation time corresponds to the first zero crossing of endinfl(t, y), i.e. epsilon = 1 within the solver's tolerance
- The generated spectrum figure (spectra.gif) exists and the computed scalar primordial power spectrum qualitatively matches the result reported in the paper's applications section
- Per-mode initial conditions bdic(k, phi, dphi, ddphi, N) use the background values and e-fold count at/derived from the epsilon = 1 crossing
- Mode solutions evaluated at horizon exit produce a spectrum over the full requested k range without NaNs or failed integrations

## Limitations

- The WKB approximation embedded in oscode is valid only when the frequency changes slowly relative to the integration timescale, so highly non-adiabatic background regimes may reduce solver efficiency/accuracy
- The root finder detects inflation's end only if the epsilon = 1 crossing is inside the requested integration interval; a bad choice of ti/tf misses the event
- The spectral comparison is qualitative in the paper's workflow; quantitative validation against an external reference is left to the user

## Evidence

- [results] Integrate cosmological background equations and detect end of inflation via root finding (epsilon = 1): [evidence span withheld — anchor status: not_found]
- [results] endinfl event function: "def endinfl(t, y):
    """ Crosses zero when inflation ends """"
- [results] Bunch-Davies initial conditions per mode: "def bdic(k, phi, dphi, ddphi, N):
    """ Defines the Bunch-Davies vacuum solution
    for a given perturbation mode """"
- [results] pyoscode function-based solver call: [evidence span withheld — anchor status: not_found]
