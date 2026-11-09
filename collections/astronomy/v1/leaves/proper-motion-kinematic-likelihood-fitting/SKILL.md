---
name: proper-motion-kinematic-likelihood-fitting
description: Use when a spherical stellar system mock or dataset has discrete proper
  motion velocities (in addition to LOS velocities) and the goal is to tighten statistical
  errors on the recovered density slope at R1/2 via higher-order Jeans fitting.
when_to_use_negative:
- The dataset contains only line-of-sight velocities with no proper motion measurements
  — the PM likelihood terms cannot be populated and the LOS-only fit is the correct
  baseline.
- The stellar system is not in dynamical equilibrium — GravSphere2 is designed to
  mass-model systems of stellar tracers in equilibrium and will be biased otherwise.
- Fewer than ~100 tracers with modest kinematics — errors grow (e.g. ~30%/20% slope
  errors with 100 tracers and no PMs), so the PM-error improvement claim may not generalize
  to very sparse samples.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: GravSphere2
  role: Runs the bin-free higher-order Jeans fit combining individual LOS and PM velocities;
    configured via initialise_ file and gravsphere2.py
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: Nested sampler used for the posterior sampling of the PM-inclusive likelihood;
    verified with dynesty 2.1.5
  repo: https://github.com/dadams42/GravSphere2
- name: corner
  role: Posterior corner plots for inspecting the slope parameter with PMs included;
    verified with corner 2.2.2
  repo: https://github.com/dadams42/GravSphere2
- name: Python 3.10.12 / numpy 1.26.4 / scipy 1.15.1 / matplotlib 3.9.0
  role: Runtime stack for executing run_multcpu_example.py (parallel) and the plotting
    notebook
  repo: https://github.com/dadams42/GravSphere2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:0b271bd56926661646eda87bfc89e3a8de6a1991b3e48d91ffbb1d3f66375c85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dadams42/GravSphere2
  tools:
  - GravSphere2
  - Python
  - Python 3.10.12 / numpy 1.26.4 / scipy 1.15.1 / matplotlib 3.9.0
  - corner
  - dynesty
  - matplotlib
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# proper-motion-kinematic-likelihood-fitting

## Summary

Re-run GravSphere2 mass-modeling fits with proper motion (PM) velocities included in the likelihood alongside line-of-sight (LOS) velocities, using the fourth-order Jeans bin-free approach to quantify how 2D kinematic information tightens recovery of the logarithmic density slope at R1/2.

## When to use

Use when a spherical stellar system mock or dataset has discrete proper motion velocities (in addition to LOS velocities) and the goal is to tighten statistical errors on the recovered density slope at R1/2 via higher-order Jeans fitting.

## When NOT to use

- The dataset contains only line-of-sight velocities with no proper motion measurements — the PM likelihood terms cannot be populated and the LOS-only fit is the correct baseline.
- The stellar system is not in dynamical equilibrium — GravSphere2 is designed to mass-model systems of stellar tracers in equilibrium and will be biased otherwise.
- Fewer than ~100 tracers with modest kinematics — errors grow (e.g. ~30%/20% slope errors with 100 tracers and no PMs), so the PM-error improvement claim may not generalize to very sparse samples.

## Inputs

- Gaia Challenge mock dataset with 1000 tracer stars (positions, LOS velocities, and PMs) from the Surrey astrowiki (cuspy and cored variants)
- initialise_ file in GravSphere2 format (tracer positions, velocities, tracer profile data, priors)
- LOS-only sampler checkpoint from the prior fit (resumable checkpoint in the Sampler Chains subfolder)

## Outputs

- PM-inclusive sampler checkpoint in the Output/Sampler Chains subfolder
- Recovered density, slope, anisotropy, dispersion, and kurtosis profiles from plots_example_notebook.ipynb
- Posterior for the logarithmic density slope at R1/2 with statistical error (e.g. 8% cuspy / 12% cored with 1000 tracers)
- Four-row comparison table (LOS cuspy, LOS cored, PM cuspy, PM cored), e.g. slope_error_table.csv

## How to apply

Start from a validated LOS-only GravSphere2 fit (dynesty nested sampler, initialisation via an initialise_ file imported in gravsphere2.py that sets the likelihood and priors), then re-run the same mock a second time with proper motion velocities included in the likelihood. GravSphere2 combines individual LOS and PM velocities to solve the Jeans equations up to fourth order without binning, using flexible, self-consistent PDFs that include kurtosis. Expect the slope-at-R1/2 statistical errors to shrink from 12% (cuspy) / 25% (cored) with 1000 LOS-only tracers to 8% (cuspy) / 12% (cored) when PMs are added, and confirm the recovered slope lies within the quoted 95% confidence intervals. Generate density, slope, anisotropy, dispersion, and kurtosis profiles from the sampler checkpoint via plots_example_notebook.ipynb and compare the LOS-only versus PM posterior tables.

## Related tools

- **GravSphere2** (Runs the bin-free higher-order Jeans fit combining individual LOS and PM velocities; configured via initialise_ file and gravsphere2.py) — https://github.com/dadams42/GravSphere2
- **dynesty** (Nested sampler used for the posterior sampling of the PM-inclusive likelihood; verified with dynesty 2.1.5) — https://github.com/dadams42/GravSphere2
- **corner** (Posterior corner plots for inspecting the slope parameter with PMs included; verified with corner 2.2.2) — https://github.com/dadams42/GravSphere2
- **Python 3.10.12 / numpy 1.26.4 / scipy 1.15.1 / matplotlib 3.9.0** (Runtime stack for executing run_multcpu_example.py (parallel) and the plotting notebook) — https://github.com/dadams42/GravSphere2

## Examples

```
python run_multcpu_example.py  # after importing the PM-inclusive initialise_ file in gravsphere2.py; then run plots_example_notebook.ipynb
```

## Evaluation signals

- The PM-inclusive slope errors are smaller than the LOS-only baseline: 8% (cuspy) and 12% (cored) versus 12% and 25% with 1000 tracers.
- The recovered logarithmic density slope at R1/2 falls within the quoted 95% confidence intervals of the mock truth.
- A resumable checkpoint file is written in the Sampler Chains subfolder and can be reloaded for plotting.
- Plots from plots_example_notebook.ipynb render density, slope, anisotropy, dispersion, and kurtosis profiles without errors.
- The four-row comparison table (LOS/PM × cuspy/cored) reproduces the published error pattern, with PM rows strictly tighter than LOS rows.

## Limitations

- GravSphere2 assumes spherical symmetry and equilibrium stellar tracer systems; mocks with triaxiality or out-of-equilibrium dynamics are outside scope.
- As tracers drop below 1000, uncertainties grow (e.g. ~30%/20% slope errors with 100 tracers, no PMs), though the method degrades gracefully down to ten tracers without induced bias.
- The likelihood sampling depends on specific verified dependency versions (python 3.10.12, scipy 1.15.1, numpy 1.26.4, matplotlib 3.9.0, dynesty 2.1.5, corner 2.2.2).

## Evidence

- [abstract] PMs improve slope errors: "Using 1000 tracers without PMs, G RAV S PHERE 2 recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data, enabling us to make a distinction between"
- [abstract] PM combination in fourth-order Jeans: "It combines individual stellar velocities from LOS and proper motion (PM) measurements to solve the Jeans equations up to fourth order, without any data binning."
- [methods] initialise_ file configuration: "The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified"
- [methods] checkpoint storage: "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [methods] plotting notebook: "We have also created a notebook (plots_example_notebook.ipynb) where we show examples of plotting density, slope, anisotropy, dispersion, and kurtosis profiles"
