---
name: primordial-power-spectrum-computation
description: Use when an inflationary background solution is available and the goal
  is the scalar primordial power spectrum P(k) over a range of perturbation modes
  k, requiring oscillatory mode integration from Bunch-Davies initial conditions.
when_to_use_negative:
- The equation of interest is not of oscillatory oscillator form x''(t) + g(t)x'(t)
  + w^2(t)x(t) = 0 with computable frequency w(t) and friction g(t), e.g. general
  nonlinear ODEs outside oscode's scope.
- The frequency w(t) varies rapidly everywhere in the integration range, invalidating
  the WKB approximation that oscode relies on for efficiency.
- Only the background evolution is needed; the perturbation-mode calculation and horizon-exit
  evaluation are unnecessary overhead.
edam_operation: http://edamontology.org/operation_2426
edam_topics: []
tools:
- name: pyoscode
  role: Integrates the Mukhanov-Sasaki perturbation mode equations and background
    equations, switching between Runge-Kutta and WKB steps
  repo: https://github.com/fruzsinaagocs/oscode
- name: numpy
  role: Array handling for k modes, solver inputs, and spectrum assembly (installed
    automatically with pyoscode)
- name: matplotlib
  role: Plotting the computed scalar primordial power spectrum
- name: scipy
  role: Root finding for detecting the end of inflation (epsilon = 1)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:7a8ca756d35676173a8e54de23be25738c6d404864b5dba6ccba360bcfc50566
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fruzsinaagocs/oscode
  tools:
  - Python
  - matplotlib
  - numpy
  - pyoscode
  - scipy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# primordial-power-spectrum-computation

## Summary

Computation of the scalar primordial power spectrum by integrating the highly oscillatory Mukhanov-Sasaki perturbation modes with pyoscode, using Bunch-Davies vacuum initial conditions and inflation-end detection via the slow-roll condition epsilon = 1.

## When to use

Use when an inflationary background solution is available and the goal is the scalar primordial power spectrum P(k) over a range of perturbation modes k, requiring oscillatory mode integration from Bunch-Davies initial conditions.

## When NOT to use

- The equation of interest is not of oscillatory oscillator form x''(t) + g(t)x'(t) + w^2(t)x(t) = 0 with computable frequency w(t) and friction g(t), e.g. general nonlinear ODEs outside oscode's scope.
- The frequency w(t) varies rapidly everywhere in the integration range, invalidating the WKB approximation that oscode relies on for efficiency.
- Only the background evolution is needed; the perturbation-mode calculation and horizon-exit evaluation are unnecessary overhead.

## Inputs

- Inflationary background solution (phi, dphi as functions of time) integrated with pyoscode
- Set of perturbation mode wavenumbers k covering the desired k range of the spectrum
- Bunch-Davies vacuum initial conditions per mode from bdic(k, phi, dphi, ddphi, N)

## Outputs

- Scalar primordial power spectrum P(k) over the chosen k range
- Plotted spectrum figure (e.g. spectra.gif)
- Mode solutions evaluated at horizon exit

## How to apply

First integrate the inflationary background equations with pyoscode and detect the end of inflation by root finding on the slow-roll condition epsilon = 1 (the endinfl function crosses zero when inflation ends). For each perturbation mode k, set Bunch-Davies vacuum initial conditions using the bdic(k, phi, dphi, ddphi, N) function, which defines the Bunch-Davies vacuum solution for a given perturbation mode. Integrate the highly oscillatory Mukhanov-Sasaki mode equations with pyoscode.solve_fn, letting the solver switch dynamically between Runge-Kutta and WKB steps in regions of slowly varying frequency, which allows it to skip over many periods of oscillation in one step. Evaluate the resulting mode solutions at horizon exit and assemble the scalar primordial power spectrum over the k range, then plot the computed spectrum (e.g. producing spectra.gif).

## Related tools

- **pyoscode** (Integrates the Mukhanov-Sasaki perturbation mode equations and background equations, switching between Runge-Kutta and WKB steps) — https://github.com/fruzsinaagocs/oscode
- **numpy** (Array handling for k modes, solver inputs, and spectrum assembly (installed automatically with pyoscode))
- **matplotlib** (Plotting the computed scalar primordial power spectrum)
- **scipy** (Root finding for detecting the end of inflation (epsilon = 1))

## Examples

```
sol = pyoscode.solve_fn(w, g, ti, tf, x0, dx0, t_eval=t_eval)  # integrate one Mukhanov-Sasaki mode with Bunch-Davies ICs from bdic(k, phi, dphi, ddphi, N)
```

## Evaluation signals

- Verify that the generated spectrum figure exists (spectra.gif).
- Check that the computed power spectrum qualitatively matches the reported scalar primordial power spectrum from the paper's applications section.
- Confirm that the end-of-inflation root-finding event (epsilon = 1) triggered correctly during the background integration.
- Confirm each mode k was initialized from the Bunch-Davies vacuum via bdic before integration.
- Check that mode solutions evaluated at horizon exit yield a smooth spectrum over the k range without numerical artifacts from the oscillatory integration.

## Limitations

- The WKB-based efficiency gain requires the frequency to change slowly relative to the timescales of integration; the approximation is valid only in slowly varying regions.
- pyoscode's minimum numpy version is numpy>=1.20.0 and Python 2.7 is not supported (3.7+).
- Eigen include errors can occur during C++ builds; the location may need to be set via CPLUS_INCLUDE_PATH.

## Evidence

- [results] Bunch-Davies vacuum initial conditions via bdic: "def bdic(k, phi, dphi, ddphi, N):
    """ Defines the Bunch-Davies vacuum solution
    for a given perturbation mode """"
- [results] End of inflation detection: "def endinfl(t, y):
    """ Crosses zero when inflation ends """"
- [abstract] RK/WKB dynamic switching: "dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [results] Function-based solver interface used for mode equations: "Users can now define w, g as functions in Python (pyoscode) and call the solver via pyoscode.solve_fn(...)"
- [readme] Solver scope: oscillatory ODEs with array frequency and friction: "Oscode is a C++ tool with a Python interface that solves **osc**\illatory **o**\rdinary **d**\ifferential **e**\quations efficiently"
