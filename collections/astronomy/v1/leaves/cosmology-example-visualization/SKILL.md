---
name: cosmology-example-visualization
description: Use when you need to reproduce or validate the scalar primordial power
  spectrum from Mukhanov-Sasaki mode functions, or benchmark pyoscode on oscillatory
  cosmological perturbation equations with Bunch-Davies initial conditions.
when_to_use_negative:
- The ODE is not of the form x'' + g(t) x' + w^2(t) x = 0 with w and g expressible
  as arrays or Python functions — oscode's stepping assumes this oscillator form
- The frequency w(t) varies rapidly (not slowly relative to integration timescales)
  over the whole domain, so the WKB approximation is invalid everywhere and plain
  Runge-Kutta integration is more appropriate
- The system is not oscillatory — the RK/WKB hybrid offers no advantage over standard
  solvers
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: pyoscode
  role: Integrates the Mukhanov-Sasaki mode equations with Bunch-Davies initial conditions
    via pyoscode.solve_fn, dynamically switching between Runge-Kutta and WKB steps
  repo: https://pypi.org/project/pyoscode/
- name: matplotlib
  role: Optional plotting dependency used to plot the assembled scalar primordial
    power spectrum and produce the spectra figure
  repo: https://pypi.org/project/matplotlib/
- name: numpy
  role: Strictly necessary Python dependency installed automatically with pyoscode;
    used for array-based handling of background quantities and spectrum assembly
  repo: https://pypi.org/project/numpy/
- name: oscode
  role: The underlying C++ tool with a Python interface solving the oscillatory ODE
    x'' + g x' + w^2 x = 0 that the cosmology example exercises
  repo: https://github.com/fruzsinaagocs/oscode
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:a017b6473c1f0c72e8e24d0549709698954275f87de7a286233755957e50f064
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matplotlib
  - numpy
  - oscode
  - pyoscode
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cosmology-example-visualization

## Summary

Reproduces the pyoscode cosmology example: integrating inflationary background and Mukhanov-Sasaki perturbation modes with Bunch-Davies initial conditions, then assembling and plotting the scalar primordial power spectrum. Demonstrates oscode's RK/WKB hybrid stepping on a physical, highly oscillatory system.

## When to use

Use when you need to reproduce or validate the scalar primordial power spectrum from Mukhanov-Sasaki mode functions, or benchmark pyoscode on oscillatory cosmological perturbation equations with Bunch-Davies initial conditions.

## When NOT to use

- The ODE is not of the form x'' + g(t) x' + w^2(t) x = 0 with w and g expressible as arrays or Python functions — oscode's stepping assumes this oscillator form
- The frequency w(t) varies rapidly (not slowly relative to integration timescales) over the whole domain, so the WKB approximation is invalid everywhere and plain Runge-Kutta integration is more appropriate
- The system is not oscillatory — the RK/WKB hybrid offers no advantage over standard solvers

## Inputs

- Perturbation mode wavenumber k (scalar, one mode per integration)
- Inflationary background state (phi, dphi, ddphi, N) feeding bdic()
- Mukhanov-Sasaki mode equation supplied as Python functions w(t), g(t) for pyoscode.solve_fn
- examples/cosmology.ipynb notebook from the fruzsinaagocs/oscode repository

## Outputs

- Scalar primordial power spectrum P(k) evaluated at horizon exit over the chosen k range
- Spectrum figure (spectra.gif in the repo example)
- Background solution with inflation-end point determined by epsilon = 1 root finding

## How to apply

Install pyoscode via 'pip install pyoscode' (numpy is pulled in automatically; install matplotlib for plotting) and open examples/cosmology.ipynb from the fruzsinaagocs/oscode repository. First integrate the inflationary background equations with pyoscode and detect the end of inflation by root finding on the slow-roll condition epsilon = 1 via the endinfl function. Then set Bunch-Davies vacuum initial conditions for each perturbation mode k using the bdic(k, phi, dphi, ddphi, N) function, and integrate the highly oscillatory Mukhanov-Sasaki mode equations with pyoscode.solve_fn, letting the solver switch dynamically between Runge-Kutta and WKB steps in regions of slowly varying frequency. Evaluate the mode solutions at horizon exit, assemble the scalar primordial power spectrum over the k range, and plot it with matplotlib; validate that the spectra figure exists (spectra.gif) and qualitatively matches the scalar primordial power spectrum reported in the paper's applications section.

## Related tools

- **pyoscode** (Integrates the Mukhanov-Sasaki mode equations with Bunch-Davies initial conditions via pyoscode.solve_fn, dynamically switching between Runge-Kutta and WKB steps) — https://pypi.org/project/pyoscode/
- **matplotlib** (Optional plotting dependency used to plot the assembled scalar primordial power spectrum and produce the spectra figure) — https://pypi.org/project/matplotlib/
- **numpy** (Strictly necessary Python dependency installed automatically with pyoscode; used for array-based handling of background quantities and spectrum assembly) — https://pypi.org/project/numpy/
- **oscode** (The underlying C++ tool with a Python interface solving the oscillatory ODE x'' + g x' + w^2 x = 0 that the cosmology example exercises) — https://github.com/fruzsinaagocs/oscode

## Examples

```
sol = pyoscode.solve_fn(w, g, ti, tf, x0, dx0, t_eval=t_eval)  # integrate one Mukhanov-Sasaki mode with Bunch-Davies ICs from bdic(k, phi, dphi, ddphi, N)
```

## Evaluation signals

- The generated spectrum figure exists (spectra.gif in the example output)
- The computed power spectrum qualitatively matches the scalar primordial power spectrum reported in the paper's applications section
- Each perturbation mode k was initialized from the Bunch-Davies vacuum via bdic(k, phi, dphi, ddphi, N) rather than arbitrary initial conditions
- End of inflation was detected by root finding on the slow-roll condition epsilon = 1 (endinfl crosses zero)
- Mode solutions were evaluated at horizon exit before assembling the k-range spectrum

## Limitations

- The WKB approximation used by oscode is valid only when the frequency changes slowly relative to the timescales of integration; the method is worth applying when this holds for at least part of the integration range
- PyPI install of pyoscode requires Eigen; CPLUS_INCLUDE_PATH may need to be set (e.g. CPLUS_INCLUDE_PATH=/usr/include/eigen3) if Eigen/Dense import errors occur
- matplotlib (and scipy) are optional dependencies needed for examples/plotting; without them the plotting step cannot run

## Evidence

- [results] Bunch-Davies vacuum initial conditions: "def bdic(k, phi, dphi, ddphi, N):
    """ Defines the Bunch-Davies vacuum solution
    for a given perturbation mode """"
- [results] Inflation-end root finding: "def endinfl(t, y):
    """ Crosses zero when inflation ends """"
- [readme] WKB validity condition: "The approximation is valid when the frequency changes slowly relative to the timescales of integration"
- [readme] Cosmology example notebook location: "https://mybinder.org/v2/gh/fruzsinaagocs/oscode/master?filepath=examples/cosmology.ipynb"
- [abstract] Hybrid RK/WKB stepping rationale: "dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
