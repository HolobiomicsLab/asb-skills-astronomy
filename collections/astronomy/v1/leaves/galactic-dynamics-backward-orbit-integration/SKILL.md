---
name: galactic-dynamics-backward-orbit-integration
description: Use when you need to model Milky Way disk kinematics with a bar and spiral
  arms constrained by Gaia DR3 radial-velocity features (e.g., the ridges of the local
  velocity distribution or median Galactocentric radial velocity maps), via backward
  orbit integration.
when_to_use_negative:
- Only a minimal version of the code is currently available for quick computations,
  so workflows needing the fuller feature set of the model should wait for the fuller
  release rather than forcing the minimal notebook
- You need fast, repeated parameter scans — the fiducial bins_model=32+1 setting takes
  about 2 hours and 64+1 significantly increases computation time
- Your analysis targets purely axisymmetric Milky Way potentials without bar/spiral
  perturbations — the skill is specific to non-axisymmetric backward-integration modeling
edam_operation: http://edamontology.org/operation_2945
tools:
- name: SPIBACK
  role: Jupyter notebook implementing the backward-integration Milky Way model and
    the kinematic plots
  repo: github.com/yrkhalil/SPIBACK
- name: PyTorch
  role: Core dependency for the differentiable backward orbit integration
  repo: https://pytorch.org
- name: AGAMA
  role: Galactic dynamics library used for the Milky Way potential (installed via
    pip install agama)
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: torchdiffeq
  role: ODE solver library used for the backward integration (installed via pip install
    torchdiffeq)
- name: Gaia DR3
  role: Stellar kinematic dataset whose in-plane motions the model is compared against
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:1982a74e5c52ee046b7d9874fbc52a4124bf254347b17a316c91df32738ecd49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AGAMA
  - Gaia DR3
  - Matplotlib
  - PyTorch
  - Python
  - SPIBACK
  - SciPy
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# galactic-dynamics-backward-orbit-integration

## Summary

Generates Milky Way disk models with a non-axisymmetric potential (bar + spiral modes) via the backward integration method, using the SPIBACK Jupyter notebook, and reproduces the fiducial model of Khalil et al. 2025 (bar pattern speed 37 km/s/kpc, two-armed and three-armed spiral modes) fitted to Gaia DR3 kinematics.

## When to use

Use when you need to model Milky Way disk kinematics with a bar and spiral arms constrained by Gaia DR3 radial-velocity features (e.g., the ridges of the local velocity distribution or median Galactocentric radial velocity maps), via backward orbit integration.

## When NOT to use

- Only a minimal version of the code is currently available for quick computations, so workflows needing the fuller feature set of the model should wait for the fuller release rather than forcing the minimal notebook
- You need fast, repeated parameter scans — the fiducial bins_model=32+1 setting takes about 2 hours and 64+1 significantly increases computation time
- Your analysis targets purely axisymmetric Milky Way potentials without bar/spiral perturbations — the skill is specific to non-axisymmetric backward-integration modeling

## Inputs

- spiback_minimalversion_run.ipynb notebook files from github.com/yrkhalil/SPIBACK placed in a single folder
- Model parameters: bar pattern speed (default 37 km/s/kpc), two-armed and three-armed spiral modes (pattern speeds 13.1 and 16.4 km/s/kpc, ~25% and ~9% local contrast density)
- bins_model grid setting (default 32+1; 16+1 for quick tests; 64+1 for data-matching resolution)

## Outputs

- 2D local velocity-space distribution at the Sun's position (moving groups and phase-space ridges), exportable as PNG/PDF
- Median Galactocentric radial velocity map across the Galactic disk area probed with Gaia DR3, exportable as PNG/PDF

## How to apply

Set up a Python 3.10+ environment and install PyTorch, AGAMA (pip install agama), torchdiffeq, SciPy, and Matplotlib; place all SPIBACK files from github.com/yrkhalil/SPIBACK in a single folder and run every cell of spiback_minimalversion_run.ipynb in Jupyter. Keep the default parameters to reproduce the fiducial model — a bar with pattern speed 37 km/s/kpc plus a two-armed spiral mode (pattern speed 13.1 km/s/kpc, ~25% local contrast surface density) and a three-armed mode (pattern speed 16.4 km/s/kpc, ~9% local contrast density). Set bins_model to 32+1 (about 2 hours runtime) for the fiducial computation, 16+1 for quick tests, or 64+1 for resolution matching the Gaia DR3 data plots at much higher cost. The backward integration adjusts the bar to selected peaks of the solar-neighborhood velocity distribution and the spiral amplitude, phase, pitch angle, and pattern speed to the median radial velocity as a function of disk position; the notebook then plots the 2D local velocity-space distribution at the Sun and the median Galactocentric radial velocity map over the Gaia DR3 probed disk area, which should be compared against the figures of arXiv:2411.12800.

## Related tools

- **SPIBACK** (Jupyter notebook implementing the backward-integration Milky Way model and the kinematic plots) — github.com/yrkhalil/SPIBACK
- **PyTorch** (Core dependency for the differentiable backward orbit integration) — https://pytorch.org
- **AGAMA** (Galactic dynamics library used for the Milky Way potential (installed via pip install agama)) — https://github.com/GalacticDynamics-Oxford/Agama
- **torchdiffeq** (ODE solver library used for the backward integration (installed via pip install torchdiffeq))
- **Gaia DR3** (Stellar kinematic dataset whose in-plane motions the model is compared against)

## Examples

```
pip install agama && pip install torchdiffeq  # then open spiback_minimalversion_run.ipynb in Jupyter and run all cells
```

## Evaluation signals

- All cells of spiback_minimalversion_run.ipynb execute without error under Python 3.10+ and plots display properly
- With default parameters, the 2D local velocity distribution shows the moving-group peaks the bar was adjusted to, and the phase-space ridges are qualitatively reproduced as a posteriori check
- The median Galactocentric radial velocity map matches the corresponding figure of the source paper (arXiv:2411.12800)
- Reproduced model matches the fiducial parameters: bar pattern speed 37 km/s/kpc, two-armed mode at 13.1 km/s/kpc, three-armed mode at 16.4 km/s/kpc
- Runtime is consistent with the chosen bins_model value (~2 hours for 32+1)

## Limitations

- Currently only a minimal version of the code is available for quick computations; a fuller version will be provided soon
- bins_model=32+1 takes about 2 hours to complete, and 64+1 (needed to match the data plotting resolution) significantly increases computation time
- The solution is validated qualitatively against moving groups and phase-space ridges, not by a quantitative statistical fit in the notebook

## Evidence

- [readme] The SPIral arms & Bar bACKward integrations (SPIBACK) Jupyter Notebook generates Milky Way models through the backward integration method, and allows users to plot the 2D local velocity space distribution: "The SPIral arms & Bar bACKward integrations (SPIBACK) Jupyter Notebook generates Milky Way models through the backward integration method, and allows users to plot the 2D local velocity space"
- [abstract] This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively.: "This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively."
- [methods] Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly.: "Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly."
- [readme] One important parameter is bins_model. This is set by default to 32+1 taking about 2 hours to complete.: "One important parameter is bins_model. This is set by default to 32+1 taking about 2 hours to complete."
- [abstract] with the backward integration method, first adjusting the bar model to selected peaks of the stellar velocity distribution in the solar neighborhood: "with the backward integration method, first adjusting the bar model to selected peaks of the stellar velocity distribution in the solar neighborhood"
