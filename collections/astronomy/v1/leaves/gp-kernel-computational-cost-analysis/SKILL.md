---
name: gp-kernel-computational-cost-analysis
description: Use when evaluating whether a GP structure-learning pipeline (gallifrey,
  JAX backend) can handle larger time series, or when benchmarking runtime/memory
  scaling of SMC-based posterior approximation.
when_to_use_negative:
- The analysis goal is GP accuracy or uncertainty quality (e.g., transit parameter
  uncertainties) rather than computational cost scaling
- The data are already very large real mission time series (e.g., PLATO-scale), where
  the paper explicitly notes scalability is not yet achieved and results would be
  infeasible
- Benchmarking a non-SMC, fixed-kernel GP package where the structure-learning loop
  (SMC + involutive MCMC) does not exist to profile
tools:
- name: gallifrey
  role: Provides the SMC kernel-structure-learning loop (GPModel.fit_smc) whose runtime
    and memory are measured at each series length
  repo: https://github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend for the GP and SMC computation being benchmarked; set --xla_force_host_platform_device_count
    to the CPU count for reproducible runs
- name: Python
  role: 'Host language for the benchmark harness: data generation (numpy), timing
    (time), memory profiling (tracemalloc/memory_profiler), and log-log regression'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:36c7fae7fec42f09e2e800bbdab3aa390b29ece012158e109fb531291989ac50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/gallifrey
  tools:
  - JAX
  - Python
  - gallifrey
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gp-kernel-computational-cost-analysis

## Summary

Quantifies how the computational cost (wall-clock runtime and peak memory) of the gallifrey SMC kernel-structure-learning loop scales with time series length, to test whether scalability to large datasets (e.g., PLATO-scale) is achieved or remains an open limitation.

## When to use

Use when evaluating whether a GP structure-learning pipeline (gallifrey, JAX backend) can handle larger time series, or when benchmarking runtime/memory scaling of SMC-based posterior approximation.

## When NOT to use

- The analysis goal is GP accuracy or uncertainty quality (e.g., transit parameter uncertainties) rather than computational cost scaling
- The data are already very large real mission time series (e.g., PLATO-scale), where the paper explicitly notes scalability is not yet achieved and results would be infeasible
- Benchmarking a non-SMC, fixed-kernel GP package where the structure-learning loop (SMC + involutive MCMC) does not exist to profile

## Inputs

- Synthetic GP time series (.npz files) at a ladder of increasing lengths, e.g., 100, 250, 500, 1000, 2000 points
- gallifrey GPModel configuration: GPConfig, num_particles, LinearSchedule annealing schedule, n_mcmc and n_hmc step counts (held fixed across runs)

## Outputs

- scaling_table.csv with columns: series length, runtime (wall-clock), peak memory
- Empirical scaling exponent from log-log linear regression of cost vs. series length

## How to apply

Generate synthetic GP time series at a ladder of increasing lengths (e.g., 100, 250, 500, 1000, 2000 points) with numpy using a fixed random seed, saving each as an .npz file. For each series, initialize a gallifrey GPModel (JAX backend) with a fixed number of SMC particles and run the fit_smc loop with a fixed annealing schedule (e.g., LinearSchedule) and fixed n_mcmc/n_hmc step counts, so only series length varies. Measure wall-clock runtime with Python's time module and peak memory with tracemalloc or memory_profiler for every run. Assemble series length, runtime, and peak memory into a table (scaling_table.csv) and fit an empirical scaling exponent via log-log linear regression with numpy. Interpret the exponent against the paper's own statement that scalability to larger datasets is deferred to a future road map aimed at missions like PLATO.

## Related tools

- **gallifrey** (Provides the SMC kernel-structure-learning loop (GPModel.fit_smc) whose runtime and memory are measured at each series length) — https://github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend for the GP and SMC computation being benchmarked; set --xla_force_host_platform_device_count to the CPU count for reproducible runs)
- **Python** (Host language for the benchmark harness: data generation (numpy), timing (time), memory profiling (tracemalloc/memory_profiler), and log-log regression)

## Examples

```
gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- scaling_table.csv contains one row per series length in the ladder (e.g., 5 rows for 100–2000 points) with non-empty runtime and peak-memory values
- All runs used identical SMC settings (same num_particles, n_mcmc, n_hmc, annealing schedule), isolating series length as the only variable
- Log-log fit yields a positive empirical scaling exponent, and cost strictly increases with series length
- Runtimes are wall-clock measurements from the time module with a fixed random seed, so repeated runs are reproducible
- Peak memory measured with tracemalloc or memory_profiler exceeds a trivial baseline and grows monotonically with series length

## Limitations

- Scalability to larger datasets is not yet achieved; the authors defer algorithmic improvements for scalability to a future road map aimed at missions like PLATO
- The GP structure-learning loop uses SMC with involutive MCMC and HMC sub-steps, so cost depends on particle count and MCMC step numbers, not only series length; these must be fixed to isolate the length effect
- Measured scaling on synthetic data may not directly transfer to real astronomical time series with gaps or noise characteristics

## Evidence

- [abstract] missing signal on scalability: "We discuss a road map for algorithmic improvements in terms of scalability to larger datasets"
- [abstract] PLATO-scale motivation: "future stellar and exoplanet missions such as PLATO"
- [readme] SMC fitting loop being benchmarked: "final_smc_state, history = gpmodel.fit_smc(
    smc_key,
    annealing_schedule=annealing_schedule,
    n_mcmc=50,      # Number of MCMC steps per SMC iteration"
- [readme] fixed particle count as benchmark parameter: [evidence span withheld — anchor status: external_doc]
