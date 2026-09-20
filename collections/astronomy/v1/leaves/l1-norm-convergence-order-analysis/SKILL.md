---
name: l1-norm-convergence-order-analysis
description: Use when you have numerical solutions of a hyperbolic PDE at multiple
  resolutions plus an exact analytic reference, and need to measure whether L1 error
  decreases and what convergence order the scheme attains.
when_to_use_negative:
- You lack an exact analytic reference solution — L1 norms cannot be computed without
  one
- The discretisation is non-conservative (like Eq 10 for primitive variables); the
  analysis will merely confirm convergence to the wrong weak solution rather than
  certify the scheme
- Resolution pairs do not differ by a factor of 2, since the order estimator log(error_i/error_{i-1})/log(1/2)
  assumes halved grid spacing
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: aztekas
  role: HRSC finite-volume solver used to generate the per-resolution numerical solutions
    whose L1 errors and convergence orders are measured
  repo: github.com/aztekas-code/aztekas-main
- name: S1_File.tar.gz
  role: Supplementary deposit containing the exact solution data used to obtain the
    L1-norm comparisons
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:831ed80c8d27b2fc60ce767ffa751571e3dc0e1621f841788fc29f8cc6c9b611
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - S1_File.tar.gz
  - aztekas
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# l1-norm-convergence-order-analysis

## Summary

Quantifies numerical scheme convergence by computing the L1-norm error of simulation output against an exact reference solution at successive grid resolutions and deriving the convergence order as log(error_i/error_{i-1})/log(1/2). In this paper it validates the PVRS scheme (and diagnoses the non-convergent FDM-only primitive-variable scheme) on relativistic Sod shock tubes and a static Gaussian profile.

## When to use

Use when you have numerical solutions of a hyperbolic PDE at multiple resolutions plus an exact analytic reference, and need to measure whether L1 error decreases and what convergence order the scheme attains.

## When NOT to use

- You lack an exact analytic reference solution — L1 norms cannot be computed without one
- The discretisation is non-conservative (like Eq 10 for primitive variables); the analysis will merely confirm convergence to the wrong weak solution rather than certify the scheme
- Resolution pairs do not differ by a factor of 2, since the order estimator log(error_i/error_{i-1})/log(1/2) assumes halved grid spacing

## Inputs

- Numerical solution grids from aztekas runs at resolutions Δx = 1/200 through 1/6400 (relativistic Sod weak, mildly relativistic, and strong blast-wave initial states from Table 1, or a static Gaussian profile on [0,1])
- Exact analytic reference solution data from S1_File.tar.gz (Riemann solutions or static Gaussian)

## Outputs

- Per-resolution L1-norm error table (e.g. fdm_l1_errors.csv or gaussian_convergence.csv)
- Per-resolution convergence orders log(error_i/error_{i-1})/log(1/2)
- Plots of numerical vs exact density profiles (e.g. fdm_sod_profile.png)

## How to apply

Run the aztekas solver configured for the scheme under test (PVRS with HLL Riemann solver, minmod limiter, MoL-RK4; or the FDM-only Eq (10) scheme for the negative control) at the six resolutions Δx = 1/200, 1/400, 1/800, 1/1600, 1/3200, 1/6400 on domain [0,1] with the discontinuity or profile at x = 0.5, using time steps satisfying the CFL bound of Eq (40) (λΔt/Δx < 1; Δt = 0.01 fixed for the static Gaussian). Interpolate each run onto the exact Riemann or Gaussian reference data shipped in S1_File.tar.gz and compute the L1-norm error per resolution. Derive the convergence order at each refinement as log(error_i/error_{i-1})/log(1/2). Accept PVRS results when error decreases monotonically with resolution and first-order convergence is obtained for all three shock-tube tests at least at one resolution, and about 2 for the static Gaussian; a failure of L1 error to decrease (order near zero at the shock) is the expected diagnostic for the non-conservative FDM-only scheme.

## Related tools

- **aztekas** (HRSC finite-volume solver used to generate the per-resolution numerical solutions whose L1 errors and convergence orders are measured) — github.com/aztekas-code/aztekas-main
- **S1_File.tar.gz** (Supplementary deposit containing the exact solution data used to obtain the L1-norm comparisons)

## Examples

```
cd SIMULATIONS/HD/Shock-Tube && make clean && make && ./aztekas shock.param
```

## Evaluation signals

- L1-norm error must decrease monotonically with increasing resolution for the conservative PVRS scheme
- Computed convergence order ≈ 1 for all three relativistic Sod tests in at least one resolution
- Static Gaussian convergence order must be ≈ 2 at all tested resolutions with fixed Δt = 0.01
- For the FDM-only Eq (10) control run, L1 error must fail to decrease monotonically (order near zero at the shock), reproducing 'stable but not consistent and so no convergent'
- Time step must respect the CFL bound λΔt/Δx < 1 (Eq 40) at every resolution

## Limitations

- At shock discontinuities the achievable convergence order is dominated by the discontinuity, so smooth-profile orders (~2) are not attainable on shock-tube tests (~1)
- The non-conservative FDM-only primitive-variable scheme violates entropy and Rankine-Hugoniot jump conditions and converges to a different weak solution than a conservative method, so error metrics on it diagnose failure rather than certify accuracy
- In relativistic hydrodynamics, traditional primitive recovery requires solving a 10th degree algebraic polynomial per component per time step; convergence tests do not remove this cost for non-PVRS schemes

## Evidence

- [results] The L1-norm error decreases with increasing resolution, and the PVRS scheme attains first-order convergence for all shock-tube tests at least at one resolution.: [evidence span withheld — anchor status: not_found]
- [results] Static Gaussian order ~2: [evidence span withheld — anchor status: not_found]
- [results] FDM failure on shocks: [evidence span withheld — anchor status: not_found]
- [supplementary] Exact-solution data deposit: "In the file “S1_File.tar.gz”, we attached the relevant data of the comparison between numerical simulations and exact solutions used to obtain the L1—norm."
- [readme] Solver description: "_aztekas_ is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
