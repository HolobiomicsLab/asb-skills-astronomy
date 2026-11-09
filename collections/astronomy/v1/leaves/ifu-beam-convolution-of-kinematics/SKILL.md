---
name: ifu-beam-convolution-of-kinematics
description: Use when a 1D rotation curve from IFU observations must be compared to
  a disk+bulge+halo mass model, and instrumental beam smearing (PSF convolution) must
  be applied to the model before fitting.
when_to_use_negative:
- The system is marginally resolved (PSF FWHM > R_eff) — discrepancies with dysmalpy
  reach up to 15 per cent
- The galaxy is not a rotationally supported axisymmetric disk (V/sigma_0 <~ 2) or
  has strong deviations from circular motions / no well-defined kinematic axis
- Inclination is outside the 10-80 degree range where RotCurves is best suited
edam_operation: http://edamontology.org/operation_0364
tools:
- name: RotCurves
  role: Constructs the disk+bulge+halo mass model, computes the equilibrium rotation
    velocity, and mock-observes it with beam projection/PSF convolution to produce
    the beam-smeared rotation curve
  repo: github.com/nestoramit/RotCurves
- name: dysmalpy
  role: Established parametric benchmark code used to validate the beam-smeared rotation
    and dispersion curves from RotCurves
- name: Python
  role: Implementation language of RotCurves; the beam-convolution workflow is executed
    as Python calls to the RotCurves submodules
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:282cd0cf1509c3f84bba51f25781381684bdb436144a6d048a6bb8cb2b89a2b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - dysmalpy
  edam_operation: http://edamontology.org/operation_0364
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ifu-beam-convolution-of-kinematics

## Summary

Forward-model the instrumental PSF ('beam projection') onto the plane of a parametric axisymmetric galaxy to reconstruct a beam-smeared rotation curve (and dispersion curve) from the equilibrium rotation velocity, as implemented in the RotCurves Python package for high-z IFU rotation curve analysis.

## When to use

Use when a 1D rotation curve from IFU observations must be compared to a disk+bulge+halo mass model, and instrumental beam smearing (PSF convolution) must be applied to the model before fitting.

## When NOT to use

- The system is marginally resolved (PSF FWHM > R_eff) — discrepancies with dysmalpy reach up to 15 per cent
- The galaxy is not a rotationally supported axisymmetric disk (V/sigma_0 <~ 2) or has strong deviations from circular motions / no well-defined kinematic axis
- Inclination is outside the 10-80 degree range where RotCurves is best suited

## Inputs

- Mass-model configuration (disk, bulge, dark-matter halo components with selectable analytical density profiles)
- Galaxy sky orientation (inclination 10-80 degrees, well-defined kinematic axis)
- Instrumental point-spread-function (PSF), preferably FWHM <= R_eff of the disk

## Outputs

- Beam-smeared (mock-observed) rotation curve per galaxy
- Dispersion curve per galaxy
- Each realization computed in ~10 ms

## How to apply

First define the mass-model configuration — disk, bulge, and dark-matter halo components with selectable analytical density profiles — using RotCurves.mass_model and RotCurves.dm_halos, and assemble the axisymmetric galaxy with RotCurves.galaxy_model, staying inside the tool's validity regime of a rotationally supported disk (V/sigma_0 >= 2) with a well-defined kinematic axis and inclination between 10 and 80 degrees. Compute the equilibrium rotation velocity of the combined mass components on the galaxy plane. Then mock-observe this velocity field with RotCurves.rotation_curve given the sky orientation and the instrumental PSF: the projected beam is simulated on the plane of the galaxy ('beam projection') and the PSF is convolved in that plane to produce the beam-smeared rotation curve. Keep PSF FWHM <= R_eff so the system is well resolved; each realization takes ~10 ms, enabling rapid exploratory testing and MCMC fitting.

## Related tools

- **RotCurves** (Constructs the disk+bulge+halo mass model, computes the equilibrium rotation velocity, and mock-observes it with beam projection/PSF convolution to produce the beam-smeared rotation curve) — github.com/nestoramit/RotCurves
- **dysmalpy** (Established parametric benchmark code used to validate the beam-smeared rotation and dispersion curves from RotCurves)
- **Python** (Implementation language of RotCurves; the beam-convolution workflow is executed as Python calls to the RotCurves submodules)

## Examples

```
pip install rotcurves
```

## Evaluation signals

- beam_smeared_curves.csv exists for the realization
- For well-resolved systems (PSF FWHM < R_eff), the mock observed rotation and dispersion curves agree with the dysmalpy reference to within 5 per cent up to 3 R_eff
- Inclination of the assembled model lies within 10-80 degrees and the disk satisfies V/sigma_0 >= 2
- PSF FWHM used for convolution is <= R_eff of the disk
- Per-realization runtime is on the order of ~10 ms

## Limitations

- Marginally resolved systems (PSF FWHM >~ R_eff) show discrepancies up to 15 per cent relative to dysmalpy
- RotCurves assumes axisymmetric disks that are rotationally supported (V/sigma_0 >~ 2) with a well-defined kinematic axis and no strong deviations from circular motions
- Systematic biases emerge in MCMC parameter recovery for the effective radius and for low disc masses (M_disc <~ 3x10^9 M_sun), and recovery degrades at S/N <~ 25

## Evidence

- [abstract] correcting for ‘beam smearing’ by projecting and convolving the beam PSF in the plane of the galaxy: "correcting for ‘beam smearing’ by projecting and convolving the beam PSF in the plane of the galaxy"
- [intro] mock-observed equilibrium velocity with PSF: "The equilibrium rotation velocity is mock-observed given the sky-orientation and the instrumental point-spread-function (PSF)."
- [intro] beam projection reconstruction: "We simulate the projected beam on the plane of the galaxy (i.e., "beam projection"), reconstructing the beam-smeared rotation curve"
- [readme] resolved systems fitting condition: "The fitting procedure peforms best for resolved systems, with a beam size that is of the same size or smaller than the effective radius of the disk"
- [abstract] well-resolved validation tolerance: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
