---
name: frequency-array-problem-specification
description: Use when setting up an oscillatory ODE x'' + g(t)x' + w(t)^2 x = 0 whose
  frequency and friction can be sampled or given as arrays/functions before calling
  pyoscode.solve.
when_to_use_negative:
- The problem is not oscillatory, or w(t) varies rapidly everywhere so the WKB approximation
  embedded in the stepper has no valid region of application
- The equation is not of the form x'' + g(t)x' + w(t)^2 x = 0 (oscode is designed
  specifically for this oscillator form with gamma and omega given as arrays)
- A generic non-oscillatory stiff or adaptive solver suffices and the WKB/RK hybrid
  machinery would add no benefit
tools:
- name: pyoscode
  role: Python interface used to register the array-specified problem (ts, ws, gs)
    and run the hybrid RK/WKB solver via pyoscode.solve or pyoscode.solve_fn
  repo: https://github.com/fruzsinaagocs/oscode
- name: numpy
  role: Strictly necessary Python dependency for building the ts, ws, gs arrays and
    t_eval grid passed to pyoscode.solve
  repo: https://pypi.org/project/numpy/
- name: Python
  role: Runtime environment in which the problem arrays and solve call are scripted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:518d17a08686e139b08fb0e9404e8a148b6e07108be8085f22ffd0a1e8504887
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fruzsinaagocs/oscode
  tools:
  - Python
  - numpy
  - pyoscode
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# frequency-array-problem-specification

## Summary

Specifies an oscillatory ODE problem for the oscode/pyoscode solver by encoding the frequency w(t) and friction g(t) as arrays (ts, ws, gs) over the integration interval, or as callables for pyoscode.solve_fn, plus initial conditions x0, dx0 and a t_eval grid.

## When to use

Use when setting up an oscillatory ODE x'' + g(t)x' + w(t)^2 x = 0 whose frequency and friction can be sampled or given as arrays/functions before calling pyoscode.solve.

## When NOT to use

- The problem is not oscillatory, or w(t) varies rapidly everywhere so the WKB approximation embedded in the stepper has no valid region of application
- The equation is not of the form x'' + g(t)x' + w(t)^2 x = 0 (oscode is designed specifically for this oscillator form with gamma and omega given as arrays)
- A generic non-oscillatory stiff or adaptive solver suffices and the WKB/RK hybrid machinery would add no benefit

## Inputs

- Time grid ts spanning the integration interval (e.g. t in [1, 100]); need not be evenly spaced
- Frequency array ws sampled from w(t) (e.g. w(t)=sqrt(t))
- Friction array gs sampled from g(t) (e.g. g=0 for the Airy case)
- Initial conditions x0 and dx0 (solution value and derivative)
- Start time ti and end time tf
- Optional dense-output evaluation grid t_eval
- Alternatively, w and g as Python callables for pyoscode.solve_fn

## Outputs

- Solver solution object from pyoscode.solve containing the numerical solution
- Dense-output solution evaluated at t_eval (times + values) for comparison against analytic solutions like Ai(-t)
- Validation series: pointwise relative error of the numerical trajectory vs the analytic solution at rtol=1e-4

## How to apply

Sample the frequency and friction terms on a time grid matching the article's example problem, e.g. w(t)=sqrt(t) and g=0 over t in [1, 100], producing arrays ts, ws, gs — the README notes these arrays need not be evenly spaced. Assemble initial conditions x0, dx0 for the solution and its derivative, a start time ti, end time tf, and a dense-output grid t_eval, then call pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval). Alternatively, define w and g as Python callables and use pyoscode.solve_fn (added in version 1.1.0). Validate by comparing dense output against the analytic solution — for the article's Airy example, Ai(-t) — requiring the relative error to stay at or below the requested rtol (1e-4 in the paper's accuracy plots) across the whole interval.

## Related tools

- **pyoscode** (Python interface used to register the array-specified problem (ts, ws, gs) and run the hybrid RK/WKB solver via pyoscode.solve or pyoscode.solve_fn) — https://github.com/fruzsinaagocs/oscode
- **numpy** (Strictly necessary Python dependency for building the ts, ws, gs arrays and t_eval grid passed to pyoscode.solve) — https://pypi.org/project/numpy/
- **Python** (Runtime environment in which the problem arrays and solve call are scripted)

## Examples

```
sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval)
```

## Evaluation signals

- The numerical trajectory tracks the analytic Airy solution Ai(-t) with relative error at or below rtol=1e-4 across the full interval t in [1, 100]
- Dense output is returned at every requested t_eval point, in the requested order (the solver fixed bugs when t_eval times are in descending order)
- Problem specification passes the package's own test suite: pytest tests/ runs cleanly
- Arrays ws and gs are accepted even when unevenly spaced (guaranteed from version 1.0.0)

## Limitations

- The embedded WKB approximation is valid only when the frequency changes slowly relative to the timescales of integration; it is worth applying when this holds for at least part of the range
- oscode targets the oscillator form x'' + g x' + w^2 x = 0; equations outside this form cannot be specified this way
- Building from source requires Eigen; import errors ('Eigen/Dense: No such file or directory') may need CPLUS_INCLUDE_PATH to point at the Eigen location; numpy>=1.20.0 and Python 3.7+ are required

## Evidence

- [results] solve call with arrays: "sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0,t_eval=t_eval)"
- [readme] array interface for w and g: "where |gamma| (friction term) and |omega| (frequency) can be given as arrays."
- [readme] function-based interface: "Users can now define w, g as functions in Python (pyoscode) and call the solver via pyoscode.solve_fn(...)"
- [readme] validity condition: "The approximation is valid when the frequency changes slowly relative to the timescales of integration"
- [results] validation threshold: [evidence span withheld — anchor status: not_found]
