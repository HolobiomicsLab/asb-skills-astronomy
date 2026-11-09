---
name: wkb-runge-kutta-hybrid-solving
description: Use when integrating second-order linear oscillatory ODEs of the form
  x'' = -(w(t)^2)x - g(t)x' with a specified frequency w(t) and friction g(t) that
  oscillate rapidly but vary slowly over part of the range, where standard RK alone
  is inefficient.
when_to_use_negative:
- Problems where the frequency does not vary slowly relative to the integration timescale
  anywhere in the range — the WKB approximation is invalid and the hybrid offers no
  advantage
- Non-oscillatory ODEs solvable efficiently with a standard RK integrator, where the
  hybrid machinery adds overhead without benefit
- Cases requiring error-controlled stepping with no user-supplied frequency function,
  since the method relies on w(t) and g(t) being specified as arrays or functions
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: oscode
  role: Header-only C++11 package that solves oscillatory ODEs with the hybrid RK/WKB
    stepping method (e.g. the burst example); compiled with g++ -I../include/ -std=c++11
  repo: https://github.com/fruzsinaagocs/oscode
- name: pyoscode
  role: Python interface to the solver, installed via pip or setup.py; exposes pyoscode.solve(ts,
    ws, gs, ti, tf, x0, dx0, t_eval=t_eval) and pyoscode.solve_fn
  repo: https://github.com/fruzsinaagocs/oscode
- name: Eigen
  role: Header-only linear algebra library required to build the C++ interface (bundled
    in the source for pip installability)
- name: numpy
  role: Parses solver output (e.g. np.genfromtxt with converters for complex step
    pairs) and evaluates analytic reference solutions on the step grid
  repo: https://pypi.org/project/numpy/
- name: matplotlib
  role: Plots RK and WKB solution steps against analytic solutions to validate the
    hybrid integration
  repo: https://pypi.org/project/matplotlib/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:65ffccfa99ae2a4b810a362a8ae1d6baf1f62fd9b6378ee13acb888f2cc0b172
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fruzsinaagocs/oscode
  tools:
  - C++
  - Eigen
  - Python
  - matplotlib
  - numpy
  - oscode
  - pyoscode
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# wkb-runge-kutta-hybrid-solving

## Summary

A hybrid ODE integration strategy that dynamically switches between a Runge-Kutta estimate and the Wentzel-Kramers-Brillouin (WKB) approximation in regions of slowly varying frequency, implemented in the oscode C++ solver / pyoscode Python package, so many oscillation periods are skipped per step without losing accuracy.

## When to use

Use when integrating second-order linear oscillatory ODEs of the form x'' = -(w(t)^2)x - g(t)x' with a specified frequency w(t) and friction g(t) that oscillate rapidly but vary slowly over part of the range, where standard RK alone is inefficient.

## When NOT to use

- Problems where the frequency does not vary slowly relative to the integration timescale anywhere in the range — the WKB approximation is invalid and the hybrid offers no advantage
- Non-oscillatory ODEs solvable efficiently with a standard RK integrator, where the hybrid machinery adds overhead without benefit
- Cases requiring error-controlled stepping with no user-supplied frequency function, since the method relies on w(t) and g(t) being specified as arrays or functions

## Inputs

- Frequency and friction term w(t), g(t) as arrays (ts, ws, gs) or as functions/classes
- Initial conditions: start/end times (ti, tf), initial value x0 and derivative dx0
- Optional t_eval array of evaluation times for dense output
- For the C++ interface: a system definition compiled against the header-only oscode sources (Eigen included in source)

## Outputs

- Numerical solution steps tagged by step type (RK steps and WKB steps), e.g. written to output.txt as complex-valued (t, x) pairs
- Dense output of the solution at requested t_eval times
- Comparison plots (e.g. matplotlib) of solver steps versus analytic reference solutions

## How to apply

Define the frequency and friction term either as arrays (ts, ws, gs) or as callables and hand them to the solver via pyoscode.solve / pyoscode.solve_fn, or embed the system in a C++ program including <oscode/solver.hpp> and <oscode/system.hpp>. The solver steps along the solution, dynamically switching between a Runge-Kutta estimate and the WKB approximation wherever the frequency changes slowly relative to the integration timescale, so it can jump over many oscillation periods in one step. Provide initial conditions (ti, tf, x0, dx0) and optionally t_eval for dense output at requested evaluation times; the WKB approximation is applied only where its validity condition (slowly varying frequency) holds, otherwise the method falls back to RK. For the C++ burst example, compile with 'g++ -I../include/ -g -Wall -std=c++11' and run ./burst, which writes numerical steps (RK and WKB step types distinguished) to output.txt; parse the complex-valued step pairs with numpy.genfromtxt and compare against an available analytic solution such as 100*(1+t^2)^0.5/100*cos(100*arctan(t)).

## Related tools

- **oscode** (Header-only C++11 package that solves oscillatory ODEs with the hybrid RK/WKB stepping method (e.g. the burst example); compiled with g++ -I../include/ -std=c++11) — https://github.com/fruzsinaagocs/oscode
- **pyoscode** (Python interface to the solver, installed via pip or setup.py; exposes pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval) and pyoscode.solve_fn) — https://github.com/fruzsinaagocs/oscode
- **Eigen** (Header-only linear algebra library required to build the C++ interface (bundled in the source for pip installability))
- **numpy** (Parses solver output (e.g. np.genfromtxt with converters for complex step pairs) and evaluates analytic reference solutions on the step grid) — https://pypi.org/project/numpy/
- **matplotlib** (Plots RK and WKB solution steps against analytic solutions to validate the hybrid integration) — https://pypi.org/project/matplotlib/

## Examples

```
sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval)
```

## Evaluation signals

- The numerical solution visually and numerically matches the analytic burst-equation reference 100*(1+t^2)^0.5/100*cos(100*arctan(t)) envelope over the integration range
- RK and WKB steps are distinguishable in the comparison plot, confirming the solver switched step types as the frequency behavior demanded
- For the Airy test (w(t)=sqrt(t), g(t)=0, t in [1,100]), the solution matches Ai(-t) through both the array-based and function-based interfaces
- Dense output at t_eval is consistent with the underlying RK/WKB steps
- pytest tests/ suite passes (pytest plus scipy are the documented test dependencies)

## Limitations

- The WKB approximation is only valid when the frequency changes slowly relative to the integration timescales; outside that regime accuracy degrades and the method relies on RK fallback
- Known dense-output bugs existed in releases 1.0.5-1.1.2 (e.g. wrong output when t_eval times were in descending order) — use a recent version
- Build issues can arise with Eigen include paths; set CPLUS_INCLUDE_PATH (e.g. /usr/include/eigen3) if 'Eigen/Dense: No such file or directory' occurs
- numpy >= 1.20.0 is required; Python 2.7 is no longer supported

## Evidence

- [abstract] hybrid switch rationale: "dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [readme] skipping oscillations rationale: "Oscode makes use of an analytic approximation of x(t) embedded in a
stepping procedure to skip over long regions of oscillations"
- [methods] C++ burst compile/run: "g++ -I../include/ -g -Wall -std=c++11 -c -o burst.o burst.cpp"
- [results] parsing output.txt step pairs: "data = np.genfromtxt(f1,dtype=complex,converters={1:parse_pair,2:parse_pair},delimiter=";""
