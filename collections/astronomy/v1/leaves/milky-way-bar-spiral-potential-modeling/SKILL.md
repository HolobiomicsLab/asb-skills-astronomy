---
name: milky-way-bar-spiral-potential-modeling
description: Use when modeling Milky Way bar/spiral non-axisymmetric potentials to
  reproduce Gaia DR3 moving groups or median Galactocentric radial velocity maps,
  or when comparing a candidate bar/spiral parameter set against Khalil et al. 2025
  fiducial kinematic features.
when_to_use_negative:
- The goal is an axisymmetric Milky Way potential or modeling not driven by bar/spiral
  perturbations — SPIBACK's whole purpose is the non-axisymmetric bar + two-armed/three-armed
  spiral configuration
- Only a quick exploratory test is needed and compute time is limited — the default
  bins_model=32+1 takes ~2 hours, so it is the wrong setting (use 16+1 instead)
- The analysis needs features beyond the minimal released code — 'Currently only a
  minimal version of the code is available', so workflows requiring the fuller version
  cannot be supported yet
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SPIBACK
  role: Jupyter notebook that generates Milky Way models through the backward integration
    method and plots the 2D local velocity distribution and median radial velocity
    maps
  repo: github.com/yrkhalil/SPIBACK
- name: AGAMA
  role: Galaxy-modelling library providing gravitational potential and force computation
    and orbit integration for the Milky Way model
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: PyTorch
  role: Required dependency for the notebook's differentiable backward-integration
    computation
  repo: https://pytorch.org
- name: torchdiffeq
  role: ODE solver library installed via pip to perform the backward orbit integration
- name: SciPy
  role: Required scientific Python dependency for the notebook
- name: Matplotlib
  role: Renders the 2D velocity-space distribution and median radial velocity map
    plots
- name: Python 3.10
  role: Minimum Python version required to run spiback_minimalversion_run.ipynb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:39fae180346056044d54c58130fc13880b90fe345ff325fc576d4b6cce80eb3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AGAMA
  - Matplotlib
  - PyTorch
  - Python
  - Python 3.10
  - SPIBACK
  - SciPy
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# milky-way-bar-spiral-potential-modeling

## Summary

Generates a non-axisymmetric Milky Way disk potential via the SPIBACK Jupyter notebook's backward-integration method, with a fiducial configuration (bar at 37 km s−1 kpc−1 plus two-armed and three-armed spiral modes) tuned to Gaia DR3 disk-star kinematics.

## When to use

Use when modeling Milky Way bar/spiral non-axisymmetric potentials to reproduce Gaia DR3 moving groups or median Galactocentric radial velocity maps, or when comparing a candidate bar/spiral parameter set against Khalil et al. 2025 fiducial kinematic features.

## When NOT to use

- The goal is an axisymmetric Milky Way potential or modeling not driven by bar/spiral perturbations — SPIBACK's whole purpose is the non-axisymmetric bar + two-armed/three-armed spiral configuration
- Only a quick exploratory test is needed and compute time is limited — the default bins_model=32+1 takes ~2 hours, so it is the wrong setting (use 16+1 instead)
- The analysis needs features beyond the minimal released code — 'Currently only a minimal version of the code is available', so workflows requiring the fuller version cannot be supported yet

## Inputs

- SPIBACK notebook files (spiback_minimalversion_run.ipynb) from github.com/yrkhalil/SPIBACK
- Model parameters (bar pattern speed, spiral amplitude, phase, pitch angle, pattern speed, bins_model) set in the notebook
- Gaia DR3 in-plane disk-star kinematics used as the comparison/reference data for the velocity distribution and radial velocity maps

## Outputs

- 2D local velocity-space distribution at the Sun's position (PNG/PDF plot)
- Median Galactocentric radial velocity map across the Galactic disk area probed with Gaia DR3 (PNG/PDF plot)
- Backward-integrated Milky Way model (bar + two spiral modes) underlying the plots

## How to apply

