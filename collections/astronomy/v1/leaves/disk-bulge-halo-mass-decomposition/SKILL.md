---
name: disk-bulge-halo-mass-decomposition
description: Use when a 1D rotation curve from a rotationally supported (V/sigma_0
  >~ 2) disk galaxy must be attributed to disk, bulge, and dark-matter-halo mass components
  with fitted parameter posteriors.
when_to_use_negative:
- Galaxy is not rotationally supported (V/sigma_0 <~ 2), lacks a well-defined kinematic
  axis, or shows strong non-circular motions
- System is unresolved or marginally resolved with beam size much larger than the
  disk effective radius (PSF >> R_eff), where the fitting performs poorly
- Inclination is outside 10-80 degrees, where RotCurves is not well suited
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RotCurves
  role: Forward-modelling package providing galaxy_model, mass_model, baryons, dm_halos,
    rotation_curve, and built-in MCMC fitting of disk/bulge/halo components
  repo: github.com/nestoramit/RotCurves
- name: emcee
  role: MCMC sampler used by RotCurves' built-in fitter to sample the posterior parameter
    distributions of the mass components
- name: dysmalpy
  role: Established parametric code used to benchmark RotCurves' mock rotation and
    dispersion curves on synthetic observations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:4aefef429187d309c3334af00d44ec36dc7c1e022685e608b09fea6334aac238
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - dysmalpy
  - emcee
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# disk-bulge-halo-mass-decomposition

## Summary

Decompose a galaxy's rotation curve into disk, bulge, and dark-matter-halo contributions by building a parametric axisymmetric mass model in RotCurves and recovering component parameters via forward-modelling and emcee-based MCMC fitting of the beam-smeared curve.

## When to use

Use when a 1D rotation curve from a rotationally supported (V/sigma_0 >~ 2) disk galaxy must be attributed to disk, bulge, and dark-matter-halo mass components with fitted parameter posteriors.

## When NOT to use

- Galaxy is not rotationally supported (V/sigma_0 <~ 2), lacks a well-defined kinematic axis, or shows strong non-circular motions
- System is unresolved or marginally resolved with beam size much larger than the disk effective radius (PSF >> R_eff), where the fitting performs poorly
- Inclination is outside 10-80 degrees, where RotCurves is not well suited

## Inputs

- 1D observed rotation curve (velocity vs. radius) of a rotationally supported disk galaxy
- Galaxy sky orientation (inclination, position angle)
- Instrumental PSF / beam size (e.g., FWHM relative to R_eff)
- Prior guesses and selectable analytical density profiles for disk, bulge, and DM halo components

## Outputs

- Posterior parameter distributions (medians and uncertainties) for disk, bulge, and halo parameters
- Best-fit beam-smeared model rotation curve
- Component-wise contribution of disk, bulge, and halo to the total rotation curve

## How to apply

Construct a mass model with RotCurves.mass_model and RotCurves.galaxy_model, selecting analytical density profiles for the disk, bulge, and dark-matter halo (RotCurves.baryons, RotCurves.dm_halos). Mock-observe the equilibrium rotation velocity via RotCurves.rotation_curve, applying beam projection/convolution with the instrumental PSF and the galaxy's sky orientation to reconstruct the beam-smeared rotation curve. Fit the model to the observed curve with RotCurves' built-in Bayesian MCMC procedure (emcee) to sample the posterior parameter distributions. Record posterior medians and uncertainties per component; verify that recovered parameters agree with ground truth within posterior uncertainties for well-resolved, high-S/N (S/N >~ 25) cases, and restrict results to inclinations of 10-80 degrees.

## Related tools

- **RotCurves** (Forward-modelling package providing galaxy_model, mass_model, baryons, dm_halos, rotation_curve, and built-in MCMC fitting of disk/bulge/halo components) — github.com/nestoramit/RotCurves
- **emcee** (MCMC sampler used by RotCurves' built-in fitter to sample the posterior parameter distributions of the mass components)
- **dysmalpy** (Established parametric code used to benchmark RotCurves' mock rotation and dispersion curves on synthetic observations)

## Examples

```
pip install rotcurves; import RotCurves; gm = RotCurves.galaxy_model(...); mm = RotCurves.mass_model(gm, disk=RotCurves.baryons.ExponentialDisk(...), halo=RotCurves.dm_halos.NFW(...))
```

## Evaluation signals

- Posterior medians for disk, bulge, and halo parameters agree with ground-truth/intrinsic values within reported posterior uncertainties at S/N >~ 25
- Mock rotation and dispersion curves agree with dysmalpy benchmarks within 5% up to 3 R_eff for well-resolved systems (PSF FWHM < R_eff)
- MCMC chains converge (emcee posterior sampling is well mixed) and parameter corner plots are bounded
- Fitted beam-smeared curve reproduces the observed curve when re-simulated through the same PSF projection
- Fits flagged as suspect when best-fit disc mass falls below ~3x10^9 M_sun or effective-radius biases are detected

## Limitations

- Systematic biases emerge for the effective radius and for low disc masses (M_disc <~ 3x10^9 M_sun)
- Parameter recovery degrades at low signal-to-noise (increasing deviations below S/N ~ 25)
- Marginally resolved systems show discrepancies up to 15% relative to dysmalpy benchmarks
- Best suited for inclinations of 10-80 degrees and for resolved systems with PSF <~ R_eff,disk
- Assumes axisymmetric, rotationally supported disks (V/sigma_0 >~ 2) with no strong deviations from circular motions

## Evidence

- [intro] Construct a mass model of an idealized axisymmetric galaxy with disk, bulge, and dark matter halo components with selectable analytical density profiles: "The code constructs a mass model of an idealized, axisymmetric galaxy with mass components following defined analytical density distributions"
- [intro] MCMC fitting with emcee: "using a Monte-Carlo Markov-Chain (MCMC, with $\texttt{emcee}$) algorithm to sample the posterior parameters distribution"
- [abstract] Excellent parameter recovery at S/N >~ 25: "We show excellent parameter recovery at high signal-to-noise ratios (S/N $\gtrsim 25$ ), with increasing deviations in parameter recovery at lower S/N."
- [abstract] Biases for effective radius and low disc masses: "Systematic biases emerge for the effective radius and for low disc masses ($M_{\rm disc} \lesssim 3 \times 10^{9} \, \rm M_\odot$)."
- [intro] Beam projection procedure: "We simulate the projected beam on the plane of the galaxy (i.e., "beam projection"), reconstructing the beam-smeared rotation curve"
- [readme] Scope: rotationally supported axisymmetric disks: "model axisymmetric disk galaxies that are rotationally supported ($V/\sigma_0 \gtrsim 2$) and have a well-defined kinematic axis, with no strong deviations from circular motions"
