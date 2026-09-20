---
name: spatial-density-distribution-visualization
description: Use when the analysis goal is to inspect how the IPD components of a
  zodiacal light model are distributed in space — i.e., you have a supported model
  (e.
when_to_use_negative:
- The goal is to simulate zodiacal emission timestreams or binned sky maps for an
  observer (use the Model evaluation/emission workflow instead of grid_number_density)
- The task is comparing ZodiPy output against DIRBE calibrated individual observations
  (CIO) rather than visualizing model IPD number densities
- The zodiacal model is not supported by ZodiPy (coverage is limited to the DIRBE
  IPD model and the Planck extension, with frequency extrapolation)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ZodiPy
  role: Provides the grid_number_density routine that computes and visualizes IPD
    component number densities on a spatial grid from a supported model
  repo: https://github.com/Cosmoglobe/zodipy
- name: Python
  role: Runtime environment in which ZodiPy is installed via pip and the visualization
    script is executed
- name: astropy
  role: Supplies astropy.units for specifying the wavelength/frequency (e.g. 25 *
    u.micron) and coordinate/quantity handling; ZodiPy is Astropy-affiliated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:f6b06f04834ee7036ebaf335545aae6f11299a15a003a117265c0f3fcfaffde4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Cosmoglobe/zodipy
  tools:
  - Python
  - ZodiPy
  - astropy
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spatial-density-distribution-visualization

## Summary

Renders the number density distribution of interplanetary dust (IPD) components from a supported zodiacal light model (e.g., the COBE DIRBE model as implemented in ZodiPy) on a regular spatial grid, saving the result as a figure (grid_number_density.png).

## When to use

Use when the analysis goal is to inspect how the IPD components of a zodiacal light model are distributed in space — i.e., you have a supported model (e.g. DIRBE, Planck extension) and need a component-wise number density distribution over a spatial grid rather than an intensity timestream or sky map.

## When NOT to use

- The goal is to simulate zodiacal emission timestreams or binned sky maps for an observer (use the Model evaluation/emission workflow instead of grid_number_density)
- The task is comparing ZodiPy output against DIRBE calibrated individual observations (CIO) rather than visualizing model IPD number densities
- The zodiacal model is not supported by ZodiPy (coverage is limited to the DIRBE IPD model and the Planck extension, with frequency extrapolation)

## Inputs

- A supported zodiacal light model instance (zodipy.Model, e.g. the COBE DIRBE interplanetary dust model or Planck extension)
- A chosen wavelength or frequency (e.g. 25 micron) or bandpass at which to evaluate the model
- Regular spatial grid definition (e.g. astropy.units quantities for grid coordinates)

## Outputs

- Number density values for each of the model's IPD components on the spatial grid
- A rendered distribution figure saved as grid_number_density.png (valid PNG)

## How to apply

Install ZodiPy via pip (pip install zodipy), which pulls in Astropy (>= 5.0.1), NumPy, jplephem, and SciPy automatically. Initialize a supported zodiacal light model at a chosen wavelength or frequency, e.g. model = zodipy.Model(25 * u.micron) for 25 micron — ZodiPy covers the DIRBE model over 1.25–240 µm and the Planck extension over 30–857 GHz. Call the grid_number_density routine with the model to compute the number density of each of the model's IPD components on a regular spatial grid, exploiting ZodiPy's Astropy units integration. Render the component number-density distributions as a figure (e.g., with matplotlib) and save it as grid_number_density.png.

## Related tools

- **ZodiPy** (Provides the grid_number_density routine that computes and visualizes IPD component number densities on a spatial grid from a supported model) — https://github.com/Cosmoglobe/zodipy
- **Python** (Runtime environment in which ZodiPy is installed via pip and the visualization script is executed)
- **astropy** (Supplies astropy.units for specifying the wavelength/frequency (e.g. 25 * u.micron) and coordinate/quantity handling; ZodiPy is Astropy-affiliated)

## Examples

```
pip install zodipy; model = zodipy.Model(25 * u.micron); densities = model.grid_number_density(...)
```

## Evaluation signals

- grid_number_density.png exists and is a valid PNG file
- The figure depicts the number density distribution of the supported IPD components over the spatial grid
- The model initializes successfully at the chosen wavelength/frequency (e.g. 25 micron), which must lie within the supported ranges (1.25–240 µm for DIRBE, 30–857 GHz for the Planck extension)
- Every IPD component of the selected model appears in the rendered distribution, not just the total density

## Limitations

- Only zodiacal light models supported by ZodiPy can be visualized: the COBE DIRBE interplanetary dust model (1.25–240 µm) and the Planck extension (30–857 GHz), with extrapolation to other frequencies/wavelengths
- Astropy >= 5.0.1 is required (installed automatically as a ZodiPy dependency), so very old Python/Astropy environments will fail

## Evidence

- [abstract] ZodiPy is a modern and easy-to-use Python package for modeling the zodiacal emission seen by an arbitrary Solar System observer: "We present ZodiPy, a modern and easy-to-use Python package for modeling the zodiacal emission seen by an arbitrary Solar System observer"
- [intro] ZodiPy is a Python package for zodiacal light simulations.: "ZodiPy is a Python package for zodiacal light simulations."
- [methods] Initialize a zodiacal light model at a wavelength/frequency or over a bandpass: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [methods] ZodiPy is an Astropy-affiliated package for simulating zodiacal light in intensity for arbitrary solar system observers.: "ZodiPy is an [Astropy-affiliated](https://www.astropy.org/affiliated/) package for simulating zodiacal light in intensity for arbitrary solar system observers."
- [intro] Astropy integration of ZodiPy: "As an Astropy-affiliated package, ZodiPy is highly integrated with the Astropy ecosystem"
