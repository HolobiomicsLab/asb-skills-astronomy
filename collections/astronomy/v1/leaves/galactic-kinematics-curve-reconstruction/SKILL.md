---
name: galactic-kinematics-curve-reconstruction
description: Use when a parametric mass model of a rotationally supported (V/sigma_0
  >= 2) axisymmetric disk galaxy must be forward-modelled into an observed 1D rotation
  curve with beam-smearing corrections for PSF-convolved IFU data.
when_to_use_negative:
- Galaxies that are not rotationally supported (V/sigma_0 <~ 2) or show strong deviations
  from circular motions
- Galaxies with inclinations outside 10-80 degrees or without a well-defined kinematic
  axis
- Marginally resolved systems (PSF FWHM >~ R_eff), where discrepancies with dysmalpy
  can reach up to 15 per cent
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RotCurves
  role: Python forward-modelling package that builds the mass model, mock-observes
    the equilibrium rotation velocity given the PSF, and produces the beam-smeared
    rotation and dispersion curves
  repo: github.com/nestoramit/RotCurves
- name: dysmalpy
  role: Established parametric code used as benchmark reference for validating the
    mock-observed rotation and dispersion curves
- name: Python
  role: Runtime environment for the RotCurves package and its pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:70693d6efa69f37e3927942500672f4979a21a509946dbe7bbc2c48dcdc4899f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - dysmalpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# galactic-kinematics-curve-reconstruction

## Summary

Reconstruct a beam-smeared rotation curve for an idealized axisymmetric disk+bulge+halo galaxy by mock-observing the equilibrium rotation velocity given the sky orientation and instrumental PSF, using the RotCurves Python package (~10 ms per realization, ~250x faster than dysmalpy).

## When to use

Use when a parametric mass model of a rotationally supported (V/sigma_0 >= 2) axisymmetric disk galaxy must be forward-modelled into an observed 1D rotation curve with beam-smearing corrections for PSF-convolved IFU data.

## When NOT to use

- Galaxies that are not rotationally supported (V/sigma_0 <~ 2) or show strong deviations from circular motions
- Galaxies with inclinations outside 10-80 degrees or without a well-defined kinematic axis
- Marginally resolved systems (PSF FWHM >~ R_eff), where discrepancies with dysmalpy can reach up to 15 per cent

## Inputs

- Mass-model configuration with disk, bulge, and dark-matter halo components and selectable analytical density profiles
- Sky orientation of the galaxy (inclination 10-80 degrees, kinematic axis)
- Instrumental point-spread-function (PSF), with FWHM <= R_eff for well-resolved systems

## Outputs

- Reconstructed beam-smeared rotation curve per galaxy
- Beam-smeared dispersion curve per galaxy

## How to apply

Define the mass-model configuration with disk, bulge, and dark-matter halo components using the analytical density profiles exposed by RotCurves.mass_model and RotCurves.dm_halos, then assemble the axisymmetric galaxy with RotCurves.galaxy_model, enforcing a well-defined kinematic axis and inclination between 10 and 80 degrees. Compute the equilibrium rotation velocity of the combined mass components on the galaxy plane. Mock-observe the model with RotCurves.rotation_curve given the sky orientation and instrumental PSF, projecting and convolving the beam PSF in the plane of the galaxy to apply beam-smearing corrections. Keep the PSF FWHM <= R_eff for well-resolved systems, where agreement with the dysmalpy reference is within 5 per cent up to 3 R_eff. Emit the reconstructed beam-smeared rotation and dispersion curves per galaxy; each realization completes in ~10 ms.

## Related tools

- **RotCurves** (Python forward-modelling package that builds the mass model, mock-observes the equilibrium rotation velocity given the PSF, and produces the beam-smeared rotation and dispersion curves) — github.com/nestoramit/RotCurves
- **dysmalpy** (Established parametric code used as benchmark reference for validating the mock-observed rotation and dispersion curves)
- **Python** (Runtime environment for the RotCurves package and its pipeline)

## Examples

```
pip install rotcurves
```

## Evaluation signals

- Verify beam_smeared_curves.csv exists as the emitted output
- For well-resolved systems (PSF FWHM < R_eff), mock observed rotation and dispersion curves agree with dysmalpy to within 5 per cent up to 3 R_eff
- Each realization completes in ~10 ms; flag outliers as performance regressions
- Inclination parameter lies within the 10-80 degree validity regime
- PSF FWHM satisfies FWHM <= R_eff; otherwise expect deviations up to 15 per cent and treat the system as marginally resolved

## Limitations

- Marginally resolved systems (PSF FWHM >~ R_eff) show discrepancies with dysmalpy of up to 15 per cent
- RotCurves is best suited for inclinations between 10 and 80 degrees
- Model assumes axisymmetric disks that are rotationally supported (V/sigma_0 >~ 2) with no strong deviations from circular motions
- Fitting performs best when the beam size is the same size or smaller than the disk effective radius (PSF <~ R_eff,disk)

## Evidence

- [abstract] beam projection mock-observation: [evidence span withheld — anchor status: not_found]
- [intro] beam-projection forward model: "We simulate the projected beam on the plane of the galaxy (i.e., "beam projection"), reconstructing the beam-smeared rotation curve"
- [abstract] runtime benchmark: "The typical runtime with RotCurves is a few $\sim 10$ ms, a factor $\approx 250$ faster than dysmalpy for a single realization."
- [abstract] resolution validity check: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
- [readme] scope statement: "The fitting procedure peforms best for resolved systems, with a beam size that is of the same size or smaller than the effective radius of the disk"
