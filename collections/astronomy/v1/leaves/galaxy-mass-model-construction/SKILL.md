---
name: galaxy-mass-model-construction
description: Use when you need to define a rotationally supported (V/sigma_0 >= 2)
  axisymmetric disk galaxy's mass components and analytical density profiles as input
  to forward-model rotation curves or synthetic observations.
when_to_use_negative:
- The target galaxy is not a rotationally supported axisymmetric disk (V/sigma_0 <
  2), lacks a well-defined kinematic axis, or shows strong deviations from circular
  motions
- The system is marginally resolved (PSF FWHM > R_eff), where mock curves can deviate
  from reference codes by up to 15 per cent and the fitting procedure degrades
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3361
tools:
- name: RotCurves
  role: 'Core package: mass_model, baryons, dm_halos and galaxy_model submodules define
    components and assemble the axisymmetric model'
  repo: github.com/nestoramit/RotCurves
- name: Python
  role: Language in which the RotCurves mass-model construction and forward modeling
    are implemented
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:32aae7c2bb83332e29231d4fec9e076ac45380b035e81a6fd86f7effd77c7241
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - emcee
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# galaxy-mass-model-construction

## Summary

Construct a parametric mass model of an idealized axisymmetric disk galaxy (disk + bulge + dark-matter halo, each with selectable analytical density profiles) and compute the equilibrium rotation velocity with RotCurves, as the forward-modeling foundation for rotation-curve fitting at high-z.

## When to use

Use when you need to define a rotationally supported (V/sigma_0 >= 2) axisymmetric disk galaxy's mass components and analytical density profiles as input to forward-model rotation curves or synthetic observations.

## When NOT to use

- The target galaxy is not a rotationally supported axisymmetric disk (V/sigma_0 < 2), lacks a well-defined kinematic axis, or shows strong deviations from circular motions
- The system is marginally resolved (PSF FWHM > R_eff), where mock curves can deviate from reference codes by up to 15 per cent and the fitting procedure degrades

## Inputs

- Mass-model configuration: disk, bulge, and dark-matter halo components with selectable analytical density profiles (RotCurves.mass_model, RotCurves.baryons, RotCurves.dm_halos)
- Galaxy geometry parameters: inclination (10-80 degrees), kinematic axis, V/sigma_0 >= 2
- Instrumental PSF / beam FWHM and sky orientation for subsequent mock observation

## Outputs

- Axisymmetric galaxy model object from RotCurves.galaxy_model
- Equilibrium rotation velocity of the combined mass components on the galaxy plane

## How to apply

Assemble the mass components with RotCurves.mass_model, choosing analytical density distributions from RotCurves.baryons (disk, bulge) and RotCurves.dm_halos (dark-matter halo) for each component. Combine them into an axisymmetric galaxy via RotCurves.galaxy_model, enforcing the validity regime of a rotationally supported disk (V/sigma_0 >= 2) with a well-defined kinematic axis, inclination between 10 and 80 degrees, and no strong deviations from circular motions. Compute the equilibrium rotation velocity of the combined mass components on the galaxy plane; downstream beam projection then requires setting the sky orientation and instrumental PSF, keeping PSF FWHM <= R_eff for well-resolved systems. The resulting model can then feed RotCurves.rotation_curve for mock observation and RotCurves.mcmc_fitter (emcee-based) for parameter recovery.

## Related tools

- **RotCurves** (Core package: mass_model, baryons, dm_halos and galaxy_model submodules define components and assemble the axisymmetric model) — github.com/nestoramit/RotCurves
- **Python** (Language in which the RotCurves mass-model construction and forward modeling are implemented)

## Examples

```
import rotcurves as rc; model = rc.mass_model(config); galaxy = rc.galaxy_model(model); v = galaxy.rotation_velocity()
```

## Evaluation signals

- Model validity regime holds: disk is rotationally supported (V/sigma_0 >= 2) with inclination in 10-80 degrees and a well-defined kinematic axis
- For well-resolved systems (PSF FWHM < R_eff), mock rotation and dispersion curves agree with the dysmalpy reference within 5 per cent up to 3 R_eff
- Each model realization completes in ~10 ms (a factor ~250 faster than dysmalpy); significant runtime overruns indicate misconfiguration
- Recovered parameters from downstream MCMC fitting show no systematic bias for moderate disc masses; biases emerging at M_disc <= ~3e9 Msun are expected physical behavior, not model-construction errors

## Limitations

- Biases: systematic biases in effective radius and low disc masses (M_disc <= ~3e9 Msun) appear in MCMC parameter recovery
- Parameter recovery degrades at low signal-to-noise (S/N < ~25)
- Valid only for inclination 10-80 degrees, rotationally supported disks (V/sigma_0 >= 2), and no strong deviations from circular motion
- Beam-smearing correction requires resolved systems (PSF <= R_eff, disk); marginally resolved systems show up to 15 per cent discrepancy vs dysmalpy

## Evidence

- [intro] The code constructs a mass model of an idealized, axisymmetric galaxy with mass components following defined analytical density distributions: "The code constructs a mass model of an idealized, axisymmetric galaxy with mass components following defined analytical density distributions"
- [intro] model axisymmetric disk galaxies that are rotationally supported ($V/\sigma_0 \gtrsim 2$) and have a well-defined kinematic axis: "model axisymmetric disk galaxies that are rotationally supported ($V/\sigma_0 \gtrsim 2$) and have a well-defined kinematic axis"
- [abstract] RotCurves is best suited for inclinations of $10^\circ < i < 80^\circ$.: "RotCurves is best suited for inclinations of $10^\circ < i < 80^\circ$."
- [abstract] For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
- [readme] The model consists of a disk, a bulge and a dark matter halo, each having various available profile shapes the user can choose from.: "The model consists of a disk, a bulge and a dark matter halo, each having various available profile shapes the user can choose from."
