---
name: oscillatory-ode-adaptive-stepping
description: Use when integrating a second-order ODE with a high or rapidly varying
  oscillation frequency w(t) (e.g. Airy-type w(t)=sqrt(t) on t in [1,100]) where a
  plain Runge-Kutta solver would need prohibitively small steps to resolve each period.
when_to_use_negative:
- The ODE is not of the oscillatory form x'' + g(t)x' + w(t)^2 x = 0, or w(t) varies
  rapidly relative to the integration timescale everywhere, so the WKB approximation
  is never valid and plain Runge-Kutta is the right choice
- The problem is a first-order or non-oscillatory ODE where standard integrators (e.g.
  scipy solve_ivp) already step efficiently
edam_operation: http://edamontology.org/operation_3946
tools:
- name: pyoscode
  role: Python interface to the hybrid RK/WKB stepper; installed via pip and called
    with pyoscode.solve (array-based w, g) or pyoscode.solve_fn (function-based w,
    g)
  repo: https://pypi.org/project/pyoscode/
- name: oscode
  role: Header-only C++ implementation of the oscillatory ODE solver (include oscode/solver.hpp
    and oscode/system.hpp); used for e.g. the burst equation example
  repo: https://github.com/fruzsinaagocs/oscode
- name: numpy
  role: Supplies the ts, ws, gs arrays and handles reading/comparison of solver output
  repo: https://pypi.org/project/numpy/
- name: scipy
  role: Provides analytic references (e.g. Airy Ai(-t)) used to validate the hybrid-step
    solution against rtol targets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:28d03e6a3f6f876580526b5209bde33513e3254643595d61cb816425cd9d4f71
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - numpy
  - oscode
  - pyoscode
  - scipy
  edam_operation: http://edamontology.org/operation_3946
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# oscillatory-ode-adaptive-stepping

## Summary

Runs the oscode/pyoscode hybrid stepper, which dynamically alternates between Runge-Kutta steps and an embedded WKB approximation to integrate highly oscillatory ODEs like x'' + g(t)x' + w(t)^2 x = 0, taking single steps that span many oscillation periods.

## When to use

Use when integrating a second-order ODE with a high or rapidly varying oscillation frequency w(t) (e.g. Airy-type w(t)=sqrt(t) on t in [1,100]) where a plain Runge-Kutta solver would need prohibitively small steps to resolve each period.

## When NOT to use

- The ODE is not of the oscillatory form x'' + g(t)x' + w(t)^2 x = 0, or w(t) varies rapidly relative to the integration timescale everywhere, so the WKB approximation is never valid and plain Runge-Kutta is the right choice
- The problem is a first-order or non-oscillatory ODE where standard integrators (e.g. scipy solve_ivp) already step efficiently

## Inputs

- Frequency w(t) and friction g(t) given as arrays (ts, ws, gs; need not be evenly spaced) or as Python/C++ functions
- Initial conditions x0, dx0 at integration start ti
- Integration interval [ti, tf] (direction of integration auto-detected)
- Optional dense-output evaluation grid t_eval

## Outputs

- Numerical solution trajectory x(t) over [ti, tf] produced by the hybrid RK/WKB stepping loop
- Dense-output values at requested t_eval points (e.g. written to files like airy_solution.csv or output.txt for C++ runs)
- Per-step solution values comparable against analytic solutions such as Ai(-t) or 100*(1+t^2)^0.5/100*cos(100*arctan(t))

## How to apply

Define the frequency w(t) and friction g(t) either as arrays over the grid (ts, ws, gs) or as callables, plus initial conditions x0, dx0 and integration bounds ti, tf. Call pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval) (or pyoscode.solve_fn when w and g are functions) so the fixed hybrid stepping loop can switch between a Runge-Kutta estimate and the WKB approximation wherever the frequency varies slowly relative to the integration timescale. Request dense output at t_eval to obtain solution values between the solver's internal steps. Validate by comparing the dense-output trajectory against an available analytic solution (e.g. the Airy function Ai(-t) for w=sqrt(t), g=0) and confirming the relative error stays within the requested tolerance (rtol=1e-4 as used in the paper's accuracy plots).

## Related tools

- **pyoscode** (Python interface to the hybrid RK/WKB stepper; installed via pip and called with pyoscode.solve (array-based w, g) or pyoscode.solve_fn (function-based w, g)) — https://pypi.org/project/pyoscode/
- **oscode** (Header-only C++ implementation of the oscillatory ODE solver (include oscode/solver.hpp and oscode/system.hpp); used for e.g. the burst equation example) — https://github.com/fruzsinaagocs/oscode
- **numpy** (Supplies the ts, ws, gs arrays and handles reading/comparison of solver output) — https://pypi.org/project/numpy/
- **scipy** (Provides analytic references (e.g. Airy Ai(-t)) used to validate the hybrid-step solution against rtol targets)

## Examples

```
import pyoscode; sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval)
```

## Evaluation signals

- Relative error of the hybrid-step solution against the analytic solution (e.g. Ai(-t) on t in [1,100]) stays at or below rtol=1e-4 across the interval
- Step count: the solver should take significantly fewer steps than a pure Runge-Kutta run on the same problem, since single steps span many oscillation periods
- Dense output at t_eval matches the trajectory between internal RK and WKB steps (no discontinuities or misordered points; dense output bugs appeared in changelog 1.0.6/1.1.2)
- Running pytest tests/ passes, matching the CI-verified behaviour of the package
- Direction of integration is handled automatically (ti > tf and ti < tf both work; changelog 0.1.1)

## Limitations

- The embedded analytic (WKB) approximation is only valid when the frequency changes slowly relative to the timescales of integration, so gains are limited to problems where this holds for at least part of the range
- Dense output has had known bugs (e.g. when t_eval points are in descending order; fixed only in v1.0.6/1.1.2), so dense-output results should be sanity-checked
- Eigen include paths can break the build (fatal error: Eigen/Dense: No such file or directory), requiring CPLUS_INCLUDE_PATH to be set explicitly

## Evidence

- [abstract] dynamic switching between RK and WKB: "The method steps along the numerical solution, dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [results] solver call signature: "sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0,t_eval=t_eval)"
- [readme] WKB validity condition: "The approximation is valid when the frequency changes slowly relative to the timescales of integration, it is therefore worth applying when this condition holds for at least some part of the"
- [readme] WKB skipping rationale: "Oscode makes use of an analytic approximation of x(t) embedded in a stepping procedure to skip over long regions of oscillations, giving a reduction in computing time."
- [results] dense output accuracy plot: [evidence span withheld — anchor status: not_found]
