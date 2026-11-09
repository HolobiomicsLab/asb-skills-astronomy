---
name: runge-kutta-hybrid-step-switching
description: Use when integrating a second-order linear ODE x'' + g(t)x' + w(t)^2
  x = 0 whose frequency is high or rapidly oscillatory on part of the range but slowly
  varying elsewhere, and standard RK stepping would be prohibitively expensive.
when_to_use_negative:
- The frequency changes rapidly relative to the timescales of integration over the
  whole range, violating the WKB validity condition so steps cannot skip oscillations.
- The problem is not of the oscillatory ODE form x'' + g(t)x' + w(t)^2 x = 0 that
  oscode's stepper targets.
- A non-oscillatory ODE is involved, where a plain Runge-Kutta solver is cheaper and
  the WKB machinery adds overhead.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: pyoscode
  role: Python interface to the hybrid RK/WKB stepping loop; called via pyoscode.solve(ts,
    ws, gs, ti, tf, x0, dx0, t_eval=t_eval)
  repo: github.com/fruzsinaagocs/oscode
- name: oscode
  role: Header-only C++ core implementing the oscillatory ODE solver and its analytic
    (WKB) approximation embedded in the stepping procedure
  repo: github.com/fruzsinaagocs/oscode
- name: numpy
  role: Builds the ts, ws, gs arrays and t_eval grid passed to pyoscode.solve
  repo: https://pypi.org/project/numpy/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:6689e93976123c3956fe17bfe3046a65ed331a859bb98cfde61fcb158c1f23f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - numpy
  - oscode
  - pyoscode
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# runge-kutta-hybrid-step-switching

## Summary

Implements oscode's hybrid stepping loop, which dynamically alternates between a Runge-Kutta estimate and an embedded WKB (analytic) approximation wherever the frequency w(t) varies slowly, so single steps can span many oscillation periods while staying accurate at high frequencies.

## When to use

Use when integrating a second-order linear ODE x'' + g(t)x' + w(t)^2 x = 0 whose frequency is high or rapidly oscillatory on part of the range but slowly varying elsewhere, and standard RK stepping would be prohibitively expensive.

## When NOT to use

- The frequency changes rapidly relative to the timescales of integration over the whole range, violating the WKB validity condition so steps cannot skip oscillations.
- The problem is not of the oscillatory ODE form x'' + g(t)x' + w(t)^2 x = 0 that oscode's stepper targets.
- A non-oscillatory ODE is involved, where a plain Runge-Kutta solver is cheaper and the WKB machinery adds overhead.

## Inputs

- frequency and friction terms as arrays ts, ws, gs or as functions w(t), g(t)
- initial conditions x0, dx0
- integration interval [ti, tf] (e.g. t in [1, 100] for the Airy test with w(t)=sqrt(t), g=0)
- dense-output evaluation grid t_eval
- tolerance rtol (1e-4 in the paper's accuracy plots)

## Outputs

- stepped numerical solution switching between RK and WKB steps
- dense-output solution evaluated at t_eval
- trajectory to be compared against the analytic solution (e.g. Ai(-t)) and written to e.g. airy_solution.csv

## How to apply

Supply the frequency and friction terms either as arrays (ts, ws, gs) or as Python callables (w, g), plus initial conditions x0, dx0, the integration interval [ti, tf], and an evaluation grid t_eval for dense output. pyoscode.solve then runs the fixed adaptive stepping architecture: at each step it decides whether to take a Runge-Kutta estimate or a WKB step, the latter valid when the frequency changes slowly relative to the timescale of integration, letting the solver skip over many periods of oscillation in one step. Use rtol=1e-4 as the requested tolerance, matching the paper's accuracy plots. For the Airy benchmark on t in [1, 100] with w(t)=sqrt(t) and g=0, extract the dense output at t_eval and compare pointwise against the analytic Airy solution Ai(-t) to confirm the loop switched correctly and stayed within tolerance.

## Related tools

- **pyoscode** (Python interface to the hybrid RK/WKB stepping loop; called via pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval)) — github.com/fruzsinaagocs/oscode
- **oscode** (Header-only C++ core implementing the oscillatory ODE solver and its analytic (WKB) approximation embedded in the stepping procedure) — github.com/fruzsinaagocs/oscode
- **numpy** (Builds the ts, ws, gs arrays and t_eval grid passed to pyoscode.solve) — https://pypi.org/project/numpy/

## Examples

```
pip install pyoscode; sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0, t_eval=t_eval)
```

## Evaluation signals

- Relative error of the hybrid-step solution against the analytic Airy solution Ai(-t) stays at or below rtol=1e-4 across [1, 100].
- Step trace shows RK steps in regions of rapidly varying frequency and WKB steps (spanning many periods) where frequency varies slowly.
- Dense output at t_eval is produced alongside the RK and WKB steps and matches the stepped solution.
- pytest tests/ passes for the installed package.
- Per-step cost decreases: single WKB steps cover many oscillation periods instead of one period per RK step.

## Limitations

- The WKB approximation is only valid when the frequency changes slowly relative to the timescales of integration; it is worth applying only when this holds for at least some part of the integration range.
- Dense output has known edge-case bugs in earlier versions (e.g. descending t_eval ordering), fixed progressively up to v1.1.2.
- pip/setup.py builds can fail with Eigen include errors, requiring CPLUS_INCLUDE_PATH to point at the Eigen location (e.g. /usr/include/eigen3).

## Evidence

- [abstract] hybrid stepping loop alternates RK and WKB: "dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [abstract] skip many periods per step: "thus being able to skip over many periods of"
- [readme] oscode solver description: "Oscode is a C++ tool with a Python interface that solves **osc**\illatory **o**\rdinary **d**\ifferential **e**\quations efficiently"
- [readme] WKB validity condition: "The approximation is valid when the frequency changes slowly
relative to the timescales of integration"
- [results] pyoscode solve call signature: "sol = pyoscode.solve(ts, ws, gs, ti, tf, x0, dx0,t_eval=t_eval)"
- [results] dense output support: "plt.plot(t_eval, dense, color='C1', label='dense output')"
