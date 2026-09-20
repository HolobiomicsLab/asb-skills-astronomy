---
name: sde-solver-configuration-reversible-heun
description: 'Use when training or sampling a neural SDE (e.g. an SDE-GAN generator)
  in JAX/Diffrax and you need a memory-reversible stochastic solver matched to the
  paper''s recipe: reversible Heun with unit step size.'
when_to_use_negative:
- The model is a neural CDE or ODE discriminator component — reduce the CDE to an
  ODE and use a standard ODE solver instead of a stochastic solver.
- Data are real-world irregularly sampled series requiring logsignature preprocessing
  and rectilinear/Hermite-cubic interpolation, not Brownian-motion-like SDE modelling.
- You need high-accuracy offline SDE integration — reversible Heun with unit step
  size is a training-oriented, fixed-step choice, not an adaptive high-order solver.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: Diffrax
  role: Provides ReversibleHeun SDE solver and Brownian path objects (UnsafeBrownianPath,
    VirtualBrownianTree) used to drive and solve the neural SDE generator
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which Diffrax and the neural SDE generator are implemented, providing
    autodifferentiation through the SDE solve
  repo: http://github.com/google/jax
- name: Equinox
  role: Used to define the neural SDE generator and neural CDE discriminator models
    that Diffrax solves
  repo: https://github.com/patrick-kidger/equinox
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:b31389eb64cdb23c9b6cb5ae1252de4e5f337f8085ff8ab70d253dc3f28e1ccc
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sde-solver-configuration-reversible-heun

## Summary

Configure Diffrax to solve neural SDEs with the reversible Heun method at unit step size, as used for the neural SDE-GAN generator trained to match Brownian-motion time series (Figure 4.3).

## When to use

Use when training or sampling a neural SDE (e.g. an SDE-GAN generator) in JAX/Diffrax and you need a memory-reversible stochastic solver matched to the paper's recipe: reversible Heun with unit step size.

## When NOT to use

- The model is a neural CDE or ODE discriminator component — reduce the CDE to an ODE and use a standard ODE solver instead of a stochastic solver.
- Data are real-world irregularly sampled series requiring logsignature preprocessing and rectilinear/Hermite-cubic interpolation, not Brownian-motion-like SDE modelling.
- You need high-accuracy offline SDE integration — reversible Heun with unit step size is a training-oriented, fixed-step choice, not an adaptive high-order solver.

## Inputs

- Normalised Brownian-motion training time series (zero mean, unit variance)
- Neural SDE generator model defined in Equinox
- Brownian control object: diffrax.UnsafeBrownianPath or diffrax.VirtualBrownianTree
- Trained generator parameters for sampling paths

## Outputs

- Solved neural SDE trajectories (t, y) sample paths
- Trained generator and discriminator parameter state
- Saved figure of generator sample paths compared with Brownian motion (cf. Figure 4.3)

## How to apply

Build the neural SDE generator as an Equinox model driven by a Brownian control (diffrax.UnsafeBrownianPath or diffrax.VirtualBrownianTree), then solve it with diffrax.ReversibleHeun using unit step size, as specified in the thesis workflow. The reversible Heun scheme is chosen because it admits cheap reversible stepping, enabling backpropagation through the SDE solve; applied to ODEs its two-step update coincides with the leapfrog/midpoint method and is second order. For the GAN setting, pair the generator with a neural CDE discriminator with Lipschitz-constrained weights and train both via simultaneous gradient descent with RMSprop, normalising training series to zero mean and unit variance. The same reversible Heun + unit step size configuration is applied both during adversarial training and when generating final sample paths for the saved figure.

## Related tools

- **Diffrax** (Provides ReversibleHeun SDE solver and Brownian path objects (UnsafeBrownianPath, VirtualBrownianTree) used to drive and solve the neural SDE generator) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which Diffrax and the neural SDE generator are implemented, providing autodifferentiation through the SDE solve) — http://github.com/google/jax
- **Equinox** (Used to define the neural SDE generator and neural CDE discriminator models that Diffrax solves) — https://github.com/patrick-kidger/equinox

## Examples

```
import diffrax; term = diffrax.OS-term(bm); sol = diffrax.diffeqsolve(diffrax.MultiTerm(term), diffrax.ReversibleHeun(), t0=0, t1=1, dt0=1.0, y0=y0, saveat=diffrax.SaveAt(ts=True))
```

## Evaluation signals

- Generated (t, y) sample paths visually match the reference Brownian-motion distribution of Figure 4.3, including the initial condition.
- Solver configuration matches the reported recipe: diffrax.ReversibleHeun with unit step size.
- Normalised data statistics: each dataset has zero mean and unit variance before training.
- Saved generator/discriminator parameters reproduce sample paths on re-run (deterministic reload of trained parameters).
- Generated paths exhibit Brownian-like quadratic variation / diffusivity consistent with the training distribution.

## Limitations

- Reversible Heun is applied with unit step size, a fixed coarse step — accuracy is traded for reversibility during training, and higher-order reversible ODE solvers or hypersolvers were noted as desirable but not yet available.
- Only (t, y) coarsely-spaced samples are compared visually in Figure 4.3; no quantitative distributional metric is reported.
- The workflow is demonstrated on Brownian motion; transfer to other stochastic processes is not validated in the source.

## Evidence

- [other] Diffrax Brownian drivers named in the source card workflow: [evidence span withheld — anchor status: not_found]
