---
name: blast-wave-initial-condition-configuration
description: Use when setting up 1D special relativistic shock-tube/blast-wave tests
  on the domain [0, 1] with a discontinuity at x = 0.5, needing Table 1 left/right
  states for L1-error convergence checks.
when_to_use_negative:
- The problem is not a 1D Riemann-type blast wave — e.g. multi-dimensional, smooth
  (manufactured-solution) convergence tests, or setups without a discontinuous initial
  split.
- No exact analytical solution exists for the chosen initial states, so the L1-norm
  comparison against the S1_File.tar.gz reference solutions cannot be performed.
- You only need to test flux/limiter machinery in isolation (e.g. a single flux-sweep)
  rather than configure and run full shock-tube initial-value problems.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: aztekas
  role: Solves the configured relativistic shock-tube initial-value problem with HRSC
    finite-volume schemes (HLL Riemann solver, minmod limiter, MoL-RK4) for the PVRS
    scheme
  repo: https://github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:2c06027f62b5efc8bc987f042cae70c2c10c93073a076c71db0c75839e697717
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - aztekas
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# blast-wave-initial-condition-configuration

## Summary

Configures discontinuous left/right initial states for the three relativistic Sod blast-wave tests (weak, mildly relativistic, strong) in the aztekas solver, placing the initial discontinuity at x = 0.5 on [0, 1] so numerical runs can be scored against exact Riemann solutions for L1-error convergence assessment.

## When to use

Use when setting up 1D special relativistic shock-tube/blast-wave tests on the domain [0, 1] with a discontinuity at x = 0.5, needing Table 1 left/right states for L1-error convergence checks.

## When NOT to use

- The problem is not a 1D Riemann-type blast wave — e.g. multi-dimensional, smooth (manufactured-solution) convergence tests, or setups without a discontinuous initial split.
- No exact analytical solution exists for the chosen initial states, so the L1-norm comparison against the S1_File.tar.gz reference solutions cannot be performed.
- You only need to test flux/limiter machinery in isolation (e.g. a single flux-sweep) rather than configure and run full shock-tube initial-value problems.

## Inputs

- Left/right initial states for the three relativistic Sod tests from Table 1 of the paper
- aztekas Shock-Tube simulation template (SIMULATIONS/HD/Shock-Tube) with its shock.param parameter file
- Exact analytical solutions packaged in S1_File.tar.gz (reference deposit)

## Outputs

- shock.param-style configuration files for each blast-wave test at each resolution (Δx = 1/200 through 1/6400)
- Numerical solution profiles per test/resolution on the domain [0, 1]
- L1-norm errors and per-resolution convergence orders computed as log(error_i/error_{i-1})/log(1/2)

## How to apply

Clone aztekas and use the `setup` script in a fresh directory (or edit the SIMULATIONS/HD/Shock-Tube template and its shock.param file) to write the initial conditions. Select one of the three relativistic Sod tests (weak, mildly relativistic, strong blast wave) and set the left/right states from Table 1, with the discontinuity at x = 0.5 on the domain [0, 1] and N = 800 baseline cells. Generate the resolution sweep Δx = 1/200, 1/400, 1/800, 1/1600, 1/3200, 1/6400, keeping a fixed time step per run chosen sufficiently smaller than the CFL condition (Eq 40). Pair the initial conditions with the PVRS discretisation using an HLL Riemann solver, a minmod limiter, and MoL-RK4 time integration. The rationale is that exact analytical Riemann solutions (S1_File.tar.gz) exist for these configurations, enabling L1-norm error computation and convergence-order estimates log(error_i/error_{i-1})/log(1/2).

## Related tools

- **aztekas** (Solves the configured relativistic shock-tube initial-value problem with HRSC finite-volume schemes (HLL Riemann solver, minmod limiter, MoL-RK4) for the PVRS scheme) — https://github.com/aztekas-code/aztekas-main

## Examples

```
cd SIMULATIONS/HD/Shock-Tube && make clean && make && ./aztekas shock.param
```

## Evaluation signals

- L1-norm error decreases monotonically as resolution increases from Δx = 1/200 to Δx = 1/6400.
- Convergence order log(error_i/error_{i-1})/log(1/2) attains first order for all three tests at least at one resolution.
- Initial discontinuity sits at x = 0.5 on [0, 1] with N = 800 baseline cells, and the fixed time step is chosen sufficiently smaller than the CFL condition (Eq 40).
- Numerical profiles fluctuate around the exact analytic solutions supplied in S1_File.tar.gz (the paper's graphical error visualisation).
- aztekas compiles cleanly in SIMULATIONS/HD/Shock-Tube (`make clean`, `make`) and executes `./aztekas shock.param` without errors.

## Limitations

- The demonstrated convergence is only first order (the minmod limiter is TVD and degrades to first order at discontinuities); higher-order accuracy is not claimed.
- Stability requires a fixed time step chosen sufficiently smaller than the CFL condition (Eq 40); violating this breaks the runs.
- Validated only on 1D special relativistic hydrodynamics for a perfect fluid on a fixed background metric (Schwarzschild, Minkowski, Kerr-Schild) — not generalised to other equation sets in this configuration.

## Evidence

- [methods] Set up the three relativistic Sod tests (weak, mildly relativistic, and strong blast waves) with the initial discontinuity at x = 0.5 on the domain [0, 1]: [evidence span withheld — anchor status: not_found]
- [methods] Compute the L1-norm error of each run against the exact analytical solutions provided in the S1_File.tar.gz reference deposit.: [evidence span withheld — anchor status: not_found]
- [readme] _aztekas_ is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes.: "aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
- [readme] Move to SIMULATIONS/HD/Shock-Tube: `$ cd SIMULATIONS/HD/Shock-Tube`: "Move to SIMULATIONS/HD/Shock-Tube: `$ cd SIMULATIONS/HD/Shock-Tube`"
