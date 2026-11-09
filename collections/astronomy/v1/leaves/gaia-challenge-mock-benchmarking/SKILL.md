---
name: gaia-challenge-mock-benchmarking
description: Use when validating or calibrating a stellar-dynamics mass-modeling method
  on known ground truth, especially to test whether cusped versus cored dark-matter
  density profiles in dwarf galaxies can be statistically distinguished from discrete
  tracer kinematics.
when_to_use_negative:
- The system is not a spherical stellar tracer population in equilibrium (e.g. tidally
  disturbed or rotating dwarfs) — GravSphere2 assumes spherical higher-order Jeans
  equilibrium
- The mock or real dataset has far fewer than ~100 tracers and no proper motions,
  where slope errors grow to ~30% and the cuspy/cored distinction is not the goal
- You need binned kinematic profiles as output rather than a bin-free posterior over
  the density slope — the older justinread/gravsphere + binulator workflow fits that
  case instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: GravSphere2
  role: bin-free higher-order Jeans mass-modeling code run on each Gaia Challenge
    mock; configured via initialise_ files and gravsphere2.py
  repo: https://github.com/dadams42/GravSphere2
- name: dynesty
  role: nested sampler used to sample the GravSphere2 likelihood for each mock fit
- name: corner
  role: posterior corner-plot visualization for the recovered slope and other parameters
- name: matplotlib
  role: generates density, slope, anisotropy, dispersion, and kurtosis profile plots
    from checkpoints via plots_example_notebook.ipynb
- name: numpy
  role: array handling for tracer positions and velocities in initialise_ files
- name: scipy
  role: numerical routines underpinning the Jeans-equation solver
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202557195
    title: 'GravSphere2: A higher order Jeans method for mass modeling spherical stellar
      systems'
  source_content_hash: sha256:753298117d1579556f33194d90bcf479178f3a934b51d1670a7271a127094e01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dadams42/GravSphere2
  tools:
  - GravSphere2
  - Python
  - corner
  - dynesty
  - matplotlib
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gaia-challenge-mock-benchmarking

## Summary

Benchmark a Jeans mass-modeling code against the Gaia Challenge mock dwarf-galaxy datasets by running cuspy and cored mocks with 1000 tracer stars through GravSphere2, comparing the recovered logarithmic density slope at R1/2 against the published 12%/25% (LOS-only) and 8%/12% (with proper motions) statistical errors.

## When to use

Use when validating or calibrating a stellar-dynamics mass-modeling method on known ground truth, especially to test whether cusped versus cored dark-matter density profiles in dwarf galaxies can be statistically distinguished from discrete tracer kinematics.

## When NOT to use

- The system is not a spherical stellar tracer population in equilibrium (e.g. tidally disturbed or rotating dwarfs) — GravSphere2 assumes spherical higher-order Jeans equilibrium
- The mock or real dataset has far fewer than ~100 tracers and no proper motions, where slope errors grow to ~30% and the cuspy/cored distinction is not the goal
- You need binned kinematic profiles as output rather than a bin-free posterior over the density slope — the older justinread/gravsphere + binulator workflow fits that case instead

## Inputs

- Gaia Challenge mock dataset, cuspy halo, 1000 tracer stars (positions, LOS velocities, proper motions) from the Surrey astrowiki
- Gaia Challenge mock dataset, cored halo, 1000 tracer stars (positions, LOS velocities, proper motions) from the Surrey astrowiki
- initialise_ configuration files specifying tracer data, likelihood, and priors in the GravSphere2 format

## Outputs

- dynesty sampler checkpoint files per mock (stored in Sampler Chains subfolders, resumable)
- recovered density, slope, anisotropy, dispersion, and kurtosis profile plots per mock
- four-row table (LOS cuspy, LOS cored, PM cuspy, PM cored) of logarithmic density slope at R1/2 with statistical errors, e.g. slope_error_table.csv

## How to apply

