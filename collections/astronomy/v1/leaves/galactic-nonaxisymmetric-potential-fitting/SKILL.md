---
name: galactic-nonaxisymmetric-potential-fitting
description: Use when reconstructing a non-axisymmetric Galactic potential from Gaia
  DR3 disk-star kinematics — e.g. matching moving groups, phase-space ridges, or median
  Galactocentric radial-velocity maps with bar and spiral-arm parameters.
when_to_use_negative:
- Only a minimal version of the code is available — for fuller modelling needs (beyond
  quick computations) the code is insufficient until the fuller version is released
- Axisymmetric potential fitting or problems where the bar/spiral perturbations can
  be neglected — backward integration against moving groups targets specifically non-axisymmetric
  phase-space features
- When sub-2-hour computation is impossible and only quick tests are acceptable —
  bins_model=64+1 (data-equivalent resolution) significantly increases computation
  time
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3322
- http://edamontology.org/topic_3315
tools:
- name: SPIBACK
  role: Jupyter Notebook that generates Milky Way models through the backward integration
    method and plots the 2D local velocity distribution and radial-velocity maps
  repo: https://github.com/yrkhalil/SPIBACK
- name: PyTorch
  role: Required dependency for the notebook's differentiable backward integration
    loop
  repo: https://pytorch.org
- name: AGAMA
  role: Galactic-dynamics library installed via pip install agama, used for potential
    and orbit computations
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: torchdiffeq
  role: ODE-solver library (pip install torchdiffeq) for the backward orbit integration
- name: SciPy
  role: Required numerical dependency of the notebook
- name: Matplotlib
  role: Required plotting dependency to display the velocity-space distribution and
    radial-velocity maps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:8caf86706d1123276bcac48d6581a6cf0a9311f6f3893a30a1b6751cfbd46f61
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3322
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# galactic-nonaxisymmetric-potential-fitting

## Summary

Fits the Milky Way's non-axisymmetric disk potential (bar plus multi-armed spiral modes) by backward integration of stellar orbits, tuning bar and spiral parameters against Gaia DR3 kinematics via the SPIBACK Jupyter Notebook.

## When to use

Use when reconstructing a non-axisymmetric Galactic potential from Gaia DR3 disk-star kinematics — e.g. matching moving groups, phase-space ridges, or median Galactocentric radial-velocity maps with bar and spiral-arm parameters.

## When NOT to use

- Only a minimal version of the code is available — for fuller modelling needs (beyond quick computations) the code is insufficient until the fuller version is released
- Axisymmetric potential fitting or problems where the bar/spiral perturbations can be neglected — backward integration against moving groups targets specifically non-axisymmetric phase-space features
- When sub-2-hour computation is impossible and only quick tests are acceptable — bins_model=64+1 (data-equivalent resolution) significantly increases computation time

## Inputs

- Gaia DR3 in-plane motions of disk stars (moving groups, phase-space ridges, median Galactocentric radial velocity maps)
- Model parameters set in spiback_minimalversion_run.ipynb (bar amplitude/pattern speed; spiral arm amplitude, phase, pitch angle, pattern speed; bins_model)

## Outputs

- 2D local velocity-space distribution at the Sun's position
- Median Galactocentric radial velocity maps across the Galactic disk area probed with Gaia DR3
- Fitted fiducial Milky Way model (bar + two spiral modes) with adjustable parameters

## How to apply

Install PyTorch, AGAMA (pip install agama), torchdiffeq, SciPy, and Matplotlib under Python 3.10+, then run all cells of spiback_minimalversion_run.ipynb in a Jupyter Lab/Notebook with the fiducial parameters (bar pattern speed 37 km/s/kpc; two-armed mode at 13.1 km/s/kpc with ~25% local contrast surface density; three-armed mode at 16.4 km/s/kpc with ~9% local contrast density). The two-stage fitting loop first adjusts the bar model to selected peaks of the solar-neighborhood stellar velocity distribution, then adjusts the spiral-arm amplitude, phase, pitch angle, and pattern speed to the median Galactocentric radial velocity as a function of disk position. Keep bins_model at the default 32+1 (about 2 hours of computation); use 16+1 for quick tests or 64+1 for resolution equivalent to the plotted data. The rationale is that backward integration of orbits in the perturbed potential reproduces the observed non-axisymmetric phase-space structure without forward N-body evolution.

## Related tools

- **SPIBACK** (Jupyter Notebook that generates Milky Way models through the backward integration method and plots the 2D local velocity distribution and radial-velocity maps) — https://github.com/yrkhalil/SPIBACK
- **PyTorch** (Required dependency for the notebook's differentiable backward integration loop) — https://pytorch.org
- **AGAMA** (Galactic-dynamics library installed via pip install agama, used for potential and orbit computations) — https://github.com/GalacticDynamics-Oxford/Agama
- **torchdiffeq** (ODE-solver library (pip install torchdiffeq) for the backward orbit integration)
- **SciPy** (Required numerical dependency of the notebook)
- **Matplotlib** (Required plotting dependency to display the velocity-space distribution and radial-velocity maps)

## Examples

```
pip install agama torchdiffeq; jupyter lab  # then run all cells of spiback_minimalversion_run.ipynb (Python >= 3.10, fiducial bins_model=32+1)
```

## Evaluation signals

- The resulting fiducial-model plots qualitatively reproduce the Gaia DR3 moving groups and phase-space ridges reported in Khalil et al. (a posteriori check)
- Fitted two-armed mode matches the Crux-Scutum, Local, and Outer arm segments; three-armed mode matches Carina-Sagittarius and Perseus
- All cells of spiback_minimalversion_run.ipynb are run so both plots display properly
- bins_model is set consistently with the intended run type: 16+1 (quick test), 32+1 (default, ~2 hours), or 64+1 (data-equivalent resolution)
- Bar pattern speed is higher than both spiral-mode pattern speeds, consistent with the bar having recently disconnected from the two arms

## Limitations

- Currently only a minimal version of the code is available for quick computations; a fuller version will be provided soon
- bins_model=32+1 (default) takes about 2 hours to complete; 64+1 significantly increases computation time
- The fit validation is qualitative (reproduction of moving groups and phase-space ridges), not a quantitative goodness-of-fit statistic

## Evidence

- [abstract] Two-stage fitting loop: "with the backward integration method, first adjusting the bar model to selected peaks of the stellar velocity distribution in the solar neighborhood"
- [abstract] Spiral parameter fitting: "adjusting the amplitude, phase, pitch angle, and pattern speed of spiral arms to the median radial velocity as a function of position within the disk"
- [abstract] Fiducial parameters: "This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively."
- [methods] Environment requirements: "Simply download the files and run them in a single folder on a Jupyter lab or Jupyter notebook interface."
- [readme] bins_model guidance: "This is set by default to 32+1 taking about 2 hours to complete. A value of 16+1 can be used for quick tests."
- [abstract] A posteriori validation: "We checked a posteriori that our solution also qualitatively reproduces various other features of the global non-axisymmetric phase-space distribution"
- [readme] Minimal-version caveat: "Currently only a minimal version of the code is available for quick computations. A fuller version will be provided soon."
