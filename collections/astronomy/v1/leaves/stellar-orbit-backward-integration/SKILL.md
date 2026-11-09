---
name: stellar-orbit-backward-integration
description: Use when fitting Galactic bar pattern speed, spiral-arm amplitude, phase,
  pitch angle, or pattern speed to Gaia stellar kinematics (moving groups, median
  Galactocentric radial-velocity maps) via orbit integration in a non-axisymmetric
  potential.
when_to_use_negative:
- When only an axisymmetric potential is needed — the skill specifically adjusts non-axisymmetric
  bar and spiral-arm components.
- When rapid iteration is required and 64+1 bins_model resolution is set, which significantly
  increases computation time; use 16+1 for quick tests instead.
- When studying potentials other than the Milky Way disk; SPIBACK is designed to generate
  Milky Way models from Gaia DR3 kinematics.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: SPIBACK
  role: Jupyter Notebook that generates Milky Way models through the backward integration
    method and plots velocity-space distributions and radial-velocity maps
  repo: github.com/yrkhalil/SPIBACK
- name: PyTorch
  role: Required installation underpinning the notebook's differentiable orbit integration
  repo: https://pytorch.org
- name: AGAMA
  role: Galaxy-modelling library (potentials, orbit integration), installed with pip
    install agama
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: torchdiffeq
  role: ODE solver package installed via pip install torchdiffeq for the backward
    integration
- name: SciPy
  role: Supporting scientific library needed by the notebook
- name: Matplotlib
  role: Plotting of the 2D velocity-space distribution and median radial-velocity
    maps
- name: Jupyter Notebook
  role: Interface in which spiback_minimalversion_run.ipynb is run in a single folder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:6bb83ea1a89d3f023dc147e042b1b80734b127f7c9ec4dee1670cb996f2aabb4
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# stellar-orbit-backward-integration

## Summary

Constructs a non-axisymmetric Milky Way disk potential (bar plus spiral arms) by backward-integrating stellar orbits with SPIBACK, fitting bar parameters to solar-neighborhood velocity-distribution peaks and spiral-arm parameters to Gaia DR3 median radial-velocity maps.

## When to use

Use when fitting Galactic bar pattern speed, spiral-arm amplitude, phase, pitch angle, or pattern speed to Gaia stellar kinematics (moving groups, median Galactocentric radial-velocity maps) via orbit integration in a non-axisymmetric potential.

## When NOT to use

- When only an axisymmetric potential is needed — the skill specifically adjusts non-axisymmetric bar and spiral-arm components.
- When rapid iteration is required and 64+1 bins_model resolution is set, which significantly increases computation time; use 16+1 for quick tests instead.
- When studying potentials other than the Milky Way disk; SPIBACK is designed to generate Milky Way models from Gaia DR3 kinematics.

## Inputs

- Gaia DR3 in-plane motions of disk stars (solar-neighborhood stellar velocity distribution peaks)
- Median Galactocentric radial velocity maps across the Galactic disk area probed with Gaia DR3
- Bar and spiral-arm model parameters (pattern speed, amplitude, phase, pitch angle) set in spiback_minimalversion_run.ipynb

## Outputs

- Fitted non-axisymmetric Milky Way disk potential (bar + two-armed and three-armed spiral modes)
- 2D local velocity-space distribution at the Sun's position
- Median Galactocentric radial velocity map across the Galactic disk

## How to apply

Install the SPIBACK notebook stack (Python 3.10+, PyTorch, AGAMA via pip, torchdiffeq, SciPy, Matplotlib) and run all cells of spiback_minimalversion_run.ipynb in a single folder under Jupyter. The fit is two-stage: first adjust the bar model to selected peaks of the solar-neighborhood stellar velocity distribution, then adjust the spiral-arm amplitude, phase, pitch angle, and pattern speed to the median Galactocentric radial velocity as a function of disk position. Set bar pattern speed to 37 km/s/kpc and two-armed and three-armed spiral modes at pattern speeds 13.1 and 16.4 km/s/kpc for the fiducial model. Control resolution with bins_model: default 32+1 (~2 hours), 16+1 for quick tests, 64+1 for resolution equivalent to the plotted data at increased compute cost. Run all cells so the 2D local velocity-space distribution at the Sun's position and the median radial-velocity map display properly.

## Related tools

- **SPIBACK** (Jupyter Notebook that generates Milky Way models through the backward integration method and plots velocity-space distributions and radial-velocity maps) — github.com/yrkhalil/SPIBACK
- **PyTorch** (Required installation underpinning the notebook's differentiable orbit integration) — https://pytorch.org
- **AGAMA** (Galaxy-modelling library (potentials, orbit integration), installed with pip install agama) — https://github.com/GalacticDynamics-Oxford/Agama
- **torchdiffeq** (ODE solver package installed via pip install torchdiffeq for the backward integration)
- **SciPy** (Supporting scientific library needed by the notebook)
- **Matplotlib** (Plotting of the 2D velocity-space distribution and median radial-velocity maps)
- **Jupyter Notebook** (Interface in which spiback_minimalversion_run.ipynb is run in a single folder)

## Examples

```
pip install agama; pip install torchdiffeq; pip install scipy matplotlib; jupyter lab spiback_minimalversion_run.ipynb
```

## Evaluation signals

- Fiducial-model plots qualitatively reproduce Gaia DR3 moving groups and phase-space ridges reported in Khalil et al.
- Two-armed spiral mode matches the location of the Crux-Scutum, Local, and Outer arm segments; three-armed mode matches Carina-Sagittarius and Perseus segments.
- A posteriori check that the solution qualitatively reproduces features of the global non-axisymmetric phase-space distribution.
- Both the 2D local velocity-space distribution at the Sun's position and the median radial-velocity map display properly after running all cells.
- Fitted pattern speeds fall near the fiducial values: bar 37 km/s/kpc, two-armed spiral 13.1 km/s/kpc, three-armed spiral 16.4 km/s/kpc.

## Limitations

- Currently only a minimal version of the code is available for quick computations; a fuller version will be provided soon.
- bins_model = 32+1 (default) takes about 2 hours; 64+1 significantly increases computation time.
- Validation is qualitative — accepting the fit requires visual comparison of plots against Gaia DR3 moving groups and ridges rather than a numeric threshold.
- Contact yassin.khalil@unistra.fr for suggestions or feature requests.

## Evidence

- [abstract] Two-stage fitting loop: "with the backward integration method, first adjusting the bar model to selected peaks of the stellar velocity distribution in the solar neighborhood"
- [abstract] Spiral-arm fit target: "adjusting the amplitude, phase, pitch angle, and pattern speed of spiral arms to the median radial velocity as a function of position within the disk"
- [methods] Install stack: [evidence span withheld — anchor status: not_found]
- [readme] Run notebook: "Simply download the files and run them in a single folder on a Jupyter lab or Jupyter notebook interface. Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots"
- [readme] bins_model guidance: "This is set by default to 32+1 taking about 2 hours to complete. A value of 16+1 can be used for quick tests. A value of 64+1 will give a much better resolution"
- [readme] Python version: "This is a Python3 notebook, which needs Python 3.10 or above."
- [readme] Minimal version caveat: "Currently only a minimal version of the code is available for quick computations. A fuller version will be provided soon."
