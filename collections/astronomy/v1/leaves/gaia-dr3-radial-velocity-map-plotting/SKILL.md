---
name: gaia-dr3-radial-velocity-map-plotting
description: Use when you need to generate or reproduce the local velocity-space distribution
  and disk-wide median radial velocity maps from a backward-integrated Milky Way model
  and compare them against Gaia DR3 stellar kinematics.
when_to_use_negative:
- Only a minimal version of the SPIBACK code is available for quick computations,
  so workflows needing the fuller functionality of Khalil et al.'s complete model
  cannot yet be run from the notebook
- If your goal is Gaia DR3 data analysis itself rather than model-vs-data kinematic
  map reproduction, this notebook-based skill only generates model plots, not observational
  reductions
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SPIBACK
  role: Jupyter notebook that generates Milky Way models via backward integration
    and plots the 2D local velocity distribution and median radial velocity maps
  repo: github.com/yrkhalil/SPIBACK
- name: AGAMA
  role: Galaxy modelling library (installed via pip install agama) providing the gravitational
    potential and orbit machinery used by the backward integration
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: PyTorch
  role: Required dependency for the notebook's differentiable backward integration
    computation
  repo: https://pytorch.org
- name: torchdiffeq
  role: Required ODE solver package (pip install torchdiffeq) for the backward orbit
    integration
- name: Matplotlib
  role: Plotting backend that renders the velocity-space distribution and radial velocity
    map figures
- name: Jupyter Notebook
  role: Interface in which all cells of spiback_minimalversion_run.ipynb are run to
    display the plots properly
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:1ea20bc8976f62d5715ccbb2738b5642173539df052e9791df27d1eab08a1ec4
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# gaia-dr3-radial-velocity-map-plotting

## Summary

Reproduce the SPIBACK notebook's kinematic plots — the 2D local velocity-space distribution at the Sun's position and the median Galactocentric radial velocity map across the Galactic disk area probed with Gaia DR3 — using the fiducial non-axisymmetric Milky Way model (bar pattern speed 37 km/s/kpc plus two-armed and three-armed spiral modes).

## When to use

Use when you need to generate or reproduce the local velocity-space distribution and disk-wide median radial velocity maps from a backward-integrated Milky Way model and compare them against Gaia DR3 stellar kinematics.

## When NOT to use

- Only a minimal version of the SPIBACK code is available for quick computations, so workflows needing the fuller functionality of Khalil et al.'s complete model cannot yet be run from the notebook
- If your goal is Gaia DR3 data analysis itself rather than model-vs-data kinematic map reproduction, this notebook-based skill only generates model plots, not observational reductions

## Inputs

- SPIBACK notebook files (spiback_minimalversion_run.ipynb) from github.com/yrkhalil/SPIBACK
- Fiducial model parameters (bar pattern speed 37 km/s/kpc, two spiral modes)
- bins_model parameter (default 32+1; 16+1 for quick tests, 64+1 for data-equivalent resolution)

## Outputs

- 2D local velocity-space distribution plot at the Sun's position (PNG/PDF)
- Median Galactocentric radial velocity map across the Galactic disk area probed with Gaia DR3 (PNG/PDF)

## How to apply

Install PyTorch, AGAMA (pip install agama), torchdiffeq (pip install torchdiffeq), SciPy, and Matplotlib with Python 3.10 or above. Download the SPIBACK files from github.com/yrkhalil/SPIBACK into a single folder and run all cells of spiback_minimalversion_run.ipynb on Jupyter lab/notebook. Keep the parameters at their default 'fiducial model' values (bar pattern speed 37 km/s/kpc; two-armed spiral mode at 13.1 km/s/kpc with ~25% local contrast; three-armed mode at 16.4 km/s/kpc with ~9% contrast); the key parameter bins_model defaults to 32+1 (~2 hours runtime), use 16+1 only for quick tests and 64+1 for resolution equivalent to the plotted data. Export the displayed plots as PNG/PDF and compare them against the corresponding figures of arXiv:2411.12800 to confirm the model reproduces the reported moving groups, phase-space ridges, and radial velocity features. The rationale is that the backward-integration method adjusts the bar to peaks of the local stellar velocity distribution and the spiral arms to the median radial velocity as a function of position within the disk.

## Related tools

- **SPIBACK** (Jupyter notebook that generates Milky Way models via backward integration and plots the 2D local velocity distribution and median radial velocity maps) — github.com/yrkhalil/SPIBACK
- **AGAMA** (Galaxy modelling library (installed via pip install agama) providing the gravitational potential and orbit machinery used by the backward integration) — https://github.com/GalacticDynamics-Oxford/Agama
- **PyTorch** (Required dependency for the notebook's differentiable backward integration computation) — https://pytorch.org
- **torchdiffeq** (Required ODE solver package (pip install torchdiffeq) for the backward orbit integration)
- **Matplotlib** (Plotting backend that renders the velocity-space distribution and radial velocity map figures)
- **Jupyter Notebook** (Interface in which all cells of spiback_minimalversion_run.ipynb are run to display the plots properly)

## Examples

```
pip install agama torchdiffeq; jupyter lab  # then run all cells of spiback_minimalversion_run.ipynb (bins_model=32+1, ~2 h)
```

## Evaluation signals

- Exported 2D velocity-space plot shows the moving-group peaks and phase-space ridges matching the corresponding figures of the source paper (arXiv:2411.12800)
- Median Galactocentric radial velocity map qualitatively matches the Gaia DR3-probed disk area figures of the source paper
- Plots display properly only after all cells of spiback_minimalversion_run.ipynb have been run — partial execution yields missing/incomplete figures
- bins_model=64+1 output should be visually equivalent in resolution to the plotted Gaia DR3 data; 16+1 output is coarser and only suitable for quick tests
- Fiducial model reproduces the bar pattern speed 37 km/s/kpc and both spiral modes (13.1 and 16.4 km/s/kpc pattern speeds) as reported in the abstract

## Limitations

- bins_model=32+1 (default) takes about 2 hours to complete; 64+1 significantly increases computation time
- Currently only a minimal version of the code is available; a fuller version will be provided soon
- The solution only qualitatively (a posteriori) reproduces moving groups and phase-space ridges — it is a model-adjustment check, not a quantitative fit validation

## Evidence

- [intro] The SPIBACK notebook's default fiducial model parameters: "The parameters are set by default to be those of the "fiducial model", see Khalil et al for all the details"
- [readme] Notebook execution requirement: "Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly."
- [readme] bins_model runtime/resolution tradeoff: "This is set by default to 32+1 taking about 2 hours to complete. A value of 16+1 can be used for quick tests."
- [abstract] Outputs of the notebook: "allows users to plot the 2D local velocity space distribution at the Sun's position"
- [abstract] Validation against source paper figures: "We checked a posteriori that our solution also qualitatively reproduces various other features of the global non-axisymmetric phase-space distribution"
- [methods] Required dependencies: "This Notebook crucially needs the installation of:

- PyTorch: https://pytorch.org"
