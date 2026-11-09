---
name: oscillatory-ode-wkb-solver-usage
description: Use when an ODE is highly oscillatory (e.g. Airy, burst, or Mukhanov-Sasaki
  mode equations) with w(t) and g(t) given as arrays or functions, and standard RK
  integration would need many steps per oscillation period.
when_to_use_negative:
- The ODE is non-oscillatory or the frequency w(t) varies rapidly everywhere — the
  WKB approximation is only valid when the frequency changes slowly relative to integration
  timescales, so no step-skipping benefit is gained
- The equation is nonlinear or not of the form x'' + g(t)x' + w(t)^2 x = 0, since
  oscode's analytic WKB embedding assumes this linear oscillator form
- You need a validated general-purpose integrator for a non-oscillatory problem already
  handled well by standard Runge-Kutta (e.g. via scipy)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pyoscode
  role: Python interface to the oscillatory ODE solver; called via pyoscode.solve
    (array w, g) or pyoscode.solve_fn (function w, g) with initial conditions and
    t_eval
  repo: https://pypi.org/project/pyoscode/
- name: oscode
  role: 'Header-only C++ core solver (e.g. examples/burst.cpp) that performs the RK/WKB
    switching; used directly in C++ with #include <oscode/solver.hpp>'
  repo: github.com/fruzsinaagocs/oscode
- name: numpy
  role: Strictly necessary Python dependency automatically installed with pyoscode;
    holds ts/ws/gs arrays and solver output
  repo: https://pypi.org/project/numpy/
- name: matplotlib
  role: Optional plotting of computed solutions, step points, and the assembled primordial
    power spectrum (spectra.gif)
  repo: https://pypi.org/project/matplotlib/
- name: scipy
  role: Optional dependency for tests and examples/plotting (analytic comparison solutions)
  repo: https://pypi.org/project/scipy/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:c151249997fbe7fab986c0b62f4067407bc3cd044bf5f757efc72072a485f18d
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# oscillatory-ode-wkb-solver-usage

## Summary

Solve second-order oscillatory ODEs x'' + g(t)x' + w(t)^2 x = 0 efficiently with pyoscode/oscode, which dynamically switches between Runge-Kutta and WKB steps to skip over many oscillation periods where the frequency w(t) varies slowly.

## When to use

Use when an ODE is highly oscillatory (e.g. Airy, burst, or Mukhanov-Sasaki mode equations) with w(t) and g(t) given as arrays or functions, and standard RK integration would need many steps per oscillation period.

## When NOT to use

- The ODE is non-oscillatory or the frequency w(t) varies rapidly everywhere — the WKB approximation is only valid when the frequency changes slowly relative to integration timescales, so no step-skipping benefit is gained
- The equation is nonlinear or not of the form x'' + g(t)x' + w(t)^2 x = 0, since oscode's analytic WKB embedding assumes this linear oscillator form
- You need a validated general-purpose integrator for a non-oscillatory problem already handled well by standard Runge-Kutta (e.g. via scipy)

## Inputs

- Second-order linear ODE specification: frequency w(t) and friction g(t) as arrays (ts, ws, gs) or Python functions
- Initial conditions: integration start/end times (ti, tf), x0, dx0
- Optional evaluation times t_eval for dense output
- Cosmology use case: Mukhanov-Sasaki mode k values and background quantities (phi, dphi, ddphi, N) for Bunch-Davies initial conditions via bdic

## Outputs

- Solution values x(t) at RK/WKB steps and at requested dense-output times t_eval
- Step-type diagnostics distinguishing RK vs WKB steps
- Scalar primordial power spectrum over the k range (cosmology example), e.g. plotted as spectra.gif
- C++ example output file 'output.txt' with complex solution pairs

## How to apply

Install pyoscode with 'pip install pyoscode' (numpy is auto-installed; matplotlib for plotting, scipy/pytest for tests). Define the frequency and friction term either as arrays (ts, ws, gs) passed to pyoscode.solve or as Python functions (w, g) passed to pyoscode.solve_fn, along with initial conditions ti, tf, x0, dx0 and optional t_eval for dense output. The solver steps along the solution, dynamically switching between a Runge-Kutta estimate and the WKB approximation wherever the frequency changes slowly relative to the integration timescale, letting it skip many oscillation periods per step. For the cosmology application, set Bunch-Davies vacuum initial conditions per mode k via bdic(k, phi, dphi, ddphi, N), integrate the Mukhanov-Sasaki mode equations, evaluate modes at horizon exit, and assemble the scalar primordial power spectrum. Judge correctness by comparing dense output and step points against analytic solutions (e.g. Airy Ai(-t) on t in [1,100] with w=sqrt(t), g=0) at the chosen tolerance such as rtol=1e-4.

## Related tools

- **pyoscode** (Python interface to the oscillatory ODE solver; called via pyoscode.solve (array w, g) or pyoscode.solve_fn (function w, g) with initial conditions and t_eval) — https://pypi.org/project/pyoscode/
- **oscode** (Header-only C++ core solver (e.g. examples/burst.cpp) that performs the RK/WKB switching; used directly in C++ with #include <oscode/solver.hpp>) — github.com/fruzsinaagocs/oscode
- **numpy** (Strictly necessary Python dependency automatically installed with pyoscode; holds ts/ws/gs arrays and solver output) — https://pypi.org/project/numpy/
- **matplotlib** (Optional plotting of computed solutions, step points, and the assembled primordial power spectrum (spectra.gif)) — https://pypi.org/project/matplotlib/
- **scipy** (Optional dependency for tests and examples/plotting (analytic comparison solutions)) — https://pypi.org/project/scipy/

## Examples

```
from pyoscode import solve; sol = solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval)
```

## Evaluation signals

- Computed solution matches the analytic Airy solution Ai(-t) with w(t)=sqrt(t), g(t)=0 on t in [1,100] to the requested tolerance (e.g. rtol 1e-4), for both array-based and function-based interfaces
- Burst-equation output ('output.txt') matches the analytic solution 100*(1+t^2)^0.5/100*cos(100*arctan(t))
- Step-type diagnostics show WKB steps concentrated in regions of slowly varying frequency and RK steps elsewhere
- Dense output at t_eval times is consistent with the stepped solution points
- Cosmology run: spectrum figure (spectra.gif) is produced and the scalar primordial power spectrum qualitatively matches the paper's applications-section result

## Limitations

- The WKB approximation is valid only when the frequency changes slowly relative to the timescales of integration; the method is worth applying only when this holds for at least part of the integration range
- Dense output had historical bugs (e.g. with descending t_eval times), fixed progressively in versions 1.0.4-1.1.2
- C++ interface requires C++11 or later and Eigen; Eigen include errors may need setting CPLUS_INCLUDE_PATH (e.g. /usr/include/eigen3)
- numpy>=1.20.0 is the minimally required version; Python 2.7 is no longer supported

## Evidence

- [abstract] The method dynamically switches between a Runge-Kutta estimate and the WKB approximation in areas of slowly varying frequency, allowing it to skip over many periods of oscillation: [evidence span withheld — anchor status: not_found]
- [results] pyoscode.solve with array inputs: "sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0,t_eval=t_eval)"
- [results] Bunch-Davies initial conditions function: "def bdic(k, phi, dphi, ddphi, N):
    """ Defines the Bunch-Davies vacuum solution
    for a given perturbation mode """"
- [readme] WKB validity condition: "The approximation is valid when the frequency changes slowly
relative to the timescales of integration, it is therefore worth applying when
this condition holds for at least some part of the"
- [methods] Installation via pip: "pip install pyoscode"
