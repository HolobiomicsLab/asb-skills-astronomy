---
name: wkb-rk-switching-solver-usage
description: Use when integrating a second-order linear ODE with a time-dependent
  frequency w(t) and friction g(t) that is highly oscillatory on part of the range
  (e.g. w=sqrt(t) Airy case on t in [1,100]) and standard fixed-method ODE solvers
  would require very small steps.
when_to_use_negative:
- The frequency w(t) varies rapidly over the whole integration range, so the WKB approximation
  is nowhere valid and the method offers no advantage over a plain Runge-Kutta solver
- The problem is not of the form x'' + g(t)x' + w(t)^2 x = 0 (e.g. nonlinear or first-order-only
  systems outside oscode's scope)
- A quick non-oscillatory integration where setting up frequency/friction arrays or
  functions adds overhead without benefit
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pyoscode
  role: Python interface to the WKB/RK-switching solver; called via pyoscode.solve
    (arrays) or pyoscode.solve_fn (functions) with t_eval dense output
  repo: https://github.com/fruzsinaagocs/oscode
- name: oscode
  role: Header-only C++ implementation of the same solver (include oscode/solver.hpp
    and oscode/system.hpp), used e.g. for the burst-equation example writing output.txt
  repo: https://github.com/fruzsinaagocs/oscode
- name: numpy
  role: Builds the ts/ws/gs arrays (e.g. numpy.linspace(1,100,5000), numpy.sqrt) and
    parses solver output for plotting
  repo: https://pypi.org/project/numpy/
- name: scipy
  role: Supplies the analytic Airy-function reference (scipy.special.airy) used to
    compute relative accuracy of RK steps and dense output
  repo: https://pypi.org/project/scipy/
- name: matplotlib
  role: Plots RK steps, WKB steps, dense output, and accuracy-versus-analytic curves
    (e.g. rk-dense-output-rtol1e-4 figure)
  repo: https://pypi.org/project/matplotlib/
- name: pytest
  role: Runs the package test suite to verify the installation
  repo: https://docs.pytest.org/en/stable/getting-started.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:72ba50313cc43c224b9e08d3aa29081e97526c876cc2187e0e825f8b7ce9e6e2
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
  - pytest
  - scipy
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# wkb-rk-switching-solver-usage

## Summary

Apply the oscode/pyoscode solver, which steps along the solution of x'' + g(t)x' + w(t)^2 x = 0 while dynamically switching between a Runge-Kutta estimate and a WKB approximation where the frequency varies slowly, allowing it to skip over many oscillation periods in one step.

## When to use

Use when integrating a second-order linear ODE with a time-dependent frequency w(t) and friction g(t) that is highly oscillatory on part of the range (e.g. w=sqrt(t) Airy case on t in [1,100]) and standard fixed-method ODE solvers would require very small steps.

## When NOT to use

- The frequency w(t) varies rapidly over the whole integration range, so the WKB approximation is nowhere valid and the method offers no advantage over a plain Runge-Kutta solver
- The problem is not of the form x'' + g(t)x' + w(t)^2 x = 0 (e.g. nonlinear or first-order-only systems outside oscode's scope)
- A quick non-oscillatory integration where setting up frequency/friction arrays or functions adds overhead without benefit

## Inputs

- Second-order linear ODE specified by frequency w(t) and friction g(t), either as arrays (ts, ws, gs) sampled on the integration grid or as Python/C++ functions
- Initial conditions x0, dx0 and integration range [ti, tf]
- Optional t_eval array of times at which dense output is requested, and rtol (e.g. 1e-4)
- Optional analytic reference solution for validation (e.g. Ai(-t) from scipy.special.airy)

## Outputs

- RK solution steps and WKB steps over [ti, tf]
- Dense output evaluated at the requested t_eval times
- Validation plots comparing numerical solution and dense output to the analytic solution (e.g. rk-dense-output-rtol1e-4 style figure)

## How to apply

