---
name: relativistic-hydrodynamics-flux-computation
description: Use when solving hyperbolic conservative PDEs (e.g. relativistic Euler
  equations) on a finite-volume grid and a per-interface numerical flux f_HLL must
  be computed from left/right states qL, qR, their fluxes fL, fR, and bounding wave
  speeds λL, λR.
when_to_use_negative:
- When the problem is not hyperbolic-conservative or lacks a well-defined flux vector
  F^i (e.g. diffusion-dominated equations)
- When bounding speeds λL, λR are not admissible or violate the CFL condition λΔt/Δx
  < 1
- When recovering primitive variables u(q) analytically is trivial and a standard
  Riemann solver is already available at no extra cost — note PVRS exists precisely
  because relativistic recovery otherwise requires solving a 10th-degree polynomial
  per component
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3474
tools:
- name: aztekas
  role: Reference HRSC finite-volume code implementing the PVRS scheme and HLL flux
    for relativistic hydrodynamics; provides Shock-Tube simulation setup and compilation
    workflow
  repo: https://github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:a1843096a4d797343c123b10fb74770533477b50132261c27284f40400c369d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - aztekas
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# relativistic-hydrodynamics-flux-computation

## Summary

Computes interface numerical fluxes for hyperbolic conservative systems (e.g. 1D special relativistic hydrodynamics) using the HLL two-state approximate Riemann solver with upwind flux selection, as implemented in the aztekas finite-volume HRSC code.

## When to use

Use when solving hyperbolic conservative PDEs (e.g. relativistic Euler equations) on a finite-volume grid and a per-interface numerical flux f_HLL must be computed from left/right states qL, qR, their fluxes fL, fR, and bounding wave speeds λL, λR.

## When NOT to use

- When the problem is not hyperbolic-conservative or lacks a well-defined flux vector F^i (e.g. diffusion-dominated equations)
- When bounding speeds λL, λR are not admissible or violate the CFL condition λΔt/Δx < 1
- When recovering primitive variables u(q) analytically is trivial and a standard Riemann solver is already available at no extra cost — note PVRS exists precisely because relativistic recovery otherwise requires solving a 10th-degree polynomial per component

## Inputs

- Left and right conservative/Riemann states qL, qR at a cell interface
- Physical fluxes fL, fR evaluated at qL and qR
- Bounding characteristic (wave) speeds λL and λR, admissible under the CFL condition λΔt/Δx < 1

## Outputs

- Per-component HLL numerical flux vector f_HLL at each cell interface
- Flux-selection decision (left-flux, right-flux, or HLL flux) per interface

## How to apply

Implement Eq 44, the HLL intermediate state qaHLL = (λR qaR − λL qaL + faL − faR)/(λR − λL), and Eq 45, the numerical flux faHLL = (λR faL − λL faR + λR λL (qaR − qaL))/(λR − λL), as a callable taking qL, qR, fL, fR, λL, λR. Add the Eq 46 upwind selection: return faL when 0 ≤ λL, faR when 0 ≥ λR, and the HLL flux when λL < 0 < λR. Derive the formula by integrating the hyperbolic equation over the space-time cell and applying the Rankine-Hugoniot jump condition at each characteristic ray. Evaluate on sample Riemann states (e.g. relativistic Sod shock-tube states of Table 1) with bounding speeds satisfying the CFL condition λΔt/Δx < 1, and cross-check the callable output against an independent closed-form evaluation of Eq 45. This finite-volume flux avoids violating the entropy and Rankine-Hugoniot jump conditions, while time evolution proceeds by forward finite differences in the PVRS scheme.

## Related tools

- **aztekas** (Reference HRSC finite-volume code implementing the PVRS scheme and HLL flux for relativistic hydrodynamics; provides Shock-Tube simulation setup and compilation workflow) — https://github.com/aztekas-code/aztekas-main

## Examples

```
cd SIMULATIONS/HD/Shock-Tube && make clean && make && ./aztekas shock.param
```

## Evaluation signals

- Callable HLL flux output matches the closed-form Eq 45 expression (λR fL − λL fR + λR λL (qR − qL))/(λR − λL) evaluated independently on the same states
- Flux-selection logic returns faL when 0 ≤ λL, faR when 0 ≥ λR, and the HLL flux only when λL < 0 < λR
- Convergence verified against exact analytic shock-tube solutions via L1-norm of fluctuations of numerical values relative to the exact solution (supplementary S1_File.tar.gz data)
- Bounding speeds satisfy the CFL condition λΔt/Δx < 1 for the chosen Δt, Δx
- Numerical scheme does not violate the entropy or Rankine-Hugoniot jump conditions

## Limitations

- Bounding wave speeds must be admissible and satisfy the CFL condition; poorly estimated λL, λR degrade accuracy or stability
- HLL uses only two states and two bounding rays, so it is a two-state approximation and may smear contact discontinuities compared with multi-wave solvers
- The PVRS time update relies on forward finite differences, so temporal accuracy is bounded by first order unless higher-order time integration is added

## Evidence

- [methods] HLL flux formula per component derived via Rankine-Hugoniot: [evidence span withheld — anchor status: not_found]
- [abstract] Finite-volume flux preserves entropy and Rankine-Hugoniot conditions: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
- [intro] PVRS avoids costly primitive-variable recovery in relativistic hydrodynamics: "at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u"
- [readme] aztekas is a conservative finite-volume HRSC solver: "aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes"
- [supplementary] Convergence validated via L1-norm against exact solutions: "In the file “S1_File.tar.gz”, we attached the relevant data of the comparison between numerical simulations and exact solutions used to obtain the L1—norm."
