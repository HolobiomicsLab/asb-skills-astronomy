---
name: relativistic-hydrodynamics-conservative-variable-formulation
description: Use when solving relativistic hydrodynamics with discontinuous solutions
  (shock tubes, relativistic Sod problems) where the discretisation must be conservative
  to converge to the physically correct weak solution.
when_to_use_negative:
- 'When the goal is to demonstrate failure of non-conservative schemes: the pure-FDM
  primitive-variable update of Eq (10) is deliberately non-conservative and will not
  satisfy the jump conditions'
- When source terms or geometries (e.g. curved-spacetime GRHD) require modifications
  beyond the flat-space 1D SRHD charges D, Sx, tau described here
- When primitive-variable recovery is needed analytically but u(q) is unknown — use
  the PVRS chain-rule approach instead, which requires no analytical knowledge of
  u(q)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: aztekas
  role: Open-source code that solves hyperbolic PDEs in conservative form with HRSC
    schemes, implementing the conservative SRHD formulation and the PVRS technique
    of the paper
  repo: https://github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:ea6bb5654a064345a84e3c03f74a9d955929f85825ac8af1bd9168d27e83a65b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - aztekas
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# relativistic-hydrodynamics-conservative-variable-formulation

## Summary

Formulate 1D special relativistic hydrodynamics (SRHD) in conservative form, evolving the conservative charges D, Sx, tau (Eqs 2-4) with the polytropic equation of state e = nm + p/(kappa-1), so that shocks are captured by satisfying entropy and Rankine-Hugoniot jump conditions. This conservative framing underpins both the paper's PVRS scheme and the demonstration that a pure finite-difference primitive-variable scheme fails to converge at shocks.

## When to use

Use when solving relativistic hydrodynamics with discontinuous solutions (shock tubes, relativistic Sod problems) where the discretisation must be conservative to converge to the physically correct weak solution.

## When NOT to use

- When the goal is to demonstrate failure of non-conservative schemes: the pure-FDM primitive-variable update of Eq (10) is deliberately non-conservative and will not satisfy the jump conditions
- When source terms or geometries (e.g. curved-spacetime GRHD) require modifications beyond the flat-space 1D SRHD charges D, Sx, tau described here
- When primitive-variable recovery is needed analytically but u(q) is unknown — use the PVRS chain-rule approach instead, which requires no analytical knowledge of u(q)

## Inputs

- Initial Riemann (shock-tube) states for the relativistic Sod problem on [0,1] with the discontinuity at x = 0.5 (Table 1 states)
- Equation of state parameters (polytropic e = nm + p/(kappa-1) with adiabatic index kappa)
- Grid resolution (e.g. N = 800 cells, or dx = 1/200 to 1/6400 for convergence studies) and a time step satisfying CFL lambda*dt/dx < 1 (Eq 40)
- Exact Riemann solution data from S1_File.tar.gz for L1-norm error computation

## Outputs

- Discrete fields of the conservative charges (D, Sx, tau) and the corresponding primitive variables u per cell and time step
- Jacobian matrix M_ab = (dq/du)^{-1}(df/du) (Eq 9) evaluated on the grid
- L1-norm errors and convergence orders per resolution (e.g. fdm_l1_errors.csv) and numerical-vs-exact density profiles (e.g. fdm_sod_profile.png)

## How to apply

Write the 1D SRHD system as dq/dt + df/dx = 0 for the conservative charges D (rest-mass density), Sx (momentum) and tau (energy), with the equation of state e = nm + p/(kappa-1) closing the system. Build the Jacobian matrix M_ab = (dq/du)^{-1}(df/du) of Eq (9), which maps between conservative variables q and primitive variables u and is required both for flux evaluations and for the primitive-variable recovery step. Evolve the conservative system with a finite-volume flux discretisation in space and a forward finite difference in time, as the paper's PVRS does, so that the entropy and Rankine-Hugoniot jump conditions are respected across shocks. Respect the CFL stability bound lambda*dt/dx < 1 (Eq 40) when choosing the time step. For validation, compare against the exact Riemann solution using L1-norm errors and per-resolution convergence orders log(error_i/error_{i-1})/log(1/2) — the conservative formulation is what guarantees these errors decrease with resolution, unlike the non-conservative FDM-only Eq (10) which converges to a different weak solution.

## Related tools

- **aztekas** (Open-source code that solves hyperbolic PDEs in conservative form with HRSC schemes, implementing the conservative SRHD formulation and the PVRS technique of the paper) — https://github.com/aztekas-code/aztekas-main

## Examples

```
git clone https://github.com/aztekas-code/aztekas-main && cd SIMULATIONS/HD/Shock-Tube && make && ./aztekas shock.param
```

## Evaluation signals

- L1-norm error against the exact Riemann solution must decrease monotonically with resolution, with convergence orders log(error_i/error_{i-1})/log(1/2) approaching the scheme's nominal order
- Across captured shocks, entropy and Rankine-Hugoniot jump conditions must be satisfied — a signature of a conservative discretisation (the FDM-only Eq 10 fails this test)
- Time step must satisfy the CFL bound lambda*dt/dx < 1 (Eq 40) throughout the run for stability
- Evolved conservative charges (D, Sx, tau) must map back to physically admissible primitive variables (positive density and pressure, subluminal velocity)
- Converged shock solution must match the weak solution selected by a conservative method, not the different weak solution the non-conservative FDM scheme converges to

## Limitations

- The non-conservative FDM-only discretisation of the primitive-variable equation (Eq 10) is 'stable but not consistent and so no convergent' on shocks: it violates entropy and Rankine-Hugoniot conditions and converges to a different weak solution
- Traditional SRHD implementations must recover primitive variables at each time step by solving a 10th degree algebraic polynomial per component, adding computational cost as space-time resolution increases
- CFL condition lambda*dt/dx < 1 (Eq 40) restricts the admissible time step; violating it destabilises the scheme

## Evidence

- [abstract] a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
- [abstract] This numerical technique evades the recovery of the primitive vector by solving an algebraic system of equations: "This numerical technique evades the recovery of the primitive vector by solving an algebraic system of equations as it is often used and so, it generalises standard techniques"
- [intro] at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u: "at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u"
- [readme] aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes: "aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes"
- [full_text] is stable but not consistent and so no convergent: "is stable but not consistent and so no convergent"
