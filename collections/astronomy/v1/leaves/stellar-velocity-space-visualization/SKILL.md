---
name: stellar-velocity-space-visualization
description: Use when reproducing or inspecting Milky Way disk kinematic substructure
  — local velocity-space moving groups or disk-wide median radial velocity maps —
  from a non-axisymmetric bar+spiral potential, or comparing such plots to Gaia DR3
  observations.
when_to_use_negative:
- When the goal is fitting bar/spiral parameters to new data rather than visualizing
  a pre-configured model — use the parameter-adjustment workflow instead of the default
  fiducial run
- When only a quick check is needed and multi-hour runtime is unacceptable without
  lowering bins_model to 16+1
- When working outside the Milky Way disk context or with axisymmetric potentials
  — the notebook is built for the backward-integration non-axisymmetric bar+two-spiral-modes
  model
edam_operation: http://edamontology.org/operation_0337
tools:
- name: SPIBACK
  role: Jupyter Notebook that generates the Milky Way backward-integration model and
    displays the velocity-space plots
  repo: https://github.com/yrkhalil/SPIBACK
- name: PyTorch
  role: Required dependency for running the backward-integration computation
  repo: https://pytorch.org
- name: AGAMA
  role: Galactic potential/orbit-integration library required by the notebook; installed
    via pip install agama
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: torchdiffeq
  role: ODE-integration dependency for the backward integration, installed via pip
    install torchdiffeq
- name: SciPy
  role: Supporting scientific computing dependency
- name: Matplotlib
  role: Renders the 2D velocity distribution and radial velocity map plots
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:3fc5850e31ba94b121b28bfc78157e0019eb431f18695c1b13204234eb8f5274
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yrkhalil/SPIBACK
  tools:
  - AGAMA
  - Matplotlib
  - PyTorch
  - Python
  - SPIBACK
  - SciPy
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# stellar-velocity-space-visualization

## Summary

Generates the 2D local velocity-space distribution at the Sun's position and median Galactocentric radial velocity maps across the Galactic disk using the SPIBACK backward-integration Milky Way model, enabling visual comparison of bar/spiral-induced kinematic features against Gaia DR3 data.

## When to use

Use when reproducing or inspecting Milky Way disk kinematic substructure — local velocity-space moving groups or disk-wide median radial velocity maps — from a non-axisymmetric bar+spiral potential, or comparing such plots to Gaia DR3 observations.

## When NOT to use

- When the goal is fitting bar/spiral parameters to new data rather than visualizing a pre-configured model — use the parameter-adjustment workflow instead of the default fiducial run
- When only a quick check is needed and multi-hour runtime is unacceptable without lowering bins_model to 16+1
- When working outside the Milky Way disk context or with axisymmetric potentials — the notebook is built for the backward-integration non-axisymmetric bar+two-spiral-modes model

## Inputs

- SPIBACK notebook files (spiback_minimalversion_run.ipynb and companion files) placed in a single folder
- Model parameters (bar and spiral arm configuration; defaults = fiducial model of Khalil et al. 2025)
- bins_model resolution parameter (default 32+1; 16+1 for quick tests; 64+1 for data-equivalent resolution)

## Outputs

- 2D local velocity-space distribution at the Sun's position (displayed plot, exportable as PNG/PDF)
- Median Galactocentric radial velocity map across the area of the Galactic disk probed with Gaia DR3 (displayed plot, exportable as PNG/PDF)

## How to apply

Install the SPIBACK dependencies (PyTorch, AGAMA via pip install agama, torchdiffeq, SciPy, Matplotlib) under Python 3.10+, then download the SPIBACK notebook files from github.com/yrkhalil/SPIBACK into a single folder and run all cells of spiback_minimalversion_run.ipynb in Jupyter. Keep the model parameters at the fiducial-model defaults (bar pattern speed 37 km/s/kpc plus a two-armed spiral mode at 13.1 km/s/kpc with ~25% local contrast and a three-armed mode at 16.4 km/s/kpc with ~9% local contrast); leave bins_model at 32+1 (about 2 hours of computation), using 16+1 only for quick tests. The notebook executes the backward integration of the non-axisymmetric potential and displays the 2D local velocity-space distribution at the Sun's position and the median Galactocentric radial velocity map across the disk area probed with Gaia DR3. Export the plots as PNG/PDF and compare against the corresponding figures of arXiv:2411.12800 to confirm the fiducial model reproduces the reported kinematic features (moving groups, phase-space ridges, radial velocity gradients).

## Related tools

- **SPIBACK** (Jupyter Notebook that generates the Milky Way backward-integration model and displays the velocity-space plots) — https://github.com/yrkhalil/SPIBACK
- **PyTorch** (Required dependency for running the backward-integration computation) — https://pytorch.org
- **AGAMA** (Galactic potential/orbit-integration library required by the notebook; installed via pip install agama) — https://github.com/GalacticDynamics-Oxford/Agama
- **torchdiffeq** (ODE-integration dependency for the backward integration, installed via pip install torchdiffeq)
- **SciPy** (Supporting scientific computing dependency)
- **Matplotlib** (Renders the 2D velocity distribution and radial velocity map plots)

## Examples

```
pip install agama torchdiffeq; jupyter lab  # then run all cells of spiback_minimalversion_run.ipynb (bins_model=32+1)
```

## Evaluation signals

- All cells of spiback_minimalversion_run.ipynb run without error under Python 3.10+ and plots display properly
- The 2D local velocity-space distribution shows the moving-group peaks/ridges consistent with the fiducial model figures in arXiv:2411.12800
- The median Galactocentric radial velocity map covers the same disk area probed with Gaia DR3 and reproduces the reported spiral-arm-driven radial velocity gradients
- Exported PNG/PDF plots match the corresponding figures of the source paper qualitatively
- With bins_model=32+1 the run completes in roughly 2 hours, matching the documented runtime

## Limitations

- Currently only a minimal version of the code is available for quick computations; a fuller version will be provided soon
- bins_model=32+1 takes about 2 hours to complete; 64+1 (data-equivalent resolution) significantly increases computation time
- The model reproduces phase-space features qualitatively (a posteriori check), not as a quantitative fit
- Requires Python 3.10 or above and the specific dependency set (PyTorch, AGAMA, torchdiffeq, SciPy, Matplotlib)

## Evidence

- [intro] The SPIral arms & Bar bACKward integrations (SPIBACK) Jupyter Notebook generates Milky Way models: "The SPIral arms & Bar bACKward integrations (SPIBACK) Jupyter Notebook generates Milky Way models"
- [intro] plot the 2D local velocity space distribution: "allows users to plot the 2D local velocity space distribution at the Sun's position"
- [abstract] median Galactocentric radial velocity maps: "median Galactocentric radial velocity maps across the area of the Galactic disk probed with Gaia DR3"
- [readme] Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly.: "Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly. This is a Python3 notebook, which needs Python 3.10 or above."
- [readme] bins_model default and runtime: "This is set by default to 32+1 taking about 2 hours to complete. A value of 16+1 can be used for quick tests."
- [readme] minimal version only: "Currently only a minimal version of the code is available for quick computations. A fuller version will be provided soon."
