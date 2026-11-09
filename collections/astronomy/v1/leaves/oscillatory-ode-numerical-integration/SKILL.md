---
name: oscillatory-ode-numerical-integration
description: Use when numerically solving a second-order ODE with an oscillatory solution
  whose frequency w(t) varies slowly relative to the timescale of integration over
  at least part of the range (e.g. Airy, burst equation, or cosmological perturbation
  modes).
when_to_use_negative:
- The frequency does not vary slowly relative to the integration timescale over any
  part of the range — the WKB approximation underlying the speedup is then invalid
- The ODE is not of the supported second-order oscillatory form x'' + g(t)x' + w(t)^2
  x = 0 with scalar w(t) and g(t)
- A first-order multistep solver such as scipy.integrate would suffice because oscillations
  are cheap to resolve — oscode's benefit is skipping periods of high-frequency oscillation
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: pyoscode
  role: Python interface to the oscode solver; used via pyoscode.solve (array-based
    w, g) and pyoscode.solve_fn (callable w, g), returning RK/WKB steps and dense
    output under 'x_eval'
  repo: https://github.com/fruzsinaagocs/oscode
- name: oscode
  role: Header-only C++ solver compiled with g++ -std=c++11 (requires Eigen, included
    in source); the burst example writes steps to output.txt
  repo: https://github.com/fruzsinaagocs/oscode
- name: numpy
  role: Builds the ts/ws/gs grids and t_eval array, parses complex step pairs from
    output.txt with np.genfromtxt, and extracts dense output via np.asarray(sol['x_eval'])
  repo: https://pypi.org/project/numpy/
- name: scipy
  role: Provides scipy.special.airy to construct the analytic Airy reference Ai(-t)
    used to compute relative accuracy of RK steps and dense output
- name: matplotlib
  role: Plots RK steps, WKB steps, and dense output alongside analytic solutions and
    saves comparison figures such as rk-dense-output-rtol1e-4
  repo: https://pypi.org/project/matplotlib/
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_001
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:77b546ea9585735f3504c7b4e954e865c1975f4946ec15a96b56b1bd806a847a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fruzsinaagocs/oscode
  tools:
  - C++
  - Python
  - matplotlib
  - numpy
  - oscode
  - pyoscode
  - scipy
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# oscillatory-ode-numerical-integration

## Summary

Integrate second-order oscillatory ODEs of the form x'' + g(t)x' + w(t)^2 x = 0 with oscode/pyoscode, which dynamically switches between Runge-Kutta steps and a WKB approximation to skip many oscillation periods in one step. Validated against analytic references such as the Airy equation Ai(-t) and the burst equation cos(100*arctan(t)) at rtol=1e-4.

## When to use

Use when numerically solving a second-order ODE with an oscillatory solution whose frequency w(t) varies slowly relative to the timescale of integration over at least part of the range (e.g. Airy, burst equation, or cosmological perturbation modes).

## When NOT to use

- The frequency does not vary slowly relative to the integration timescale over any part of the range — the WKB approximation underlying the speedup is then invalid
- The ODE is not of the supported second-order oscillatory form x'' + g(t)x' + w(t)^2 x = 0 with scalar w(t) and g(t)
- A first-order multistep solver such as scipy.integrate would suffice because oscillations are cheap to resolve — oscode's benefit is skipping periods of high-frequency oscillation

## Inputs

- Second-order ODE specified by frequency w(t) and friction g(t) as evenly- or unevenly-spaced arrays (ts, ws, gs) over t in [ti, tf], e.g. ts = numpy.linspace(1, 100, 5000), ws = numpy.sqrt(ts), gs = 0
- Frequency and friction as Python callables w(t), g(t) for pyoscode.solve_fn
- Initial conditions x0, dx0 (e.g. Bunch-Davies vacuum initial conditions for cosmological perturbation modes)
- Array t_eval of requested dense-output evaluation times (ascending or descending order, not necessarily evenly spaced)

## Outputs

- Numerical solution split into RK step points and WKB step points
- Dense output: interpolated solution values at each requested t_eval point, stored in the result dictionary under the 'x_eval' key (retrieved as np.asarray(sol['x_eval']) and writable to dense_output.csv)
- For the C++ interface: solver step pairs written to 'output.txt' (complex-valued pairs parsed with np.genfromtxt)
- Comparison plots against analytic references, e.g. airy_solution_comparison.png and burst_comparison.png

