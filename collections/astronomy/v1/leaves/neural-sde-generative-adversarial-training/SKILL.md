---
name: neural-sde-generative-adversarial-training
description: Use when the goal is to generate synthetic stochastic time series (e.g.
  Brownian-motion paths) whose distribution — including initial condition — matches
  a set of observed (t, y) trajectories, via adversarial training of a neural SDE
  generator.
when_to_use_negative:
- The target data is deterministic (noise-free ODE trajectories) where a plain neural
  ODE fit suffices rather than a stochastic generator
- The task is time-series prediction/forecasting rather than distribution matching
  — a neural CDE alone (without adversarial training) is the appropriate model
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Diffrax
  role: Provides the neural SDE generator solver (diffrax.ReversibleHeun with unit
    step size) and Brownian controls (UnsafeBrownianPath / VirtualBrownianTree)
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which the Diffrax-based generator/discriminator and training
    loop are implemented
  repo: http://github.com/google/jax
- name: Equinox
  role: Neural network / model definition library for the JAX-based neural SDE and
    neural CDE
  repo: https://github.com/patrick-kidger/equinox
- name: Optax
  role: Gradient processing and optimisation library used alongside the JAX/Equinox/Diffrax
    stack for the simultaneous gradient descent
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:dfbb6205bf61365cd7f7f326ce2a93a949dd3ca02cfcbb893eaf58a7f32d8271
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# neural-sde-generative-adversarial-training

## Summary

Train a small SDE-GAN — a neural SDE generator driven by a Brownian control against a neural CDE discriminator with Lipschitz-constrained weights — to match the distribution of Brownian-motion time series including their initial condition, using Diffrax/JAX/Equinox.

## When to use

Use when the goal is to generate synthetic stochastic time series (e.g. Brownian-motion paths) whose distribution — including initial condition — matches a set of observed (t, y) trajectories, via adversarial training of a neural SDE generator.

## When NOT to use

- The target data is deterministic (noise-free ODE trajectories) where a plain neural ODE fit suffices rather than a stochastic generator
- The task is time-series prediction/forecasting rather than distribution matching — a neural CDE alone (without adversarial training) is the appropriate model

## Inputs

- Brownian-motion training time series as (t, y) samples
- Generator neural SDE parameters (Equinox/JAX pytrees)
- Discriminator neural CDE parameters (Equinox/JAX pytrees)

## Outputs

- Trained generator and discriminator parameters (saved pytrees)
- Generated sample paths from the trained generator, saved as a figure (e.g. Figure 4.3-style coarsely-spaced (t, y) samples)

## How to apply

Generate and preprocess Brownian-motion training time series, normalising each dataset to zero mean and unit variance. Implement the generator as a neural SDE driven by a Brownian control (diffrax.UnsafeBrownianPath or diffrax.VirtualBrownianTree) and solve it with diffrax.ReversibleHeun using unit step size. Implement the discriminator as a neural controlled differential equation with Lipschitz-constrained weights. Train generator and discriminator jointly via simultaneous gradient descent using the RMSprop optimiser, then save parameters and generate sample paths from the trained generator as a figure.

## Related tools

- **Diffrax** (Provides the neural SDE generator solver (diffrax.ReversibleHeun with unit step size) and Brownian controls (UnsafeBrownianPath / VirtualBrownianTree)) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which the Diffrax-based generator/discriminator and training loop are implemented) — http://github.com/google/jax
- **Equinox** (Neural network / model definition library for the JAX-based neural SDE and neural CDE) — https://github.com/patrick-kidger/equinox
- **Optax** (Gradient processing and optimisation library used alongside the JAX/Equinox/Diffrax stack for the simultaneous gradient descent)

## Examples

```
import diffrax; bm = diffrax.UnsafeBrownianPath(shape=()); sol = diffrax.diffeqsolve(diffrax.MultiTerm(diffrax.ODETerm(drift), diffrax.ControlTerm(diffusion, bm)), diffrax.ReversibleHeun(), t0=0, t1=1, dt0=1.0, y0=y0)
```

## Evaluation signals

- Generated sample paths visually match the reference distribution (Figure 4.3) when compared against real Brownian-motion (t, y) samples
- Generator output reproduces the initial-condition distribution, not just the path dynamics
- Normalisation check: training data has zero mean and unit variance before fitting
- Discriminator weights satisfy the Lipschitz constraint throughout training (RMSprop, simultaneous gradient descent stable)
- Trained generator and discriminator parameters can be saved and reloaded to regenerate consistent sample paths

## Limitations

- Optimise-then-discretise backpropagation was needed simply to avoid running out of memory on long time series
- Validation is visual/qualitative against the reference figure rather than a quantitative distributional metric
- At time of writing there does not exist a JAX library for computing logsignatures, constraining CDE preprocessing options in the JAX ecosystem

## Evidence

- [discussion] experiment available as Diffrax example: [evidence span withheld — anchor status: not_found]
