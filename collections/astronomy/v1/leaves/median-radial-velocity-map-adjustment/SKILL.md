---
name: median-radial-velocity-map-adjustment
description: 'Use when fitting non-axisymmetric Milky Way disk potentials: after the
  bar stage has matched solar-neighborhood velocity-distribution peaks, and median
  Galactocentric radial velocity maps from Gaia DR3 are the fitting target for spiral-arm
  parameters.'
when_to_use_negative:
- The bar stage of the backward-integration fit has not yet been adjusted to the selected
  peaks of the solar-neighborhood stellar velocity distribution — the two stages are
  sequential.
- The goal is fitting a purely axisymmetric Galactic potential with no spiral-arm
  components to adjust.
- Only a quick qualitative test is needed and the ~2-hour bins_model=32+1 default
  runtime is unacceptable without switching to 16+1.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SPIBACK
  role: Jupyter Notebook that generates Milky Way models through the backward integration
    method and plots median Galactocentric radial velocity maps
  repo: github.com/yrkhalil/SPIBACK
- name: PyTorch
  role: Required dependency for the notebook's backward orbit integration
  repo: https://pytorch.org
- name: AGAMA
  role: Galactic-dynamics library used for potential/orbit computation; installed
    via pip install agama
  repo: https://github.com/GalacticDynamics-Oxford/Agama
- name: torchdiffeq
  role: ODE solver package required for orbit integration in the notebook
- name: Gaia DR3
  role: Source of the median Galactocentric radial velocity maps used as the fitting
    target
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:4ac53058c278aa5db7b4b11e3a76deef1f2910a9993e31c4bd50c341096a7ad8
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# median-radial-velocity-map-adjustment

## Summary

Second stage of the SPIBACK backward-integration fitting loop: adjusts the spiral-arm amplitude, phase, pitch angle, and pattern speed so the modeled median Galactocentric radial velocity as a function of disk position matches the map probed with Gaia DR3.

## When to use

Use when fitting non-axisymmetric Milky Way disk potentials: after the bar stage has matched solar-neighborhood velocity-distribution peaks, and median Galactocentric radial velocity maps from Gaia DR3 are the fitting target for spiral-arm parameters.

## When NOT to use

- The bar stage of the backward-integration fit has not yet been adjusted to the selected peaks of the solar-neighborhood stellar velocity distribution — the two stages are sequential.
- The goal is fitting a purely axisymmetric Galactic potential with no spiral-arm components to adjust.
- Only a quick qualitative test is needed and the ~2-hour bins_model=32+1 default runtime is unacceptable without switching to 16+1.

## Inputs

- Gaia DR3 median Galactocentric radial velocity map across the probed area of the Galactic disk
- spiback_minimalversion_run.ipynb with bar-stage-fit model parameters (fiducial defaults)
- bins_model setting (default 32+1; 16+1 quick tests; 64+1 data-equivalent resolution)

## Outputs

- Fitted spiral-arm amplitude, phase, pitch angle, and pattern speed values
- Median Galactocentric radial velocity map plot across the Galactic disk
- 2D local velocity-space distribution at the Sun's position for a posteriori validation

## How to apply

Run backward integration of stellar orbits in the SPIBACK Jupyter Notebook (spiback_minimalversion_run.ipynb) with the bar stage already fit to selected peaks of the solar-neighborhood stellar velocity distribution. Adjust the spiral-arm amplitude, phase, pitch angle, and pattern speed so the modeled median radial velocity as a function of position within the disk matches the Gaia DR3 median Galactocentric radial velocity maps. Use the fiducial configuration: bar pattern speed 37 km/s/kpc plus a two-armed spiral mode at 13.1 km/s/kpc (~25% local contrast surface density) and a three-armed mode at 16.4 km/s/kpc (~9% local contrast density). Set bins_model to the default 32+1 (~2 hours computation), 16+1 for quick tests, or 64+1 for resolution equivalent to the plotted data at increased computation time. Run all notebook cells to display the median radial velocity map properly, then check a posteriori that the solution qualitatively reproduces Gaia DR3 moving groups and phase-space ridges.

## Related tools

- **SPIBACK** (Jupyter Notebook that generates Milky Way models through the backward integration method and plots median Galactocentric radial velocity maps) — github.com/yrkhalil/SPIBACK
- **PyTorch** (Required dependency for the notebook's backward orbit integration) — https://pytorch.org
- **AGAMA** (Galactic-dynamics library used for potential/orbit computation; installed via pip install agama) — https://github.com/GalacticDynamics-Oxford/Agama
- **torchdiffeq** (ODE solver package required for orbit integration in the notebook)
- **Gaia DR3** (Source of the median Galactocentric radial velocity maps used as the fitting target)

## Examples

```
pip install agama torchdiffeq; jupyter lab spiback_minimalversion_run.ipynb  # run all cells; bins_model=32+1
```

## Evaluation signals

- Modeled median Galactocentric radial velocity as a function of disk position qualitatively matches the Gaia DR3 median radial velocity map
- A posteriori solution reproduces Gaia DR3 moving groups and phase-space ridges reported in Khalil et al.
- Fitted spiral pattern speeds fall near the fiducial values (two-armed at 13.1 km/s/kpc, three-armed at 16.4 km/s/kpc)
- All cells of spiback_minimalversion_run.ipynb were run so the map and velocity-space plots display properly
- bins_model matches the desired resolution/computation trade-off (32+1 default, 64+1 for data-equivalent resolution)

## Limitations

- Currently only a minimal version of the SPIBACK code is available; a fuller version was not yet released at time of writing.
- bins_model=32+1 takes about 2 hours; 64+1 resolution equivalent to the plotted data significantly increases computation time.
- Validation of the fiducial model against moving groups and phase-space ridges is qualitative, not a quantitative goodness-of-fit.

## Evidence

- [abstract] Spiral-arm adjustment target: "adjusting the amplitude, phase, pitch angle, and pattern speed of spiral arms to the median radial velocity as a function of position within the disk"
- [intro] Map plotting capability: "median Galactocentric radial velocity maps across the area of the Galactic disk probed with Gaia DR3"
- [readme] bins_model guidance: "This is set by default to 32+1 taking about 2 hours to complete. A value of 16+1 can be used for quick tests. A value of 64+1 will give a much better resolution"
- [methods] Run all cells: "Run all the cells of the spiback_minimalversion_run.ipynb Notebook to display the plots properly."
