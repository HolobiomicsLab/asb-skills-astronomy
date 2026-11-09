---
name: brownian-motion-path-simulation
description: Use when a task requires generating stochastic Brownian-motion sample
  paths in JAX, or driving a neural SDE generator with a Brownian control (e.g. diffrax.UnsafeBrownianPath
  / diffrax.VirtualBrownianTree) to match a reference distribution.
when_to_use_negative:
- The task is predicting on real-world irregularly-sampled time series (e.g. the BIDMC
  datasets), which calls for neural CDEs with spline interpolation rather than Brownian
  path simulation.
- The dynamics are deterministic ODEs with no stochastic Brownian noise component,
  making SDE machinery unnecessary.
- The implementation stack is PyTorch rather than JAX, where torchsde would be the
  appropriate SDE solver library instead of Diffrax.
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Diffrax
  role: Provides the ODE/SDE/CDE solvers, ReversibleHeun method, UnsafeBrownianPath
    and VirtualBrownianTree Brownian controls used to integrate the neural SDE generator
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which the Diffrax solvers run, supplying autodifferentiation
    and GPU capability for the training loop
  repo: http://github.com/google/jax
- name: Equinox
  role: Defines the neural SDE generator and neural CDE discriminator as JAX-compatible
    model modules
  repo: https://github.com/patrick-kidger/equinox
- name: Optax
  role: Supplies the RMSprop optimiser used for simultaneous gradient descent on generator
    and discriminator
  repo: https://github.com/google-deepmind/optax
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:f780feec4a980d36a1bb1e7724d1f116cdea4e829940fcbdb9f0cb26074d4f2c
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
  - Optax
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# brownian-motion-path-simulation

## Summary

Simulate Brownian-motion (and neural-SDE-driven) sample paths in JAX using Diffrax, normalising the training series to zero mean and unit variance and integrating with the Reversible Heun solver at unit step size. This reproduces the thesis's Brownian-motion SDE-GAN experiment (Figure 4.3).

## When to use

Use when a task requires generating stochastic Brownian-motion sample paths in JAX, or driving a neural SDE generator with a Brownian control (e.g. diffrax.UnsafeBrownianPath / diffrax.VirtualBrownianTree) to match a reference distribution.

## When NOT to use

- The task is predicting on real-world irregularly-sampled time series (e.g. the BIDMC datasets), which calls for neural CDEs with spline interpolation rather than Brownian path simulation.
- The dynamics are deterministic ODEs with no stochastic Brownian noise component, making SDE machinery unnecessary.
- The implementation stack is PyTorch rather than JAX, where torchsde would be the appropriate SDE solver library instead of Diffrax.

## Inputs

- Brownian-motion training time series (coarsely-spaced (t, y) samples)
- Neural SDE generator and neural CDE discriminator parameter definitions (Equinox modules)

## Outputs

- Generated Brownian-motion-like sample paths from the trained neural SDE generator
- Saved generator and discriminator parameters
- Figure of generated sample paths comparable to Figure 4.3

## How to apply

First generate and preprocess the Brownian-motion training time series, normalising each dataset to zero mean and unit variance. Implement the generator as a neural SDE driven by a Brownian control (diffrax.UnsafeBrownianPath or diffrax.VirtualBrownianTree) and solve it with diffrax.ReversibleHeun at unit step size; Reversible Heun is second order, whose two-step update coincides with the leapfrog/midpoint method. Define models with Equinox and train generator and discriminator (a neural CDE with Lipschitz-constrained weights) jointly via simultaneous gradient descent using the RMSprop optimiser from Optax. Save the trained generator and discriminator parameters, generate sample paths from the trained generator, and save them as a figure. Validate by comparing the generated (t, y) sample paths against the reported reference distribution (Figure 4.3) and confirming they visually match, including the initial condition.

## Related tools

- **Diffrax** (Provides the ODE/SDE/CDE solvers, ReversibleHeun method, UnsafeBrownianPath and VirtualBrownianTree Brownian controls used to integrate the neural SDE generator) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which the Diffrax solvers run, supplying autodifferentiation and GPU capability for the training loop) — http://github.com/google/jax
- **Equinox** (Defines the neural SDE generator and neural CDE discriminator as JAX-compatible model modules) — https://github.com/patrick-kidger/equinox
- **Optax** (Supplies the RMSprop optimiser used for simultaneous gradient descent on generator and discriminator) — https://github.com/google-deepmind/optax

## Examples

```
import diffrax; solver = diffrax.ReversibleHeun(); bm = diffrax.UnsafeBrownianPath(shape=(1,))
```

## Evaluation signals

- Generated sample paths visually match the reference Brownian-motion distribution (Figure 4.3), including the initial condition.
- Training data are normalised to zero mean and unit variance before use.
- The neural SDE is integrated with diffrax.ReversibleHeun at unit step size, not an adaptive solver.
- Trained generator and discriminator parameters are persisted and sample-path generation succeeds from the saved generator.
- Discriminator weights satisfy a Lipschitz constraint throughout joint RMSprop training.

## Limitations

- Only unit step size is prescribed for Reversible Heun; higher-order reversible ODE solvers and hypersolvers during training are noted as desirable but not yet available.
- Validation is visual comparison against Figure 4.3, not a quantitative distribution metric.
- Software version numbers are not stated for any tool, so environment reproducibility is limited.
- Optimise-then-discretise backpropagation was needed in related experiments simply to avoid running out of memory on long time series.

## Evidence

- [discussion] Reversible Heun with unit step size: "solved using the reversible Heun method (Section 5.3.2.2
), with unit step size"
- [discussion] zero mean unit variance normalisation: "Each dataset is normalised to have zero mean and unit variance."
- [discussion] software stack for the experiment: "implemented using the JAX, Equinox, Diffrax, and Optax software libraries"
- [discussion] Diffrax example availability: [evidence span withheld — anchor status: not_found]
- [readme] Diffrax solver coverage: "ODE/SDE/CDE (ordinary/stochastic/controlled) solvers;"
