---
name: runtime-scaling-benchmarking
description: Use when you need to quantify whether a tool's compute cost (runtime,
  memory) scales acceptably with dataset size, e.g. before applying gallifrey to large
  time series from future missions such as PLATO.
when_to_use_negative:
- The scientific question is about GP model accuracy or uncertainty estimation (structure
  learning vs. fixed-kernel comparison), not computational cost
- The tool already has published complexity guarantees and the goal is verifying correctness
  on a single fixed-size dataset, not characterizing scaling
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: gallifrey
  role: Target software whose SMC kernel-structure-learning loop is benchmarked across
    increasing series lengths
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend for gallifrey; backend configuration (e.g. host platform
    device count) affects runtime measurements
- name: Python
  role: Language used to generate synthetic series, time runs, profile memory, and
    fit the log-log scaling regression
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:3ceaffb25d140cf1187756111b163f0cd1f9b1ecc0c9aa10c9b12c7afcc51bd0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python
  - gallifrey
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# runtime-scaling-benchmarking

## Summary

Empirically characterizes how the compute cost of an algorithmic pipeline (here, gallifrey's JAX-based SMC kernel-structure-learning loop) grows with input size, by running it on synthetic data at a ladder of series lengths, measuring wall-clock runtime and peak memory, and fitting a log-log scaling exponent. Motivated by the paper's stated limitation that scalability to PLATO-scale datasets is deferred to a future road map.

## When to use

Use when you need to quantify whether a tool's compute cost (runtime, memory) scales acceptably with dataset size, e.g. before applying gallifrey to large time series from future missions such as PLATO.

## When NOT to use

- The scientific question is about GP model accuracy or uncertainty estimation (structure learning vs. fixed-kernel comparison), not computational cost
- The tool already has published complexity guarantees and the goal is verifying correctness on a single fixed-size dataset, not characterizing scaling

## Inputs

- Synthetic GP time series (.npz files) at increasing lengths, e.g. 100, 250, 500, 1000, 2000 points, generated with a fixed random seed
- Fixed gallifrey SMC configuration (num_particles, n_mcmc, n_hmc, annealing schedule) held constant across all runs

## Outputs

- scaling_table.csv with columns: series length, wall-clock runtime, peak memory
- Fitted empirical scaling exponent from log-log linear regression of cost vs. series length

## How to apply

Generate synthetic Gaussian process time series with numpy (fixed random seed) at a ladder of lengths, e.g. 100, 250, 500, 1000, 2000 points, and save each to an .npz file. Run gallifrey's SMC structure-learning loop (fit_smc with a LinearSchedule annealing schedule, fixed num_particles and fixed n_mcmc/n_hmc steps) on each series with the JAX backend, keeping all algorithmic settings constant across lengths so only data size varies. Measure wall-clock runtime with Python's time module and peak memory with tracemalloc or memory_profiler. Assemble a table of series length, runtime, and peak memory, then fit an empirical scaling exponent via log-log linear regression with numpy to characterize how cost grows with series length; save the result as scaling_table.csv.

## Related tools

- **gallifrey** (Target software whose SMC kernel-structure-learning loop is benchmarked across increasing series lengths) — github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend for gallifrey; backend configuration (e.g. host platform device count) affects runtime measurements)
- **Python** (Language used to generate synthetic series, time runs, profile memory, and fit the log-log scaling regression)

## Examples

```
import numpy as np, time, tracemalloc; from gallifrey.model import GPModel; from gallifrey.schedule import LinearSchedule; tracemalloc.start(); t0=time.time(); gp = GPModel(key, x=x, y=y, num_particles=8); final_state, _ = gp.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(x), 10), n_mcmc=50, n_hmc=10); print(time.time()-t0, tracemalloc.get_traced_memory())
```

## Evaluation signals

- scaling_table.csv exists with rows for every length in the ladder (e.g. 100–2000 points) and non-null runtime and peak-memory values
- A fitted log-log scaling exponent is reported with a plausible regression fit (R² close to 1 for smooth empirical scaling)
- All gallifrey runs used identical SMC settings (num_particles, n_mcmc, n_hmc) and a fixed random seed, so cost differences are attributable to series length only
- Runtime and memory are monotonically non-decreasing with series length; deviations suggest measurement noise or backend warm-up effects (e.g. JAX JIT compilation) that must be excluded

## Limitations

- Scalability to larger datasets is not yet achieved in gallifrey; the authors defer algorithmic improvements for scalability to a future road map aimed at missions like PLATO, so measured cost growth on long series is expected to be steep
- GP inference costs grow with the square of series length in principle, so the ladder must include enough lengths to fit an exponent reliably; very long series may be intractable with the current implementation
- JAX JIT compilation and device-count configuration add constant overhead that can distort per-run timing at short series lengths

## Evidence

- [other] Scalability to larger datasets identified as future work needed for missions such as PLATO: "We discuss a road map for algorithmic improvements in terms of scalability to larger datasets"
- [readme] gallifrey uses JAX and SMC for posterior approximation: "utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
- [readme] SMC fitting API used as the benchmarked workload: "final_smc_state, history = gpmodel.fit_smc(
    smc_key,
    annealing_schedule=annealing_schedule,
    n_mcmc=50,      # Number of MCMC steps per SMC iteration
    n_hmc=10,"
- [abstract] SMC kernel-structure-learning loop is the paper's core algorithm: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
