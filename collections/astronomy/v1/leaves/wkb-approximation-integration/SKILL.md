---
name: wkb-approximation-integration
description: Use when the ODE is highly oscillatory with a frequency w(t) that varies
  slowly relative to the integration timescale over at least part of the range, and
  standard Runge-Kutta stepping would be forced to resolve every period.
when_to_use_negative:
- The frequency w(t) changes rapidly relative to the integration timescale over the
  whole interval — the WKB approximation is only valid when the frequency varies slowly,
  so the method offers no advantage there
- The ODE is not of the oscillatory form x'' + g(t)x' + w^2(t)x = 0 (the solver targets
  second-order oscillatory equations, not general first-order systems)
- A non-oscillatory stiff or algebraic problem where a standard Runge-Kutta solver
  is already efficient
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pyoscode
  role: Python interface to the hybrid RK/WKB stepping solver; installed via pip and
    invoked through pyoscode.solve or pyoscode.solve_fn
  repo: https://pypi.org/project/pyoscode/
- name: oscode
  role: Header-only C++ implementation of the oscillatory ODE solver with the same
    RK/WKB stepping loop, used e.g. for the burst equation example
  repo: https://github.com/fruzsinaagocs/oscode
- name: numpy
  role: Supplies the ts, ws, gs arrays and handles dense-output extraction and error
    computation against the analytic solution
  repo: https://pypi.org/project/numpy/
- name: scipy
  role: Optional dependency used for tests and for the analytic Airy-function comparison
    (Ai(-t)) in examples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:514ec6bacb712a1b0235407881dc763cc7f9b8f9ccd01e8e981285e926d980ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - numpy
  - oscode
  - pyoscode
  - scipy
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# wkb-approximation-integration

## Summary

Integrates highly oscillatory ODEs of the form x'' + g(t)x' + w^2(t)x = 0 by embedding a WKB (Wentzel-Kramers-Brillouin) analytic approximation in a stepping procedure, dynamically switching between WKB and Runge-Kutta estimates so single steps can span many oscillation periods.

## When to use

Use when the ODE is highly oscillatory with a frequency w(t) that varies slowly relative to the integration timescale over at least part of the range, and standard Runge-Kutta stepping would be forced to resolve every period.

## When NOT to use

- The frequency w(t) changes rapidly relative to the integration timescale over the whole interval — the WKB approximation is only valid when the frequency varies slowly, so the method offers no advantage there
- The ODE is not of the oscillatory form x'' + g(t)x' + w^2(t)x = 0 (the solver targets second-order oscillatory equations, not general first-order systems)
- A non-oscillatory stiff or algebraic problem where a standard Runge-Kutta solver is already efficient

## Inputs

- Frequency array ws and friction array gs sampled on a grid ts (or w(t), g(t) as Python callables for pyoscode.solve_fn)
- Initial conditions x0, dx0 and integration interval [ti, tf]
- Optional dense-output evaluation grid t_eval
- Solver tolerance rtol (e.g. 1e-4)

## Outputs

- Numerical solution trajectory x(t) over [ti, tf] spanning many oscillation periods
- Dense-output values at the requested t_eval times
- Comparison artifacts, e.g. times, numerical solution, and analytic values written to a CSV such as airy_solution.csv

## How to apply

Define the frequency and friction term either as arrays (ts, ws, gs) or as callables and supply initial conditions x0, dx0. Call pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval) (or pyoscode.solve_fn / the C++ header-only oscode with Eigen); the hybrid stepper alternates between a Runge-Kutta estimate and the WKB approximation wherever w(t) changes slowly, letting steps skip over many oscillation periods. Request dense output at the t_eval times, which is produced alongside the RK and WKB steps. Validate the trajectory against an analytic solution (e.g. the Airy equation with w(t)=sqrt(t), g=0 solved on t in [1, 100] against Ai(-t)), requiring the relative error to stay at or below the requested tolerance, rtol=1e-4 as used in the paper's accuracy plots.

## Related tools

- **pyoscode** (Python interface to the hybrid RK/WKB stepping solver; installed via pip and invoked through pyoscode.solve or pyoscode.solve_fn) — https://pypi.org/project/pyoscode/
- **oscode** (Header-only C++ implementation of the oscillatory ODE solver with the same RK/WKB stepping loop, used e.g. for the burst equation example) — https://github.com/fruzsinaagocs/oscode
- **numpy** (Supplies the ts, ws, gs arrays and handles dense-output extraction and error computation against the analytic solution) — https://pypi.org/project/numpy/
- **scipy** (Optional dependency used for tests and for the analytic Airy-function comparison (Ai(-t)) in examples)

## Examples

```
sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval)
```

## Evaluation signals

- Relative error of the numerical trajectory against the analytic solution (e.g. Ai(-t) for the Airy test with w(t)=sqrt(t), g=0) stays at or below the requested rtol (1e-4 in the paper's accuracy plots) across the whole interval
- Step counts / runtime are substantially smaller than a pure Runge-Kutta integration, demonstrating that steps skip over many oscillation periods
- Dense output values at t_eval are produced alongside the RK and WKB steps and align with the stepped solution when plotted
- Solver output read back from file (e.g. output.txt from the C++ burst example) matches the analytic solution 100*(1+t^2)^0.5/100*cos(100*arctan(t))
- pytest tests/ from the repository passes, confirming the installed pyoscode behaves as validated by CI

## Limitations

- The WKB approximation is valid only when the frequency changes slowly relative to the timescales of integration; the method is worth applying only when this holds for at least part of the range
- Frequency w and friction g must be supplied as arrays (which need not be evenly spaced) or as callables, so problems outside this parameterization require reformulation
- Historically, dense-output generation had correctness bugs at the C++ interface (fixed in 1.1.2/1.0.x releases), so dense output quality depends on the library version

## Evidence

- [abstract] The method steps along the numerical solution, dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency: "The method steps along the numerical solution, dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [readme] Oscode makes use of an analytic approximation of x(t) embedded in a stepping procedure to skip over long regions of oscillations, giving a reduction in computing time.: "Oscode makes use of an analytic approximation of x(t) embedded in a stepping procedure to skip over long regions of oscillations, giving a reduction in computing time."
- [readme] The approximation is valid when the frequency changes slowly relative to the timescales of integration: "The approximation is valid when the frequency changes slowly relative to the timescales of integration, it is therefore worth applying when
this condition holds for at least some part of the"
- [results] sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0,t_eval=t_eval): "pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0,t_eval=t_eval)"
