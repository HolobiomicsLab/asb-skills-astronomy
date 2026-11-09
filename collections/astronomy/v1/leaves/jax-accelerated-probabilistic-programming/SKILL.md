---
name: jax-accelerated-probabilistic-programming
description: Use when a Bayesian model (e.g. GP structure learning over discrete kernel
  structures plus continuous hyperparameters) requires sampling-intensive inference
  and Python/JAX is available to JIT-compile the SMC and MCMC kernels.
when_to_use_negative:
- When a fixed covariance function is known a priori and the user explicitly specifies
  it — plain GP packages (e.g. tinygp-style regression) avoid the structure-learning
  sampler entirely
- 'When the dataset is very large: the article flags scalability to larger datasets
  (e.g. PLATO-era missions) as future algorithmic work'
- When the environment cannot satisfy the pinned dependency stack (Python >=3.10,
  jax >=0.5.0,<0.6.0, blackjax >=1.2.5,<2.0.0)
tools:
- name: gallifrey
  role: Python package that implements the SMC-based Bayesian kernel structure learning
    pipeline with involutive MCMC transitions
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: JIT-compiled numerical backend executing the SMC particle population and MCMC
    transition kernels across CPU cores
- name: blackjax
  role: Pinned MCMC dependency (>=1.2.5,<2.0.0) underlying gallifrey's sampling stack
- name: AutoGP.jl
  role: Original Julia implementation of SMC structure discovery that gallifrey re-implements
    in JAX
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:9474174e3f1fdfdfaec6c469f9004b38ebc91b314be66c8b611d033d55b4dcbe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoGP.jl
  - JAX
  - Python
  - blackjax
  - gallifrey
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# jax-accelerated-probabilistic-programming

## Summary

Implement Bayesian inference (here, SMC with involutive MCMC for GP kernel structure learning) as JIT-compiled JAX code so particle populations and MCMC transitions execute efficiently on CPU/GPU, as done in the gallifrey package for astronomical time series.

## When to use

Use when a Bayesian model (e.g. GP structure learning over discrete kernel structures plus continuous hyperparameters) requires sampling-intensive inference and Python/JAX is available to JIT-compile the SMC and MCMC kernels.

## When NOT to use

- When a fixed covariance function is known a priori and the user explicitly specifies it — plain GP packages (e.g. tinygp-style regression) avoid the structure-learning sampler entirely
- When the dataset is very large: the article flags scalability to larger datasets (e.g. PLATO-era missions) as future algorithmic work
- When the environment cannot satisfy the pinned dependency stack (Python >=3.10, jax >=0.5.0,<0.6.0, blackjax >=1.2.5,<2.0.0)

## Inputs

- Astronomical time series: paired time and flux measurements loaded with numpy/pandas
- Training x/y arrays as jnp arrays (e.g. xtrain, ytrain split from the light curve)
- GPConfig object specifying the kernel vocabulary / prior configuration
- Annealing schedule (e.g. from gallifrey.schedule.LinearSchedule)
- JAX PRNGKey for model initialization and SMC fitting

## Outputs

- Posterior particle set over GP kernel structures and hyperparameters with SMC log weights (saved to smc_kernel_samples.csv)
- final_smc_state and per-iteration history from fit_smc
- Predictive mixture distribution over test points (predictive mean and standard deviation via get_mixture_distribution)

## How to apply

Express the model with a prior over latent discrete structures (kernel vocabulary) and continuous hyperparameters, then run Sequential Monte Carlo: initialize particles from the prior, reweight by the GP marginal likelihood, adaptively resample when weights degenerate, and rejuvenate each particle with an involutive MCMC transition kernel that jointly updates discrete structure and continuous hyperparameters (gallifrey uses n_mcmc=50 MCMC steps per SMC iteration with n_hmc=10 HMC steps inside each). Drive JAX to use all CPU cores via XLA_FLAGS='--xla_force_host_platform_device_count', split PRNG keys explicitly (jr.split), and generate an annealing schedule (e.g. LinearSchedule().generate(len(xtrain), 10)) before calling fit_smc. The rationale is JAX's vectorization/JIT makes a population-based SMC sampler tractable, following the AutoGP.jl design reimplemented from Julia to Python.

## Related tools

- **gallifrey** (Python package that implements the SMC-based Bayesian kernel structure learning pipeline with involutive MCMC transitions) — github.com/ChrisBoettner/gallifrey
- **JAX** (JIT-compiled numerical backend executing the SMC particle population and MCMC transition kernels across CPU cores)
- **blackjax** (Pinned MCMC dependency (>=1.2.5,<2.0.0) underlying gallifrey's sampling stack)
- **AutoGP.jl** (Original Julia implementation of SMC structure discovery that gallifrey re-implements in JAX)

## Examples

```
final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10, verbosity=1)
```

## Evaluation signals

- The pipeline runs end-to-end in gallifrey/JAX without errors (Python >=3.10; pip install gallifrey)
- The final posterior particle set is non-degenerate after the reweight–resample–rejuvenate cycle
- fit_smc is called with a valid annealing_schedule, n_mcmc=50 and n_hmc=10, returning final_smc_state and history
- Prediction via get_mixture_distribution yields finite predictive mean and stddev over the test grid
- Qualitative check: learned structures improve accuracy and uncertainty estimation relative to fixed-kernel approaches (no numeric threshold reported in the paper)

## Limitations

- Scalability to larger datasets is identified as future work needed for missions such as PLATO
- Dependencies are tightly pinned (jax >=0.5.0,<0.6.0; flax >=0.10.3,<0.11.0; equinox >=0.11.11,<0.12.0), so version drift can break installs
- Particle weight degeneracy during SMC requires adaptive resampling, which adds scheduling decisions (annealing schedule, particle count)

## Evidence

- [abstract] SMC transitions use involutive MCMC in JAX: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [intro] JAX for numerical computation and SMC for posterior approximation: "gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
- [readme] gallifrey is a JAX-based re-implementation of AutoGP.jl: "It is a JAX-based python implementation of the julia package AutoGP.jl by Feras Saad."
- [other] Scalability to larger datasets is future work: "We discuss a road map for algorithmic improvements in terms of scalability to larger datasets"
