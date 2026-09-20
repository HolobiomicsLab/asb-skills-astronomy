---
name: jupyter-notebook-reproducible-execution
description: Use when a paper ships a Jupyter notebook (e.g. SPIBACK) and the goal
  is to regenerate its fiducial-model outputs — the 2D local velocity-space distribution
  and median Galactocentric radial velocity map — on your own machine.
when_to_use_negative:
- Only the minimal version of the SPIBACK code is currently available for quick computations
  — do not expect the fuller version's features until it is released
- When you need high-resolution output equivalent to the plotted data but cannot afford
  the significantly increased computation time of bins_model = 64+1
- If your goal is to fit new bar/spiral parameters from data rather than reproduce
  the fiducial run — the notebook's defaults are set to Khalil et al.'s fiducial model,
  not a fitting harness
tools:
- name: SPIBACK
  role: The Jupyter notebook being executed; generates Milky Way models via the backward
    integration method and displays the velocity-distribution and radial-velocity
    plots
  repo: github.com/yrkhalil/SPIBACK
- name: Jupyter Notebook
  role: Interface in which the SPIBACK files are run in a single folder and all cells
    executed to display the plots
  repo: github.com/yrkhalil/SPIBACK
- name: PyTorch
  role: Crucially required dependency for the notebook's backward-orbit-integration
    computation
  repo: https://pytorch.org
- name: AGAMA
  role: Galactic dynamics library (pip install agama) used by the notebook for potential/force
    computation of the Milky Way model
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: torchdiffeq
  role: Pip-installed dependency for differentiable ODE integration during backward
    orbit integration
- name: Python 3.10
  role: Minimum Python version required by the notebook
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:af937af09981bac86b339569d922ca5a26f0978287209d528003c5623b32d916
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AGAMA
  - Jupyter Notebook
  - Matplotlib
  - PyTorch
  - Python
  - Python 3.10
  - SPIBACK
  - SciPy
  - torchdiffeq
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# jupyter-notebook-reproducible-execution

## Summary

Reproduce a published computational result by installing the notebook's pinned dependency stack and executing all cells of a Jupyter notebook (here, SPIBACK's spiback_minimalversion_run.ipynb) in a single folder, yielding the paper's fiducial-model plots.

## When to use

Use when a paper ships a Jupyter notebook (e.g. SPIBACK) and the goal is to regenerate its fiducial-model outputs — the 2D local velocity-space distribution and median Galactocentric radial velocity map — on your own machine.

## When NOT to use

- Only the minimal version of the SPIBACK code is currently available for quick computations — do not expect the fuller version's features until it is released
- When you need high-resolution output equivalent to the plotted data but cannot afford the significantly increased computation time of bins_model = 64+1
- If your goal is to fit new bar/spiral parameters from data rather than reproduce the fiducial run — the notebook's defaults are set to Khalil et al.'s fiducial model, not a fitting harness

## Inputs

- SPIBACK notebook files (spiback_minimalversion_run.ipynb and companion files) downloaded from github.com/yrkhalil/SPIBACK into a single folder
- Python 3.10+ environment with PyTorch, AGAMA, torchdiffeq, SciPy, and Matplotlib installed
- Notebook parameters set to the fiducial model values (e.g. bins_model = 32+1)

## Outputs

- 2D local velocity-space distribution plot at the Sun's position (exportable as PNG/PDF)
- Median Galactocentric radial velocity map across the Galactic disk area probed with Gaia DR3 (exportable as PNG/PDF)
- Backward-integration Milky Way model computed with the chosen bar and spiral-arm parameters

## How to apply

Set up Python 3.10 or above, then install the notebook's declared stack with pip: PyTorch (from pytorch.org), AGAMA (pip install agama), torchdiffeq (pip install torchdiffeq), plus SciPy and Matplotlib. Download the notebook files from github.com/yrkhalil/SPIBACK into a single folder and open them in Jupyter Lab or Jupyter notebook. Choose the bins_model value deliberately: the default 32+1 takes about 2 hours to complete, 16+1 is for quick tests, and 64+1 gives resolution equivalent to the plotted data at significantly increased computation time. Run all cells of spiback_minimalversion_run.ipynb so the backward-integration Milky Way model is computed and the plots display properly, keeping parameters at their fiducial defaults (bar pattern speed 37 km/s/kpc with two-armed and three-armed spiral modes at 13.1 and 16.4 km/s/kpc) unless deliberately varying them. Export the displayed figures and check them qualitatively against the source paper (arXiv:2411.12800) to confirm the fiducial model reproduces the reported Gaia DR3 kinematic features.

## Related tools

- **SPIBACK** (The Jupyter notebook being executed; generates Milky Way models via the backward integration method and displays the velocity-distribution and radial-velocity plots) — github.com/yrkhalil/SPIBACK
- **Jupyter Notebook** (Interface in which the SPIBACK files are run in a single folder and all cells executed to display the plots) — github.com/yrkhalil/SPIBACK
- **PyTorch** (Crucially required dependency for the notebook's backward-orbit-integration computation) — https://pytorch.org
- **AGAMA** (Galactic dynamics library (pip install agama) used by the notebook for potential/force computation of the Milky Way model) — https://github.com/GalacticDynamics-Oxford/Agama
- **torchdiffeq** (Pip-installed dependency for differentiable ODE integration during backward orbit integration)
- **Python 3.10** (Minimum Python version required by the notebook)

## Examples

```
pip install agama; pip install torchdiffeq
```

## Evaluation signals

- All cells of spiback_minimalversion_run.ipynb execute without error and both plots display properly
- Environment check: Python ≥ 3.10 with PyTorch, AGAMA, torchdiffeq, SciPy, and Matplotlib all importable
- Runtime sanity check: bins_model = 32+1 completes in roughly 2 hours; a much faster run suggests 16+1 was accidentally used
- The two output figures (2D local velocity distribution at the Sun's position; median Galactocentric radial velocity map) qualitatively reproduce the corresponding figures of the source paper (arXiv:2411.12800) and the Gaia DR3 moving groups / phase-space ridges

## Limitations

- Currently only a minimal version of the code is available for quick computations; a fuller version will be provided soon
- bins_model = 32+1 (default) takes about 2 hours, and 64+1 significantly increases computation time, so full-resolution runs are expensive
- Validation of the reproduced fiducial model is qualitative (moving groups and phase-space ridges), not quantitative

## Evidence

- [readme] Simply download the files and run them in a single folder on a Jupyter lab or Jupyter notebook interface.: "Simply download the files and run them in a single folder on a Jupyter lab or Jupyter notebook interface."
- [readme] bins_model default and runtime: "One important parameter is bins_model. This is set by default to 32+1 taking about 2 hours to complete. A value of 16+1 can be used for quick tests."
- [readme] Python version requirement: "This is a Python3 notebook, which needs Python 3.10 or above."
- [methods] Run all cells instruction: "Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly."
- [intro] Fiducial parameter defaults: "The parameters are set by default to be those of the "fiducial model", see Khalil et al for all the details"