Install PyTorch, AGAMA (pip install agama), torchdiffeq, SciPy, and Matplotlib under Python 3.10+, download the SPIBACK files into a single folder, and run all cells of spiback_minimalversion_run.ipynb in Jupyter. Keep the default 'fiducial model' parameters: bar pattern speed 37 km s−1 kpc−1, a two-armed spiral mode with ~25% local contrast surface density at pattern speed 13.1 km s−1 kpc−1 (matching Crux-Scutum, Local, and Outer arm segments), and a three-armed mode with ~9% local contrast at 16.4 km s−1 kpc−1 (matching Carina-Sagittarius and Perseus). The key compute parameter bins_model defaults to 32+1 (~2 hours); use 16+1 for quick tests and 64+1 for data-equivalent resolution at much higher cost. The notebook backward-integrates orbits to adjust the bar to peaks of the local stellar velocity distribution, then tunes spiral amplitude, phase, pitch angle, and pattern speed to the median radial velocity across the disk.

## Related tools

- **SPIBACK** (Jupyter notebook that generates Milky Way models through the backward integration method and plots the 2D local velocity distribution and median radial velocity maps) — github.com/yrkhalil/SPIBACK
- **AGAMA** (Galaxy-modelling library providing gravitational potential and force computation and orbit integration for the Milky Way model) — https://github.com/GalacticDynamics-Oxford/Agama
- **PyTorch** (Required dependency for the notebook's differentiable backward-integration computation) — https://pytorch.org
- **torchdiffeq** (ODE solver library installed via pip to perform the backward orbit integration)
- **SciPy** (Required scientific Python dependency for the notebook)
- **Matplotlib** (Renders the 2D velocity-space distribution and median radial velocity map plots)
- **Python 3.10** (Minimum Python version required to run spiback_minimalversion_run.ipynb)

## Examples

```
pip install agama; pip install torchdiffeq; jupyter notebook spiback_minimalversion_run.ipynb
```

## Evaluation signals

- Exported 2D local velocity-space distribution at the Sun's position reproduces the selected peaks (moving groups) of the Gaia DR3 solar-neighborhood stellar velocity distribution
- Median Galactocentric radial velocity map qualitatively matches the corresponding figures of the source paper (arXiv:2411.12800) over the disk area probed with Gaia DR3
- All cells of spiback_minimalversion_run.ipynb execute without error under Python 3.10+ with PyTorch, AGAMA, torchdiffeq, SciPy, and Matplotlib installed
- The model configuration matches the fiducial values: bar pattern speed 37 km s−1 kpc−1, two-armed mode at 13.1 km s−1 kpc−1 with ~25% contrast, three-armed mode at 16.4 km s−1 kpc−1 with ~9% contrast
- A posteriori checks confirm the solution qualitatively reproduces additional phase-space features such as moving groups and phase-space ridges

## Limitations

- Currently only a minimal version of the code is available for quick computations; a fuller version will be provided soon
- bins_model=32+1 takes about 2 hours; 64+1 gives resolution equivalent to the plotted data but significantly increases computation time
- Reproducing the fiducial model assumes Gaia DR3 in-plane disk kinematics; the spiral modes were adjusted specifically to median radial velocity and local velocity-distribution peaks in that data

## Evidence

- [abstract] fiducial bar and spiral parameters: "This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively."
- [abstract] spiral mode parameters: "The two-armed spiral mode has a ~25% local contrast surface density and a low pattern speed of 13.1 km s−1 kpc−1"
- [methods] notebook run procedure: "Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly."
- [readme] default fiducial parameters in notebook: "The parameters are set by default to be those of the "fiducial model", see Khalil et al for all the details: https://arxiv.org/abs/2411.12800"
- [readme] bins_model compute settings: "This is set by default to 32+1 taking about 2 hours to complete. A value of 16+1 can be used for quick tests."
- [readme] minimal-version caveat: "Currently only a minimal version of the code is available for quick computations. A fuller version will be provided soon."
