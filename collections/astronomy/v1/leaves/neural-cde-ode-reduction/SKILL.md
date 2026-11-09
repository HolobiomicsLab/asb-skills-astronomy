---
name: neural-cde-ode-reduction
description: Use when a neural CDE dy(t) = f_θ(y(t), dX(t)) must be solved over discrete,
  irregularly sampled time-series data, and the only available numerical machinery
  is an ODE solver rather than a native CDE integrator.
when_to_use_negative:
- The dynamics are stochastic rather than controlled — neural SDEs should instead
  be solved with the reversible Heun method with unit step size, not reduced to an
  ODE
- The data's measurability is insufficient for spline interpolation — use rectilinear
  interpolation directly rather than forcing Hermite cubic splines
- The model is a residual or recurrent network, which the article notes are already
  discretisations of ODEs/CDEs and do not need a continuous solve
edam_operation: null
tools:
- name: Diffrax
  role: JAX-based ODE/CDE solver used to integrate the reduced ODE over [t0, t1] and
    to supply adjoint backpropagation (RecursiveCheckpointAdjoint, BacksolveAdjoint)
  repo: https://github.com/patrick-kidger/diffrax
- name: torchcde
  role: Prototype PyTorch library of differentiable GPU-capable CDE solvers; for new
    projects the author recommends Diffrax instead
  repo: https://github.com/patrick-kidger/torchcde
- name: JAX
  role: Autodifferentiable numerical framework in which the reduced ODE solve is expressed;
    must run at 64-bit precision
  repo: http://github.com/google/jax
- name: Equinox
  role: JAX neural-network library used to define the vector field f_θ of the neural
    CDE
  repo: https://github.com/patrick-kidger/equinox
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:aa215c21d73215a3ab9e3f50840262c93d8a04341e7ab53cfe6488cc2c9fbc4a
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/diffrax
  tools:
  - Diffrax
  - Equinox
  - JAX
  - pytest
  - torchcde
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# neural-cde-ode-reduction

## Summary

Solve a neural controlled differential equation (neural CDE) by reducing it to an ordinary differential equation, so that standard ODE solvers in Diffrax/torchcde can integrate the CDE dynamics defined against an interpolated control path.

## When to use

Use when a neural CDE dy(t) = f_θ(y(t), dX(t)) must be solved over discrete, irregularly sampled time-series data, and the only available numerical machinery is an ODE solver rather than a native CDE integrator.

## When NOT to use

- The dynamics are stochastic rather than controlled — neural SDEs should instead be solved with the reversible Heun method with unit step size, not reduced to an ODE
- The data's measurability is insufficient for spline interpolation — use rectilinear interpolation directly rather than forcing Hermite cubic splines
- The model is a residual or recurrent network, which the article notes are already discretisations of ODEs/CDEs and do not need a continuous solve

## Inputs

- Discrete time-series observations (irregularly sampled, per-channel values)
- A continuous control path X(t) built by interpolating the data (logsignature windows, Hermite cubic splines with backward differences, or rectilinear interpolation)
- Neural vector field f_θ(y, x) with parameters θ (Equinox/JAX module or torchcde module)
- Initial hidden state y(0) and integration interval [t0, t1]

## Outputs

- Solution trajectory y(t) over [t0, t1] obtained from the reduced ODE solve
- Gradients of the solution with respect to y(0) and θ, computed by discretise-then-optimise (e.g. RecursiveCheckpointAdjoint) or a backwards-in-time adjoint CDE (optimise-then-discretise)
- Per-parameter relative gradient differences in adjoint_gradient_comparison.csv (when used for adjoint validation)

## How to apply

First convert the discrete observations into a continuous control path X(t) — the article's experiments do this by 'preprocessing the data into logsignatures, calculated over each window' and interpolating into a piecewise-linear path; Hermite cubic splines with backward differences are recommended for smoothness and measurability, with rectilinear interpolation when measurability is insufficient. Then compute the control's time derivative dX/dt, treat the CDE dy(t) = f_θ(y(t), dX(t)) as the augmented ODE dy/dt = f_θ(y(t), dX/dt(t)), and integrate it over [t0, t1] with a standard ODE solver such as Diffrax's Tsit5. Run JAX at 64-bit precision rather than the 32-bit default, since the reduced solve feeds gradient computations that must agree with discretise-then-optimise references. Rationale: this reduction is exactly how the thesis's experiments integrated neural CDEs, and Diffrax/torchcde are built so the same ODE pipeline then supports backpropagation either through the solver (RecursiveCheckpointAdjoint) or via a continuous adjoint (BacksolveAdjoint).

## Related tools

- **Diffrax** (JAX-based ODE/CDE solver used to integrate the reduced ODE over [t0, t1] and to supply adjoint backpropagation (RecursiveCheckpointAdjoint, BacksolveAdjoint)) — https://github.com/patrick-kidger/diffrax
- **torchcde** (Prototype PyTorch library of differentiable GPU-capable CDE solvers; for new projects the author recommends Diffrax instead) — https://github.com/patrick-kidger/torchcde
- **JAX** (Autodifferentiable numerical framework in which the reduced ODE solve is expressed; must run at 64-bit precision) — http://github.com/google/jax
- **Equinox** (JAX neural-network library used to define the vector field f_θ of the neural CDE) — https://github.com/patrick-kidger/equinox

## Examples

```
sol = diffrax.diffeqsolve(diffrax.ODETerm(vector_field), diffrax.Tsit5(), t0=0.0, t1=1.0, dt0=None, y0=y0)
```

## Evaluation signals

- The reduced-ODE trajectory y(t) matches a native CDE-solver reference within discretisation tolerance
- Gradients computed through the reduced ODE agree with reference discretise-then-optimise gradients (per-parameter relative differences recorded in adjoint_gradient_comparison.csv)
- A test asserting gradient agreement within discretisation tolerance passes under 'uv run pytest'
- JAX is confirmed to be running at 64-bit precision, not the 32-bit default
- The interpolation scheme applied (Hermite cubic splines with backward differences vs rectilinear) matches the measurability of the input data

## Limitations

- Neural CDEs that use natural cubic splines produce slightly worse results than other interpolation schemes
- At time of writing there does not exist a JAX library for computing logsignatures, complicating the logsignature preprocessing step
- JAX's 32-bit default precision is insufficient for accurate gradient comparisons through the reduced solve
- torchcde was the author's research prototype; for new work Diffrax is recommended as faster and production-quality

## Evidence

- [abstract] Interpolation-scheme guidance used when building the control path: [evidence span withheld — anchor status: not_found]
- [readme] torchcde README describes differentiable CDE solvers with backprop support: [evidence span withheld — anchor status: not_found]
