---
name: mukhanov-sasaki-mode-integration
description: 'Use when computing the scalar primordial power spectrum from an inflationary
  background: you need to integrate oscillatory cosmological perturbation modes over
  a k range after solving the background and locating the end of inflation.'
when_to_use_negative:
- The ODE system is non-oscillatory or its frequency varies rapidly everywhere — the
  WKB approximation in pyoscode is only valid when the frequency changes slowly relative
  to the integration timescale
- The problem is the Airy or burst-equation demonstration cases (pyoscode.solve /
  C++ burst example) rather than the cosmological perturbation pipeline with Bunch-Davies
  initial conditions
- The frequency and friction terms are known constant arrays amenable to standard
  stiff/adaptive solvers without oscillatory stepping
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: pyoscode
  role: Python interface to oscode used to integrate the Mukhanov-Sasaki mode equations
    via pyoscode.solve_fn with Bunch-Davies initial conditions and RK/WKB step switching
  repo: https://github.com/fruzsinaagocs/oscode
- name: numpy
  role: Strictly necessary Python dependency automatically installed with pyoscode
- name: matplotlib
  role: Optional dependency used to plot the computed scalar primordial power spectrum
- name: oscode
  role: C++ header-only solver with a Python interface that steps between Runge-Kutta
    and WKB approximations for oscillatory ODEs
  repo: https://github.com/fruzsinaagocs/oscode
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:3fcd393a21f7dc59f64bbf3333c2effba97577ba6ff183261079ddd096e00cb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fruzsinaagocs/oscode
  tools:
  - Python
  - matplotlib
  - numpy
  - oscode
  - pyoscode
  - scipy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mukhanov-sasaki-mode-integration

## Summary

Integration of the highly oscillatory Mukhanov-Sasaki perturbation mode equations with pyoscode, initialized from the Bunch-Davies vacuum, to assemble the scalar primordial power spectrum in the pyoscode cosmology example.

## When to use

Use when computing the scalar primordial power spectrum from an inflationary background: you need to integrate oscillatory cosmological perturbation modes over a k range after solving the background and locating the end of inflation.

## When NOT to use

- The ODE system is non-oscillatory or its frequency varies rapidly everywhere — the WKB approximation in pyoscode is only valid when the frequency changes slowly relative to the integration timescale
- The problem is the Airy or burst-equation demonstration cases (pyoscode.solve / C++ burst example) rather than the cosmological perturbation pipeline with Bunch-Davies initial conditions
- The frequency and friction terms are known constant arrays amenable to standard stiff/adaptive solvers without oscillatory stepping

## Inputs

- pyoscode installation (pip install pyoscode; numpy pulled in automatically, matplotlib for plotting)
- examples/cosmology.ipynb notebook from the fruzsinaagocs/oscode repository
- Inflationary background solution (phi, dphi, ddphi) from pyoscode integration
- Perturbation mode wavenumbers k spanning the target range of the power spectrum

## Outputs

- Mukhanov-Sasaki mode solutions evaluated at horizon exit for each k
- Scalar primordial power spectrum over the k range (plotted, cf. spectra.gif)

## How to apply

First integrate the inflationary background equations with pyoscode and detect the end of inflation by root finding on the slow-roll condition epsilon = 1 (the endinfl function crosses zero when inflation ends). For each perturbation mode k, impose Bunch-Davies vacuum initial conditions using the bdic(k, phi, dphi, ddphi, N) function, which 'Defines the Bunch-Davies vacuum solution for a given perturbation mode'. Integrate the highly oscillatory Mukhanov-Sasaki mode equations with pyoscode.solve_fn, letting the solver dynamically switch between Runge-Kutta and WKB steps in regions of slowly varying frequency. Finally, evaluate the mode solutions at horizon exit and assemble the scalar primordial power spectrum over the k range, plotting the computed spectrum (the README shows the spectra.gif figure). The WKB stepping rationale is that the analytic approximation lets the solver skip over many periods of oscillation where the frequency changes slowly relative to the integration timescale.

## Related tools

- **pyoscode** (Python interface to oscode used to integrate the Mukhanov-Sasaki mode equations via pyoscode.solve_fn with Bunch-Davies initial conditions and RK/WKB step switching) — https://github.com/fruzsinaagocs/oscode
- **numpy** (Strictly necessary Python dependency automatically installed with pyoscode)
- **matplotlib** (Optional dependency used to plot the computed scalar primordial power spectrum)
- **oscode** (C++ header-only solver with a Python interface that steps between Runge-Kutta and WKB approximations for oscillatory ODEs) — https://github.com/fruzsinaagocs/oscode

## Examples

```
from pyoscode import solve_fn; sol = solve_fn(w, g, ti, tf, x0, dx0)  # Mukhanov-Sasaki mode k with Bunch-Davies ICs from bdic(k, phi, dphi, ddphi, N)
```

## Evaluation signals

- The generated spectrum figure exists (spectra.gif) after running the cosmology notebook
- The computed power spectrum qualitatively matches the reported scalar primordial power spectrum from the paper's applications section
- Root finding on endinfl correctly crosses zero at the slow-roll condition epsilon = 1, marking the end of inflation
- Solver output shows dynamic switching between Runge-Kutta and WKB steps in regions of slowly varying frequency
- Mode solutions evaluated at horizon exit assemble a smooth spectrum across the chosen k range

## Limitations

- The WKB approximation embedded in oscode is valid only when the frequency changes slowly relative to the integration timescales; it is worth applying only when this condition holds for at least some part of the integration range
- numpy>=1.20.0 is required (per the changelog); Python 2.7 is not supported
- Eigen include errors can occur during C++ builds; set CPLUS_INCLUDE_PATH to the system Eigen location (e.g. /usr/include/eigen3)

## Evidence

- [results] Bunch-Davies vacuum initial conditions: "def bdic(k, phi, dphi, ddphi, N):
    """ Defines the Bunch-Davies vacuum solution
    for a given perturbation mode """"
- [results] end-of-inflation root finding: "def endinfl(t, y):
    """ Crosses zero when inflation ends """"
- [abstract] RK/WKB dynamic switching: "dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [results] function-based solver entry point: "Users can now define w, g as functions in Python (pyoscode) and call the solver via pyoscode.solve_fn(...)"
- [readme] WKB applicability condition: "The approximation is valid when the frequency changes slowly relative to the timescales of integration"
