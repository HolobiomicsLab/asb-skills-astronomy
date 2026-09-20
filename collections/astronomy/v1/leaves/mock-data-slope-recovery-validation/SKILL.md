---
name: mock-data-slope-recovery-validation
description: Use when testing a Jeans-based mass-modeling code for graceful degradation
  with small tracer samples, or verifying that reported density-slope uncertainties
  at R1/2 grow without introducing bias in mock-data recovery tests.
when_to_use_negative:
- The goal is recovery of full mass density or anisotropy profiles rather than specifically
  the log-slope at R1/2 degradation trend
- Proper-motion data are available and the question is how PMs improve precision —
  this skill targets line-of-sight-only fits
- The stellar system is not in equilibrium or lacks a mock dataset with a known true
  slope to validate against
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: GravSphere2
  role: Performs the bin-free higher-order Jeans fit specified by gravsphere2.py (likelihood
    and priors) on each tracer subsample
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler used to sample the posterior of the slope at R1/2, run in parallel
    via run_multcpu_example.py with checkpoint/resume support
  repo: https://github.com/dadams42/GravSphere2
- name: numpy
  role: Tabulates slope error and credible-interval width versus tracer count and
    writes subsample_slope_recovery.csv
- name: matplotlib
  role: Plots the degradation trend of slope recovery error across subsample sizes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:c9ee2247b90df21090698d8726244d55627ea6e66b8d0c3e76121034b6541df8
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mock-data-slope-recovery-validation

## Summary

Validates that GravSphere2's recovered logarithmic density slope at R1/2 remains unbiased as tracer count drops from 1000 to 100 to 10, by running line-of-sight-only dynesty fits on subsamples of a Fornax-like simulated dwarf galaxy and tabulating posterior error growth.

## When to use

Use when testing a Jeans-based mass-modeling code for graceful degradation with small tracer samples, or verifying that reported density-slope uncertainties at R1/2 grow without introducing bias in mock-data recovery tests.

## When NOT to use

- The goal is recovery of full mass density or anisotropy profiles rather than specifically the log-slope at R1/2 degradation trend
- Proper-motion data are available and the question is how PMs improve precision — this skill targets line-of-sight-only fits
- The stellar system is not in equilibrium or lacks a mock dataset with a known true slope to validate against

## Inputs

- Fornax-like simulated dwarf-galaxy dataset (tracer positions and line-of-sight velocities)
- initialise_ files for N = 1000, 100, and 10 tracers with the no-proper-motion configuration selected

## Outputs

- Posterior medians, statistical errors, and 95% credible-interval widths for the logarithmic density slope at R1/2 at each tracer count
- subsample_slope_recovery.csv tabulating slope error and credible-interval width versus N = 1000, 100, 10
- Checkpoint files in per-galaxy Sampler Chains subfolders for resumable runs

## How to apply

Create initialise_ files for the Fornax-like simulated dwarf galaxy containing 1000, 100, and 10 tracers, selecting the no-proper-motion configuration in each. Import each initialise_ file into gravsphere2.py (which specifies the bin-free higher-order Jeans likelihood and priors) and set the Output directory to auto-create per-galaxy folders with Plots and Sampler Chains subfolders. Run the dynesty nested sampler (dynesty 2.1.5) per subsample via run_multcpu_example.py for parallel execution, storing checkpoint files in the Sampler Chains subfolders so runs can be resumed. Reload each checkpoint, extract the posterior for the logarithmic density slope at R1/2, and compute the posterior median, statistical error, and 95% credible-interval width at each tracer count. Tabulate slope error and credible-interval width versus N = 1000, 100, 10 with numpy and save as subsample_slope_recovery.csv.

## Related tools

- **GravSphere2** (Performs the bin-free higher-order Jeans fit specified by gravsphere2.py (likelihood and priors) on each tracer subsample) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler used to sample the posterior of the slope at R1/2, run in parallel via run_multcpu_example.py with checkpoint/resume support) — https://github.com/dadams42/GravSphere2
- **numpy** (Tabulates slope error and credible-interval width versus tracer count and writes subsample_slope_recovery.csv)
- **matplotlib** (Plots the degradation trend of slope recovery error across subsample sizes)

## Examples

```
python run_multcpu_example.py  # with initialise_ file imported in gravsphere2.py specifying the 1000/100/10-tracer no-PM subsample and Output directory
```

## Evaluation signals

- Slope statistical errors grow from ~12% (cuspy) / ~25–30% (cored) at 1000 tracers to ~30% (20% cored) at 100 tracers, and become large at 10 tracers
- The true slope value lies within the 95% credible interval at every N (no induced bias)
- Each per-galaxy Output folder contains Plots and Sampler Chains subfolders with a resumable checkpoint file
- subsample_slope_recovery.csv contains one row per N = 1000, 100, 10 with median, statistical error, and 95% CI width
- Rerunning from a saved checkpoint reproduces the same posterior without restarting the nested sampler

## Limitations

- Uncertainties grow substantially at very low tracer counts (10 tracers), limiting precision though not inducing bias
- With 1000 tracers and no PMs, errors are 12% (cuspy) / 25% (cored); distinguishing cuspy from cored is marginal without proper motions
- Runs depend on specific verified versions (python 3.10.12, numpy 1.26.4, scipy 1.15.1, dynesty 2.1.5, corner 2.2.2)

## Evidence

- [abstract] As the number of tracers is lowered (even down to just ten tracers) it gracefully degrades, with larger uncertainties but no induced bias: "As the number of tracers is lowered (even down to just ten tracers) it gracefully degrades, with larger uncertainties but no induced bias"
- [abstract] Using 1000 tracers without PMs, G RAV S PHERE 2 recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data: "Using 1000 tracers without PMs, G RAV S PHERE 2 recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data"
- [abstract] With only 100 tracers and no PMs, we were still able to recover slopes with ∼ 30%(20%) errors.: "With only 100 tracers and no PMs, we were still able to recover slopes with ∼ 30%(20%) errors."
- [methods] This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [methods] We have included the examples addressed in Bañares-Hernández et al. featuring a Fornax-like simulated galaxy and the Gaia Challenge mocks: "We have included the examples addressed in Bañares-Hernández et al. featuring a Fornax-like simulated galaxy and the Gaia Challenge mocks"
- [intro] dynesty version 2.1.5: "dynesty version 2.1.5"
