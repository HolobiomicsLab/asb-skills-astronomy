---
name: hll-riemann-solver-implementation
description: Use when a finite-volume HRSC solver for hyperbolic conservation laws
  (e.g. 1D special relativistic hydrodynamics shock-tube problems) needs a numerical
  flux at cell interfaces given qL, qR and characteristic speeds λL, λR.
when_to_use_negative:
- When a contact/shear wave must be resolved sharply (HLL smears intermediate waves
  since it uses only two bounding speeds) — a two-state solver is the wrong choice
  for such interfaces
- When the PDE system is not hyperbolic in conservative form (aztekas targets hyperbolic
  PDEs ∂Q/∂t + ∂F^i/∂x^i = S), since the derivation relies on characteristic rays
  and Rankine-Hugoniot jumps
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: aztekas
  role: HRSC finite-volume solver for hyperbolic conservative PDEs (relativistic and
    non-relativistic Euler equations) in which this HLL flux routine is embedded for
    interface flux computation
  repo: https://github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:d089f3d2d0ae9c8ddebcd55de293bfd8ca5b72ffa818eb54d86d0bb132b115ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - aztekas
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hll-riemann-solver-implementation

## Summary

Implements the two-state HLL approximate Riemann numerical flux (Eqs 44–46 of the PVRS relativistic hydrodynamics paper), computing interface fluxes from left/right conservative states, their fluxes, and admissible bounding wave speeds, with upwind selection logic.

## When to use

Use when a finite-volume HRSC solver for hyperbolic conservation laws (e.g. 1D special relativistic hydrodynamics shock-tube problems) needs a numerical flux at cell interfaces given qL, qR and characteristic speeds λL, λR.

## When NOT to use

- When a contact/shear wave must be resolved sharply (HLL smears intermediate waves since it uses only two bounding speeds) — a two-state solver is the wrong choice for such interfaces
- When the PDE system is not hyperbolic in conservative form (aztekas targets hyperbolic PDEs ∂Q/∂t + ∂F^i/∂x^i = S), since the derivation relies on characteristic rays and Rankine-Hugoniot jumps

## Inputs

- Left and right conservative states qL, qR (e.g. relativistic Sod shock-tube initial states from Table 1)
- Left and right physical fluxes fL = f(qL), fR = f(qR)
- Bounding characteristic wave speeds λL and λR (admissible, CFL-satisfying estimates)

## Outputs

- Per-component numerical flux vector faHLL for the cell interface (Eq 45)
- Optionally the HLL intermediate state qaHLL (Eq 44)

## How to apply

Implement Eq 45, fHLL = (λR fL − λL fR + λR λL (qR − qL)) / (λR − λL), per component as a callable taking left/right states qL, qR, fluxes fL, fR, and wave speeds λL, λR; optionally also expose the Eq 44 intermediate state qaHLL = (λR qaR − λL qaL + faL − faR)/(λR − λL). Add the Eq 46 upwind selection: return faL when 0 ≤ λL, faR when 0 ≥ λR, and the HLL flux when λL < 0 < λR. Bound the speeds with an admissible estimate satisfying the CFL condition λΔt/Δx < 1 (e.g. relativistic Sod test states from Table 1). The HLL form is derived by integrating the hyperbolic equation over the space-time domain using the Rankine-Hugoniot jump condition at each characteristic ray, so it preserves consistency with the conservative finite-volume update used in aztekas.

## Related tools

- **aztekas** (HRSC finite-volume solver for hyperbolic conservative PDEs (relativistic and non-relativistic Euler equations) in which this HLL flux routine is embedded for interface flux computation) — https://github.com/aztekas-code/aztekas-main

## Examples

```
def hll_flux(qL, qR, fL, fR, lamL, lamR): return fL if lamL >= 0 else (fR if lamR <= 0 else (lamR*fL - lamL*fR + lamR*lamL*(qR-qL))/(lamR-lamL))
```

## Evaluation signals

- Flux returned for sample Riemann states matches the closed-form Eq 45 expression evaluated independently (step 4 of the task workflow)
- Upwind branch check: routine returns faL exactly when 0 ≤ λL and faR exactly when 0 ≥ λR, and HLL flux only when λL < 0 < λR
- Conservation: for symmetric states (qL = qR) the returned flux equals the physical flux f(qL)
- Wave speeds chosen satisfy the CFL condition λΔt/Δx < 1 so the finite-volume update is stable
- Sod shock-tube solution converges toward the exact analytic solution (assessed via L1-norm against exact solutions as in the paper's S1_File.tar.gz comparison data)

## Limitations

- The bounding speeds λL, λR must be admissible (enclose all signal speeds); poor estimates degrade accuracy or violate stability
- The two-state HLL solver resolves only two waves and does not capture contact discontinuities as sharply as multi-state solvers
- In relativistic hydrodynamics the overall method avoids solving a 10th degree algebraic polynomial per primitive-variable component, but the flux step itself still relies on a traditional finite-volume scheme to avoid violating the entropy and Rankine-Hugoniot jump conditions

## Evidence

- [methods] The HLL numerical flux is computed per component as fHLL = (λR fL − λL fR + λR λL (qR − qL)) / (λR − λL): [evidence span withheld — anchor status: not_found]
- [abstract] a traditional finite volume method for the flux to avoid violating entropy and Rankine-Hugoniot jump conditions: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
- [readme] aztekas solves hyperbolic PDEs in conservative form with HRSC schemes: "_aztekas_ is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
- [readme] aztekas finite-volume discretization form: "_aztekas_ uses a conservative finite-volume approach to obtain the dicrete form of a hyperbolic partial differential system of equations (PDE)."