Download the cuspy and cored Gaia Challenge mock datasets (positions, LOS velocities, and proper motions for 1000 tracers) from the Surrey astrowiki, then create an initialise_ file for each mock in the GravSphere2 format specifying tracer positions, velocities, tracer profile data, and priors. Import the initialise_ file at the top of gravsphere2.py to set the likelihood and priors, which auto-creates an Output directory with Plots and Sampler Chains subfolders. Run the LOS-velocity-only fits for both mocks with the dynesty nested sampler via run_multcpu_example.py or run_example_notebook.ipynb, storing resumable checkpoint files in the Sampler Chains subfolders, then re-run both mocks a second time including proper motion velocities in the likelihood. Use plots_example_notebook.ipynb to generate the recovered density, slope, anisotropy, dispersion, and kurtosis profiles from each checkpoint, and extract the logarithmic density slope at R1/2 and its statistical error from each posterior. Success is judged by whether the slope errors reproduce the published 12% (cuspy) / 25% (cored) for LOS-only fits and 8% (cuspy) / 12% (cored) with PMs, with slopes recovered within the quoted 95% confidence intervals.

## Related tools

- **GravSphere2** (bin-free higher-order Jeans mass-modeling code run on each Gaia Challenge mock; configured via initialise_ files and gravsphere2.py) — https://github.com/dadams42/GravSphere2
- **dynesty** (nested sampler used to sample the GravSphere2 likelihood for each mock fit)
- **corner** (posterior corner-plot visualization for the recovered slope and other parameters)
- **matplotlib** (generates density, slope, anisotropy, dispersion, and kurtosis profile plots from checkpoints via plots_example_notebook.ipynb)
- **numpy** (array handling for tracer positions and velocities in initialise_ files)
- **scipy** (numerical routines underpinning the Jeans-equation solver)

## Examples

```
python run_multcpu_example.py  # after importing initialise_cuspy_1000 in gravsphere2.py; then open plots_example_notebook.ipynb to extract slope(R1/2) from the Sampler Chains checkpoint
```

## Evaluation signals

- LOS-only fits reproduce slope errors at R1/2 of 12% (cuspy) and 25% (cored) for 1000 tracers
- Fits including proper motions improve the slope errors to 8% (cuspy) and 12% (cored)
- Recovered density, anisotropy, and logarithmic density slope profiles fall within the quoted 95% confidence intervals over r/R1/2 of about 0.1 to 10
- A second run starting from the stored Sampler Chains checkpoint file reproduces the posterior (checkpoint resumability check)
- The cuspy and cored mocks yield statistically distinguishable slope posteriors despite different error magnitudes

## Limitations

- Statistical errors are larger for cored mocks (25% LOS-only vs 12% cuspy), so distinguishing cusp from core with LOS data alone is marginal; PMs are effectively required for a robust distinction
- With only 100 tracers and no PMs, slope errors degrade to ~30% (20%), so the 1000-tracer benchmark does not transfer to sparse datasets
- Degradation with fewer tracers (down to ten) increases uncertainties but introduces no bias, so results must be reported as uncertainty growth, not bias
- The Gaia Challenge mock data carry their own citation requirement and must be acknowledged separately from the code

## Evidence

- [abstract] Using 1000 tracers without PMs, G RAV S PHERE 2 recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data: "Using 1000 tracers without PMs, G RAV S PHERE 2 recovers the logarithmic density slope at R 1/2 with 12%(25%) statistical errors for cuspy (cored) mock data"
- [methods] The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified: "The galaxy / object is selected by importing the initialisation file at the beginning of the gravsphere2.py file, where the likelihood and other arguments for the fit (e.g. priors) are specified"
- [methods] This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders): "This will create and store a checkpoint file from which all data can be retrieved and be run subsequently if needed (these are saved by default in the Sampler Chains subfolders)"
- [abstract] was able to recover the mass density, stellar velocity anisotropy, and the logarithmic slope of the mass density profile within its quoted 95% confidence intervals: "was able to recover the mass density, stellar velocity anisotropy, and the logarithmic slope of the mass density profile within its quoted 95% confidence intervals"
- [readme] For the purposes of testing, we recommend you try out one of the Gaia Challenge datasets http://astrowiki.ph.surrey.ac.uk/dokuwiki/doku.php?id=tests:sphtri:spherical: [evidence span withheld — anchor status: not_found]
