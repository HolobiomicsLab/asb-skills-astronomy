---
name: galactic-potential-source-term-implementation
description: Use when a solver for the 2-D compressible isothermal Euler equations
  in a barred galaxy potential must be set up or run, e.g. reproducing g2test or porting
  the galactic-potential and rotating-bar source terms to another code.
when_to_use_negative:
- The gas is not well described by an isothermal closure (p = c_s^2 * rho) — the package
  solves only the isothermal Euler system.
- The problem is 1-D, 3-D, or lacks a rotating weak bar / axisymmetric galactic potential,
  so the bar and potential source terms do not apply.
- A non-stationary (time-evolving) solution is the goal; the code targets stationary
  solutions.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: galax2d
  role: Reference Matlab implementation of the 2-D isothermal Euler solver with galactic-potential
    and rotating-bar source terms; run entry point g2test
  repo: https://github.com/Someone789/galax2d
- name: Matlab 8.0 (R2012b)
  role: Runtime used to execute g2test and makefigs from the packaged run and figures4paper
    directories
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.03.005
    title: 2-D stationary gas dynamics in a barred galaxy
  source_content_hash: sha256:4a06e72d445beb2db9381f158c11ea8e24905b5f0aaa41fdf51d59ad3071e8e5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Someone789/galax2d
  tools:
  - Matlab 8.0 (R2012b)
  - galax2d
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# galactic-potential-source-term-implementation

## Summary

Method for implementing the source terms (axisymmetric galactic gravitational potential plus a rotating weak bar evaluated at the bar pattern speed) that drive a stationary solution of the 2-D compressible isothermal Euler equations, as provided by the galax2d Matlab package of Mulder (2015).

## When to use

Use when a solver for the 2-D compressible isothermal Euler equations in a barred galaxy potential must be set up or run, e.g. reproducing g2test or porting the galactic-potential and rotating-bar source terms to another code.

## When NOT to use

- The gas is not well described by an isothermal closure (p = c_s^2 * rho) — the package solves only the isothermal Euler system.
- The problem is 1-D, 3-D, or lacks a rotating weak bar / axisymmetric galactic potential, so the bar and potential source terms do not apply.
- A non-stationary (time-evolving) solution is the goal; the code targets stationary solutions.

## Inputs

- g2myparms.m parameter set (grid extents, sound speed, bar strength and pattern speed)
- Parameters listed in ./paper/paper.pdf
- Initial homogeneous flow state on a Cartesian 2-D grid

## Outputs

- Stationary solution of the 2-D compressible isothermal Euler equations in a galactic potential with a rotating weak bar
- Verification metrics in solver_test_results (max norm of the residual, conservation error)
- Paper figures via figures4paper/makefigs

## How to apply

Retrieve the galax2d reference repository (github.com/Someone789/galax2d) and inspect g2myparms.m to extract the parameter set of the 2-D isothermal Euler model (grid extents, sound speed, bar strength and pattern speed). Implement the isothermal closure p = c_s^2 * rho in the density and momentum flux functions, then add the source terms: gravitational acceleration from the axisymmetric galactic potential plus the perturbation of a rotating weak bar evaluated at the bar pattern speed. Discretize with a conservative finite-volume update on a Cartesian 2-D grid, computing numerical fluxes at cell faces and advancing with an explicit time step satisfying a CFL condition, and enforce outflow/fixed boundary conditions so the bar-driven flow is well posed. Alternatively run the packaged Matlab entry point: start Matlab 8.0 (R2012b) or later, change directory to run, and start g2test; edit g2myparms.m to change the parameters listed in ./paper/paper.pdf. Paper figures can be recreated from figures4paper/ by running makefigs.

## Related tools

- **galax2d** (Reference Matlab implementation of the 2-D isothermal Euler solver with galactic-potential and rotating-bar source terms; run entry point g2test) — https://github.com/Someone789/galax2d
- **Matlab 8.0 (R2012b)** (Runtime used to execute g2test and makefigs from the packaged run and figures4paper directories)

## Examples

```
cd run; g2test;
```

## Evaluation signals

- Initializing a homogeneous state and verifying the discrete update preserves it (flux/source balance).
- A short integration reduces the residual of the stationary equations.
- Max norm of the stationary-equation residual and conservation error written to solver_test_results fall within expected tolerances.
- Recreated figures from figures4paper/makefigs match those in ./paper/paper.pdf.

## Limitations

- The run takes one or two hours depending on the Matlab version and hardware.
- Parameters must be edited in g2myparms.m to match those listed in ./paper/paper.pdf before reproduction.
- No changelog is available for the repository.

## Evidence
