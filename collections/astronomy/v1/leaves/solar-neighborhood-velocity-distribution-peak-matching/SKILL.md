---
name: solar-neighborhood-velocity-distribution-peak-matching
description: Use when fitting a non-axisymmetric Galactic potential (bar + spiral
  arms) to Gaia DR3 disk-star kinematics, e.g. matching local velocity-space moving
  groups and phase-space ridges.
when_to_use_negative:
- When only a quick test run is needed and computation time is limited — bins_model
  32+1 takes about 2 hours; use 16+1 instead
- When axisymmetric (non-bar, non-spiral) dynamics suffice — this skill specifically
  targets non-axisymmetric bar and spiral-arm components
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SPIBACK
  role: Jupyter Notebook that generates Milky Way models through the backward integration
    method and plots the velocity distributions used for fitting
  repo: github.com/yrkhalil/SPIBACK
- name: PyTorch
  role: Required dependency providing tensor/autodiff backbone for the notebook's
    orbit computations
  repo: https://pytorch.org
- name: AGAMA
  role: Galactic dynamics library used for potential/force computation and orbit integration,
    installed via pip install agama
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: torchdiffeq
  role: ODE solver package used for backward integration of stellar orbits, installed
    via pip install torchdiffeq
- name: SciPy
  role: Scientific computing dependency needed to run the notebook
- name: Matplotlib
  role: Plotting library used to display the 2D velocity distribution and radial velocity
    maps
- name: Jupyter Notebook
  role: Interface in which spiback_minimalversion_run.ipynb is run as a single folder
    of files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:27ea0699ea6fad7ce18495440b1a268442586d40e40df337690efa7ab4590957
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AGAMA
  - Jupyter Notebook
  - Matplotlib
  - PyTorch
  - Python
  - SPIBACK
  - SciPy
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Solar-neighborhood velocity distribution peak matching via backward integration

## Summary

A two-stage backward-integration fitting loop (SPIBACK) that adjusts a Milky Way bar model to selected peaks of the solar-neighborhood stellar velocity distribution, then tunes spiral-arm amplitude, phase, pitch angle, and pattern speed to the median Galactocentric radial velocity map from Gaia DR3.

## When to use

Use when fitting a non-axisymmetric Galactic potential (bar + spiral arms) to Gaia DR3 disk-star kinematics, e.g. matching local velocity-space moving groups and phase-space ridges.

## When NOT to use

- When only a quick test run is needed and computation time is limited — bins_model 32+1 takes about 2 hours; use 16+1 instead
- When axisymmetric (non-bar, non-spiral) dynamics suffice — this skill specifically targets non-axisymmetric bar and spiral-arm components

## Inputs

- Gaia DR3 in-plane motions of disk stars (median Galactocentric radial velocity maps across the Galactic disk)
- Selected peaks of the solar-neighborhood stellar velocity distribution (moving groups)
- Model parameter settings in spiback_minimalversion_run.ipynb (bar pattern speed, spiral amplitude, phase, pitch angle, pattern speed, bins_model)

## Outputs

- Fitted non-axisymmetric Milky Way disk potential (bar plus two spiral modes)
- 2D local velocity-space distribution plot at the Sun's position
- Median Galactocentric radial velocity map across the area of the Galactic disk probed with Gaia DR3

## How to apply

Install PyTorch, AGAMA (pip install agama), torchdiffeq, SciPy, and Matplotlib with Python 3.10+, then run spiback_minimalversion_run.ipynb in Jupyter. Stage 1: perform backward integration of stellar orbits and adjust the bar model to selected peaks of the stellar velocity distribution in the solar neighborhood. Stage 2: adjust spiral-arm amplitude, phase, pitch angle, and pattern speed to the median Galactocentric radial velocity as a function of disk position. Keep bins_model at the default 32+1 (~2 hours of computation); use 16+1 for quick tests or 64+1 for resolution equivalent to the plotted data at higher cost. The fiducial parameters are a bar pattern speed of 37 km/s/kpc with two-armed and three-armed spiral modes at pattern speeds 13.1 and 16.4 km/s/kpc.

## Related tools

- **SPIBACK** (Jupyter Notebook that generates Milky Way models through the backward integration method and plots the velocity distributions used for fitting) — github.com/yrkhalil/SPIBACK
- **PyTorch** (Required dependency providing tensor/autodiff backbone for the notebook's orbit computations) — https://pytorch.org
- **AGAMA** (Galactic dynamics library used for potential/force computation and orbit integration, installed via pip install agama) — https://github.com/GalacticDynamics-Oxford/Agama
- **torchdiffeq** (ODE solver package used for backward integration of stellar orbits, installed via pip install torchdiffeq)
- **SciPy** (Scientific computing dependency needed to run the notebook)
- **Matplotlib** (Plotting library used to display the 2D velocity distribution and radial velocity maps)
- **Jupyter Notebook** (Interface in which spiback_minimalversion_run.ipynb is run as a single folder of files)

## Examples

```
pip install torchdiffeq; pip install agama  # then run all cells of spiback_minimalversion_run.ipynb in Jupyter (Python 3.10+)
```

## Evaluation signals

- A posteriori check that the fitted solution qualitatively reproduces the Gaia DR3 moving groups and phase-space ridges reported in Khalil et al.
- The 2D local velocity-space distribution at the Sun's position displays the expected moving-group peaks after running all notebook cells
- The median Galactocentric radial velocity map qualitatively matches the map observed across the Galactic disk probed with Gaia DR3
- Two-armed spiral mode recovers ~25% local contrast surface density and pattern speed ~13.1 km/s/kpc; three-armed mode ~9% contrast and ~16.4 km/s/kpc, matching Crux-Scutum/Local/Outer and Carina-Sagittarius/Perseus arm segments respectively

## Limitations

- Currently only a minimal version of the code is available for quick computations; a fuller version is planned (contact yassin.khalil@unistra.fr)
- bins_model 32+1 takes about 2 hours; 64+1 significantly increases computation time
- The a posteriori validation is qualitative (moving groups and phase-space ridges), not a quantitative goodness-of-fit metric
- Requires Python 3.10 or above

## Evidence

- [abstract] backward-integration fitting loop stage 1: "with the backward integration method, first adjusting the bar model to selected peaks of the stellar velocity distribution in the solar neighborhood"
- [abstract] backward-integration fitting loop stage 2: "adjusting the amplitude, phase, pitch angle, and pattern speed of spiral arms to the median radial velocity as a function of position within the disk"
- [abstract] fiducial model parameters: "This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively."
- [methods] installation and runtime requirements: "This is a Python3 notebook, which needs Python 3.10 or above."
- [readme] bins_model tradeoff: "This is set by default to 32+1 taking about 2 hours to complete. A value of 16+1 can be used for quick tests."
- [readme] run instructions: "Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly."