## How to apply

Define the frequency w(t) and friction g(t) either as arrays (ts, ws, gs, which need not be evenly spaced) or as Python callables, then call pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval) or pyoscode.solve_fn(...) with w and g as functions, requesting rtol=1e-4. The solver steps along the solution, dynamically switching between a Runge-Kutta estimate and the WKB approximation where the frequency varies slowly, producing RK steps, WKB steps, and dense output at the requested t_eval points returned in the result dictionary under the 'x_eval' key. For the C++ interface, clone the repository, include oscode/solver.hpp and oscode/system.hpp, and compile e.g. the burst example with 'g++ -I../include/ -g -Wall -std=c++11' before running ./burst, which writes step pairs to output.txt. Validate by computing the analytic reference (e.g. scipy.special.airy for Ai(-t), or 100*(1+t^2)^0.5/100*cos(100*arctan(t)) for the burst equation) on the same grid and checking that the relative deviation of both RK steps and dense output stays at or below rtol=1e-4 across the integration range.

## Related tools

- **pyoscode** (Python interface to the oscode solver; used via pyoscode.solve (array-based w, g) and pyoscode.solve_fn (callable w, g), returning RK/WKB steps and dense output under 'x_eval') — https://github.com/fruzsinaagocs/oscode
- **oscode** (Header-only C++ solver compiled with g++ -std=c++11 (requires Eigen, included in source); the burst example writes steps to output.txt) — https://github.com/fruzsinaagocs/oscode
- **numpy** (Builds the ts/ws/gs grids and t_eval array, parses complex step pairs from output.txt with np.genfromtxt, and extracts dense output via np.asarray(sol['x_eval'])) — https://pypi.org/project/numpy/
- **scipy** (Provides scipy.special.airy to construct the analytic Airy reference Ai(-t) used to compute relative accuracy of RK steps and dense output)
- **matplotlib** (Plots RK steps, WKB steps, and dense output alongside analytic solutions and saves comparison figures such as rk-dense-output-rtol1e-4) — https://pypi.org/project/matplotlib/

## Examples

```
sol = pyoscode.solve(ts, ws, gs, ti=1, tf=100, x0=x0, dx0=dx0, t_eval=t_eval, rtol=1e-4); dense = np.asarray(sol['x_eval'])
```

## Evaluation signals

- Maximum relative deviation of both RK steps and dense output from the analytic reference (e.g. Ai(-t) via scipy.special.airy) remains at or below the requested rtol=1e-4 across the integration range
- Dense-output artifact contains exactly one row per requested t_eval point, retrievable as np.asarray(sol['x_eval'])
- In the comparison plot, the oscode solution visually and numerically follows the analytic burst envelope cos(100*arctan(t)), with RK and WKB step types distinguishable
- Output from the C++ burst run parses cleanly from output.txt as complex step pairs (np.genfromtxt with dtype=complex and pair converters)
- Repository unit tests pass: pytest tests/

## Limitations

- The WKB approximation is valid only when the frequency changes slowly relative to the timescales of integration; the speedup applies only when this holds for at least part of the range
- Dense output had bugs in earlier versions (fixed around 1.0.5–1.1.2), e.g. for descending t_eval order; use a recent version (1.3.0) to avoid incorrectly generated dense output
- Building the C++ interface may require setting CPLUS_INCLUDE_PATH (e.g. /usr/include/eigen3) if Eigen import errors occur

## Evidence

- [abstract] Dynamic RK/WKB switching: "dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [readme] WKB validity condition: "The approximation is valid when the frequency changes slowly relative to the timescales of integration"
- [results] Array-based solve call: "sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0,t_eval=t_eval)"
- [results] Dense output key: "plt.plot(t_eval, dense, color='C1', label='dense output')"
- [readme] C++ compile/run: "g++ -I../include/ -g -Wall -std=c++11 -c -o burst.o burst.cpp"
- [results] Burst output parsing: "data = np.genfromtxt(f1,dtype=complex,converters={1:parse_pair,2:parse_pair},delimiter=";""
- [readme] Header-only C++ usage: "oscode`` is a header-only C++ package, it requires no installation."
