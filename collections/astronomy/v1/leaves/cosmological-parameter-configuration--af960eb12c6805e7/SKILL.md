---
name: cosmological-parameter-configuration
description: Use when a cosmological analysis requires setting up a Boltzmann-solver
  model with explicit cosmological parameters (H0, Ωm, Ωb, w0, wa, c_s^2, lmax, reionization)
  that must be reproduced identically across two codes.
when_to_use_negative:
- When the goal is only to read or post-process existing spectra — this skill is about
  model setup and compilation, not spectral analysis
- When the target model differs from w0waCDM/ΛCDM and no corresponding SymBoltz component
  exists in the src/models/ library — you would first need to adapt or implement new
  components
- When CLASS is not installed/compiled (make class) and a cross-code comparison is
  intended
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3318
tools:
- name: SymBoltz
  role: Symbolic-numeric package in which the cosmological model (w0waCDM) is defined
    with symbolic equations and compiled to a numerical problem via CosmologyProblem
  repo: https://github.com/hersle/SymBoltz.jl
- name: Julia
  role: Implementation language in which the parameter dictionary is constructed and
    the model compiled/solved
  repo: https://github.com/hersle/SymBoltz.jl
- name: CLASS
  role: Reference Einstein-Boltzmann solver configured via CLASSProblem with identical
    cosmological parameter values for validation
  repo: https://github.com/lesgourg/class_public/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:fb370f955528b930c967926ce745e8540d73e30a68b74f67637bc99e57ae4597
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - CLASS
  - CairoMakie
  - DataInterpolations
  - Julia
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cosmological-parameter-configuration

## Summary

Configure a cosmological model (e.g. w0waCDM) in SymBoltz with a specific parameter set (Planck18 values plus w0, wa, c_s^2) and compile it into a numerical problem with CosmologyProblem, with an identically-parameterized CLASSProblem for cross-code comparison.

## When to use

Use when a cosmological analysis requires setting up a Boltzmann-solver model with explicit cosmological parameters (H0, Ωm, Ωb, w0, wa, c_s^2, lmax, reionization) that must be reproduced identically across two codes.

## When NOT to use

- When the goal is only to read or post-process existing spectra — this skill is about model setup and compilation, not spectral analysis
- When the target model differs from w0waCDM/ΛCDM and no corresponding SymBoltz component exists in the src/models/ library — you would first need to adapt or implement new components
- When CLASS is not installed/compiled (make class) and a cross-code comparison is intended

## Inputs

- Cosmological model definition (e.g. SymBoltz w0waCDM symbolic model)
- Parameter dictionary of Planck18 cosmological values (H0, Ωm, Ωb, etc.)
- Dark-energy equation-of-state parameters (w0 = -0.9, wa = 0.1) and sound speed c_s^2 = 0.9
- Precision/switch settings (lmax = 6, Hswitch = 1, Heswitch = 6, reionization enabled)
- CLASS input parameter set for the comparison run (output 'mPk, tCl, pCl, lCl')

## Outputs

- Compiled CosmologyProblem object (with analytical sparse Jacobian)
- Compiled CLASSProblem object with matching parameters
- Parameterized model ready for solving over a grid of wavenumbers (ks)

## How to apply

Define the model in SymBoltz (here a w0waCDM model with lmax = 6, reionization enabled, Hswitch = 1, Heswitch = 6) and set the parameter dictionary to Planck18 values augmented with w0 = -0.9, wa = 0.1, and a dark-energy sound speed c_s^2 = 0.9. Compile the symbolic model into a numerical problem via CosmologyProblem (with jac = true, sparse = true for an analytical sparse Jacobian). For a comparison run, configure CLASS through CLASSProblem with the exact same cosmological parameter values (and options such as recfast recombination, Newtonian gauge, adiabatic scalar modes, output 'mPk, tCl, pCl, lCl'), because agreement tests are only meaningful if both solvers receive identical inputs. The rationale is that SymBoltz compiles symbolic equations to fast numerical functions, so parameter assignment happens once symbolically before compilation, and identical parameterization enables the residual-based validation against CLASS.

## Related tools

- **SymBoltz** (Symbolic-numeric package in which the cosmological model (w0waCDM) is defined with symbolic equations and compiled to a numerical problem via CosmologyProblem) — https://github.com/hersle/SymBoltz.jl
- **Julia** (Implementation language in which the parameter dictionary is constructed and the model compiled/solved) — https://github.com/hersle/SymBoltz.jl
- **CLASS** (Reference Einstein-Boltzmann solver configured via CLASSProblem with identical cosmological parameter values for validation) — https://github.com/lesgourg/class_public/

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); sol = solve(prob, ks)
```

## Evaluation signals

- Compiled problem object is created without error and includes the analytical sparse Jacobian (jac = true, sparse = true)
- Parameter dictionary resolves every symbol of the model — no unset cosmological parameters remain before solving
- CLASS configuration (CLASSProblem) uses exactly the same cosmological parameter values as the SymBoltz model
- Downstream spectra computed from this configuration agree with CLASS within ~0.1% at standard precision levels (enforced by plot_compare tolerance assertion)
- Model-specific switches are as specified: lmax = 6, reionization enabled, Hswitch = 1, Heswitch = 6

## Limitations

- The 0.1% agreement with CLASS holds only at standard precision levels; higher precision requirements may need adjusted tolerance/switch settings
- More work is needed to implement additional features in SymBoltz, and fast reverse-mode automatic differentiation of scalar loss functions is not yet available
- SymBoltz solves linear perturbations only (linear Einstein-Boltzmann equations); nonlinear extensions would require other tools

## Evidence

- [methods] w0waCDM configuration parameters: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [abstract] agreement finding: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [methods] CLASS comparison setup: [evidence span withheld — anchor status: not_found]
- [methods] reference code role: "the fantastic Einstein-Boltzmann solver [CLASS](https://github.com/lesgourg/class_public/)"
- [intro] validation workflow: "plot_compare(a1, a2, τ1, τ2, "a", "τ"; tol = 6e-3)"