Install pyoscode via pip (or from the fruzsinaagocs/oscode repo with setup.py) and express the problem in the form x'' + g(t)x' + w(t)^2 x = 0. Define the frequency and friction either as arrays (ts, ws, gs, e.g. ts = numpy.linspace(1, 100, 5000), ws = numpy.sqrt(ts), gs = zeros) or as functions, then call pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval) or pyoscode.solve_fn with the requested tolerance (e.g. rtol=1e-4). The solver internally decides at each step whether the frequency is varying slowly enough for the WKB approximation to be valid, switching to Runge-Kutta otherwise; it returns RK steps, WKB steps, and dense output at the requested t_eval times. Validate by evaluating an analytic reference on the same grid (for the Airy test, scipy.special.airy evaluated to get Ai(-t)) and plotting/inspecting the relative error of RK steps and dense output against the requested rtol. The same method is available as header-only C++ (oscode/solver.hpp with Eigen), e.g. for the burst-equation example whose output is compared against 100*(1+t^2)^0.5/100*cos(100*arctan(t)).

## Related tools

- **pyoscode** (Python interface to the WKB/RK-switching solver; called via pyoscode.solve (arrays) or pyoscode.solve_fn (functions) with t_eval dense output) — https://github.com/fruzsinaagocs/oscode
- **oscode** (Header-only C++ implementation of the same solver (include oscode/solver.hpp and oscode/system.hpp), used e.g. for the burst-equation example writing output.txt) — https://github.com/fruzsinaagocs/oscode
- **numpy** (Builds the ts/ws/gs arrays (e.g. numpy.linspace(1,100,5000), numpy.sqrt) and parses solver output for plotting) — https://pypi.org/project/numpy/
- **scipy** (Supplies the analytic Airy-function reference (scipy.special.airy) used to compute relative accuracy of RK steps and dense output) — https://pypi.org/project/scipy/
- **matplotlib** (Plots RK steps, WKB steps, dense output, and accuracy-versus-analytic curves (e.g. rk-dense-output-rtol1e-4 figure)) — https://pypi.org/project/matplotlib/
- **pytest** (Runs the package test suite to verify the installation) — https://docs.pytest.org/en/stable/getting-started.html

## Examples

```
import numpy, pyoscode; ts = numpy.linspace(1, 100, 5000); sol = pyoscode.solve(ts, numpy.sqrt(ts), numpy.zeros_like(ts), 1, 100, x0, dx0, t_eval=t_eval, rtol=1e-4)
```

## Evaluation signals

- Relative error of both the RK solution and the dense output against the analytic reference stays at or below the requested tolerance level (e.g. rtol=1e-4) across the integration range
- Dense output is returned at exactly the requested t_eval times, in the correct (non-descending) order
- The solver demonstrably takes fewer/larger steps through slowly varying frequency regions (WKB) while switching to RK where the frequency changes fast
- pytest tests/ passes after installation, confirming the Python wrapper and C++ core work
- For the burst-equation C++ example, the parsed output.txt solution matches 100*(1+t^2)^0.5/100*cos(100*arctan(t)) when plotted

## Limitations

- The WKB approximation is only valid when the frequency changes slowly relative to the timescales of integration; the method is worthwhile only when this holds for at least some part of the integration range
- Dense output has had documented bugs (fixed progressively in 1.0.4-1.1.2), e.g. when t_eval times are in descending order, so verify dense-output ordering
- Building from source can fail on Eigen include paths (Eigen/Dense not found), requiring CPLUS_INCLUDE_PATH to be set
- Frequencies and friction given as arrays need not be evenly spaced, but the arrays must cover the integration range adequately

## Evidence

- [abstract] dynamic switching between RK and WKB: "dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [results] solver call signature: "sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0,t_eval=t_eval)"
- [methods] installation via pip: "``pyoscode`` can be installed via pip"
- [results] dense output at t_eval: "plt.plot(t_eval, dense, color='C1', label='dense output')"
- [readme] solver form and array interface: "where |gamma| (friction term) and |omega| (frequency) can be given as arrays."
- [readme] WKB validity condition: "The approximation is valid when the frequency changes slowly relative to the timescales of integration"
- [readme] dense output bug fix history: "Fix issues related to dense output not being correctly generated, e.g. when timepoints at which dense output was asked for are in descending order"
- [readme] Eigen include failure mode: "pyoscode/_pyoscode.hpp:6:10: fatal error: Eigen/Dense: No such file or directory"
