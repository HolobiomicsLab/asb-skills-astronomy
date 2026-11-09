---
name: tracer-subsample-sampling
description: Use when assessing the statistical robustness of a recovered logarithmic
  density slope at R1/2 as tracer-star count drops, or when deciding the minimum N
  of line-of-sight velocities needed for a dwarf-galaxy mass model.
when_to_use_negative:
- The goal is to test binning-based Jeans codes such as justinread/gravsphere + binulator
  or AnnaGenina/pyGravSphere, which require binned pre-processed inputs rather than
  individual tracer velocities
- The dataset includes proper motions; the article's error benchmarks (12%/25% at
  1000 tracers; ~30%/20% at 100 tracers) apply to line-of-sight-only fits, and PM
  fits improve errors to 8%/12%
- The stellar system is not in dynamical equilibrium — GravSphere2 is designed to
  mass-model systems of stellar tracers in equilibrium
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: GravSphere2
  role: Runs the bin-free fourth-order Jeans fit; the initialise_ file selects the
    no-proper-motion configuration, likelihood, priors, and Output directory
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler used for posterior sampling of the slope at R1/2 for each tracer
    subsample
  repo: https://github.com/dadams42/GravSphere2
- name: numpy
  role: Tabulates slope error and 95% credible-interval width versus N = 1000, 100,
    10 into subsample_slope_recovery.csv
- name: matplotlib
  role: Used for plotting degradation-trend diagnostics alongside the numpy table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:c5a510e955e07085d4585a699856e98119da249a3ba402eb37dc2115a7b8db0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dadams42/GravSphere2
  tools:
  - GravSphere2
  - Python
  - dynesty
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# tracer-subsample-sampling

## Summary

Quantify how GravSphere2's bin-free higher-order Jeans fit degrades with fewer stellar tracers by fitting subsamples of 1000, 100, and 10 tracers from a Fornax-like simulated dwarf galaxy in line-of-sight-only (no proper motion) configuration, and checking that only uncertainties—not bias—grow.

## When to use

Use when assessing the statistical robustness of a recovered logarithmic density slope at R1/2 as tracer-star count drops, or when deciding the minimum N of line-of-sight velocities needed for a dwarf-galaxy mass model.

## When NOT to use

- The goal is to test binning-based Jeans codes such as justinread/gravsphere + binulator or AnnaGenina/pyGravSphere, which require binned pre-processed inputs rather than individual tracer velocities
- The dataset includes proper motions; the article's error benchmarks (12%/25% at 1000 tracers; ~30%/20% at 100 tracers) apply to line-of-sight-only fits, and PM fits improve errors to 8%/12%
- The stellar system is not in dynamical equilibrium — GravSphere2 is designed to mass-model systems of stellar tracers in equilibrium

## Inputs

- Fornax-like simulated dwarf-galaxy dataset with individual stellar line-of-sight velocities (no proper motions)
- initialise_ configuration files per subsample (1000, 100, 10 tracers) specifying no-proper-motion mode, Output directory, likelihood and priors
- dynesty checkpoint files in Sampler Chains subfolders

## Outputs

- Posterior samples of the logarithmic density slope at R1/2 for each tracer count
- Table of posterior median slope, statistical error, and 95% credible-interval width versus N (subsample_slope_recovery.csv)
- Degradation-trend validation report confirming no induced bias at any N

## How to apply

Create initialise_ files for the Fornax-like simulated dwarf galaxy subsampled to 1000, 100, and 10 tracers, with the no-proper-motion configuration selected, and set the Output directory to auto-create per-galaxy folders with Plots and Sampler Chains subfolders. Import each initialise_ file into gravsphere2.py, which specifies the likelihood and priors for the bin-free higher-order Jeans fit, then run the dynesty nested sampler (dynesty 2.1.5) via run_multcpu_example.py for parallel execution, storing resumable checkpoint files in the Sampler Chains subfolders. Reload each checkpoint, extract the posterior of the logarithmic density slope at R1/2, and compute the posterior median, statistical error, and 95% credible-interval width at each tracer count. Tabulate slope error and credible-interval width versus N = 1000, 100, 10 with numpy and save as subsample_slope_recovery.csv; the expected pattern is graceful degradation: errors grow (~12% cuspy / ~25-30% at 1000-100 tracers, larger at 10) while the true slope stays within the 95% credible interval at every N.

## Related tools

- **GravSphere2** (Runs the bin-free fourth-order Jeans fit; the initialise_ file selects the no-proper-motion configuration, likelihood, priors, and Output directory) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler used for posterior sampling of the slope at R1/2 for each tracer subsample) — https://github.com/dadams42/GravSphere2
- **numpy** (Tabulates slope error and 95% credible-interval width versus N = 1000, 100, 10 into subsample_slope_recovery.csv)
- **matplotlib** (Used for plotting degradation-trend diagnostics alongside the numpy table)

## Examples

```
python run_multcpu_example.py  # imports initialise_ file into gravsphere2.py and runs dynesty per subsample (N=1000/100/10), checkpoints in Sampler Chains/
```

## Evaluation signals

- Slope statistical errors grow with decreasing N: ~12% (cuspy) / ~25-30% at 1000-100 tracers without PMs, with clearly larger uncertainties at 10 tracers
- The true logarithmic density slope at R1/2 lies inside the 95% credible interval at every tracer count (N = 1000, 100, 10) — no induced bias
- Each run produces a checkpoint file in the per-galaxy Sampler Chains subfolder that can be reloaded to recover the full posterior
- subsample_slope_recovery.csv contains rows for N = 1000, 100, 10 with median slope, statistical error, and credible-interval width columns

## Limitations

- At 10 tracers the slope uncertainty becomes large, so distinguishing cuspy from cored profiles is not reliable at such low counts
- Degradation benchmarks are established for line-of-sight-only fits on mocks (Fornax-like simulated galaxy and Gaia Challenge mock data); real datasets with different anisotropy or systematics may degrade differently
-  dynesty runs are stochastic; checkpoint files should be used to resume rather than restart interrupted sampling

## Evidence

- [abstract] graceful degradation with no bias: "As the number of tracers is lowered (even down to just ten tracers) it gracefully degrades, with larger uncertainties but no induced bias"
- [abstract] error benchmarks at 1000 and 100 tracers: "Using 1000 tracers without PMs, G RAV S PHERE 2 recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data"
- [methods] initialise_ file configuration: "the Output directory is set in the same file, if you want to change this, this can be modified in the initialise_ files, where you specify the data for each galaxy"
- [methods] checkpoint storage for resumable runs: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [readme] verified dependency versions: "dynesty version 2.1.5"
