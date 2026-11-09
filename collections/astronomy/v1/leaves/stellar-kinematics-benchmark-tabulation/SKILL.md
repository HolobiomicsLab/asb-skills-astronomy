---
name: stellar-kinematics-benchmark-tabulation
description: Use when you need to quantify whether a sophisticated Jeans-based mass
  model (GravSphere2) recovers masses more accurately than a simple mass estimator
  on mock galaxies, especially with sparse LOS velocity samples.
when_to_use_negative:
- The system is not a stellar tracer system in equilibrium (e.g. tidally disturbed
  or out-of-equilibrium dwarfs), since GravSphere2 is designed for equilibrium mass-modeling
- You need binned dispersion-profile fitting — use the original GravSphere + binulator
  or pyGravSphere instead, which rely on binning and Virial Shape Parameters
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3399
tools:
- name: GravSphere2
  role: Runs the bin-free fourth-order Jeans fits used for the inferred mass within
    R1/2
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler used to draw posterior masses from the GravSphere2 likelihood
- name: Wolf et al.-type simple mass estimator
  role: Baseline mass estimator computed at R1/2 from the same tracer kinematics for
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:061ee88500bba7cbb47005b8ea39d3c9a7bd29a70280619e495217fabda5bb3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dadams42/GravSphere2
  tools:
  - GravSphere2
  - Wolf et al.-type simple mass estimator
  - dynesty
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# stellar-kinematics-benchmark-tabulation

## Summary

Benchmarks GravSphere2 bin-free higher-order Jeans fits against Wolf et al.-type simple mass estimators on mock dwarf galaxies by computing fractional mass-recovery errors (M_inferred - M_true)/M_true at R1/2, and tabulates side-by-side results including cases with few line-of-sight velocities.

## When to use

Use when you need to quantify whether a sophisticated Jeans-based mass model (GravSphere2) recovers masses more accurately than a simple mass estimator on mock galaxies, especially with sparse LOS velocity samples.

## When NOT to use

- The system is not a stellar tracer system in equilibrium (e.g. tidally disturbed or out-of-equilibrium dwarfs), since GravSphere2 is designed for equilibrium mass-modeling
- You need binned dispersion-profile fitting — use the original GravSphere + binulator or pyGravSphere instead, which rely on binning and Virial Shape Parameters

## Inputs

- Mock dwarf galaxy tracer kinematics datasets (e.g. Gaia Challenge mocks, Fornax-like simulated galaxy) with initialise_ files
- GravSphere2 checkpoint files from completed dynesty fits (stored in Sampler Chains subfolders)
- True masses within R1/2 for each mock

## Outputs

- mass_recovery_comparison.csv with side-by-side fractional mass-recovery errors for GravSphere2 and the simple mass estimator per mock
- Posterior 95% credible interval masses within R1/2 from GravSphere2 fits
- Wolf et al.-type simple mass estimates at R1/2 per mock

## How to apply

Load each mock galaxy via its initialise_ file into gravsphere2.py, specifying the likelihood, priors, and output directory. Run bin-free higher-order Jeans fits with dynesty nested sampling, storing checkpoint files in the Sampler Chains subfolders (use run_example_notebook.ipynb or run_multcpu_example.py); retrieve each completed fit from its checkpoint and compute the posterior 95% credible interval mass within R1/2. Independently compute the Wolf et al.-type simple mass estimator at R1/2 from the same tracer kinematics. For every mock — including runs with varying numbers of LOS velocities — calculate the fractional mass-recovery error (M_inferred - M_true)/M_true for both methods and write the side-by-side errors to mass_recovery_comparison.csv.

## Related tools

- **GravSphere2** (Runs the bin-free fourth-order Jeans fits used for the inferred mass within R1/2) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler used to draw posterior masses from the GravSphere2 likelihood)
- **Wolf et al.-type simple mass estimator** (Baseline mass estimator computed at R1/2 from the same tracer kinematics for comparison)

## Examples

```
python run_multcpu_example.py  # then: python gravsphere2.py  # reload checkpoints and write mass_recovery_comparison.csv
```

## Evaluation signals

- mass_recovery_comparison.csv exists and contains fractional error columns for both GravSphere2 and the simple estimator for every mock, including few-LOS-velocity runs
- GravSphere2 achieves smaller |(M_inferred - M_true)/M_true| than the simple mass estimator across the mocks, including few-velocity cases
- GravSphere2 errors show no induced bias as tracer numbers drop — only larger uncertainties (graceful degradation, even down to ten tracers)
- Posterior masses within R1/2 are quoted as 95% credible intervals retrieved from checkpoint files in Sampler Chains subfolders

## Limitations

- GravSphere2 degrades gracefully with fewer tracers (down to ten), but uncertainties grow large — with 100 tracers and no PMs, logarithmic density slopes are recovered with ~30% (20%) errors
- Assumes spherical symmetry and equilibrium stellar tracer systems; conclusions were established on mocks (Gaia Challenge, Fornax-like simulation)
- Comparison is on mocks with known true masses — applying the benchmark to real galaxies without truth is not straightforward

## Evidence

- [abstract] GravSphere2 outperforms simple mass estimators: "We find that G RAV S PHERE 2 outperforms simple mass estimators"
- [abstract] Graceful degradation without induced bias: "As the number of tracers is lowered (even down to just ten tracers) it gracefully degrades, with larger uncertainties but no induced bias"
- [methods] Checkpoint storage and retrieval: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [methods] Running scripts for fits: "We have set some scripts to show an example of running the code (run_example_notebook.ipynb and run_multcpu_example.py for parallel programming"
- [readme] Bin-free individual-velocity fitting: "It combines individual stellar velocities from LOS and proper motion (PM) measurements to solve the Jeans equations up to fourth order, without any data binning."
