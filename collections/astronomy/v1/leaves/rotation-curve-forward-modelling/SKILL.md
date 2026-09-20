---
name: rotation-curve-forward-modelling
description: Use when you need to model or fit a 1D galactic rotation curve of a rotationally
  supported (V/sigma_0 >= 2) axisymmetric disk galaxy, and must correct for beam smearing
  given a sky orientation and instrumental PSF.
when_to_use_negative:
- The galaxy is not rotationally supported (V/sigma_0 < ~2), lacks a well-defined
  kinematic axis, or shows strong deviations from circular motions
- The inclination lies outside 10-80 degrees, or the system is marginally resolved
  (PSF FWHM >= ~1.5 R_eff) where beam-smearing discrepancies reach ~15 per cent
- You need full 2D kinematic modelling rather than 1D rotation-curve forward modelling
  (RotCurves is a lightweight exploratory tool, not a dysmalpy replacement in all
  regimes)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RotCurves
  role: 'Parametric forward-modelling package: mass_model, baryons, dm_halos, galaxy_model
    build the axisymmetric model; rotation_curve mock-observes with beam projection;
    mcmc_fitter samples posteriors'
  repo: github.com/nestoramit/RotCurves
- name: dysmalpy
  role: Established parametric code used as the benchmark reference against which
    RotCurves' beam-smeared rotation and dispersion curves are validated
- name: emcee
  role: MCMC sampler underlying RotCurves.mcmc_fitter, used to sample the posterior
    parameter distribution during Bayesian fitting
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:3a28d6cb46083b1e4e7a71a4bbcd8528ab42e8d1441f15ade33cc2012f724a53
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - dysmalpy
  - emcee
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# rotation-curve-forward-modelling

## Summary

Forward-modelling of high-z galaxy rotation curves with RotCurves: build an axisymmetric disk+bulge+halo mass model, compute the equilibrium rotation velocity, and mock-observe it with beam projection to reconstruct a beam-smeared rotation curve that can be compared directly to 1D observational data.

## When to use

Use when you need to model or fit a 1D galactic rotation curve of a rotationally supported (V/sigma_0 >= 2) axisymmetric disk galaxy, and must correct for beam smearing given a sky orientation and instrumental PSF.

## When NOT to use

- The galaxy is not rotationally supported (V/sigma_0 < ~2), lacks a well-defined kinematic axis, or shows strong deviations from circular motions
- The inclination lies outside 10-80 degrees, or the system is marginally resolved (PSF FWHM >= ~1.5 R_eff) where beam-smearing discrepancies reach ~15 per cent
- You need full 2D kinematic modelling rather than 1D rotation-curve forward modelling (RotCurves is a lightweight exploratory tool, not a dysmalpy replacement in all regimes)

## Inputs

- Parametric mass-model configuration: disk, bulge, and dark-matter halo components with selectable analytical density profiles
- Sky orientation (inclination between 10 and 80 degrees) and kinematic axis definition
- Instrumental PSF (beam FWHM, ideally <= R_eff for well-resolved systems)
- Observed 1D rotation curve with noise (for MCMC fitting)

## Outputs

- Intrinsic equilibrium rotation velocity on the galaxy plane
- Beam-smeared rotation curve and dispersion curve per galaxy
- Posterior parameter distributions (via RotCurves.mcmc_fitter/emcee) when fitting

## How to apply

Construct the mass model with RotCurves.mass_model and RotCurves.dm_halos (plus RotCurves.baryons), choosing analytical density profiles for disk, bulge, and dark-matter halo components, then assemble the galaxy with RotCurves.galaxy_model within the validity regime (inclination 10-80 degrees, V/sigma_0 >= 2). Compute the equilibrium rotation velocity on the galaxy plane out to at least 3 R_eff, then mock-observe it with RotCurves.rotation_curve given the sky orientation and instrumental PSF: the code projects the beam onto the galaxy plane and convolves to produce the beam-smeared rotation and dispersion curves, keeping PSF FWHM <= R_eff for well-resolved systems. Each realization takes ~10 ms, roughly 250x faster than dysmalpy, making the approach suited for rapid mass-model testing and large IFU survey samples. When fitting data, feed the beam-smeared model into the RotCurves.mcmc_fitter module, which samples the posterior parameter distribution with the emcee sampler under Bayesian inference.

## Related tools

- **RotCurves** (Parametric forward-modelling package: mass_model, baryons, dm_halos, galaxy_model build the axisymmetric model; rotation_curve mock-observes with beam projection; mcmc_fitter samples posteriors) — github.com/nestoramit/RotCurves
- **dysmalpy** (Established parametric code used as the benchmark reference against which RotCurves' beam-smeared rotation and dispersion curves are validated)
- **emcee** (MCMC sampler underlying RotCurves.mcmc_fitter, used to sample the posterior parameter distribution during Bayesian fitting)

## Examples

```
pip install rotcurves
```

## Evaluation signals

- For well-resolved systems (PSF FWHM < R_eff), the mock-observed rotation and dispersion curves agree with dysmalpy to within 5 per cent up to 3 R_eff
- A single realization completes in ~10 ms (runtime ratio ~250 vs dysmalpy)
- MCMC parameter recovery is consistent with intrinsic values within reported posterior uncertainties at S/N >= 25
- Recovered inclination and model parameters show no systematic bias for well-resolved, higher-mass (M_disc > ~3e9 Msun) systems; biases in R_eff and low disc masses are expected known failure modes, not validation failures of the setup
- Expected output artifacts such as beam_smeared_curves.csv and benchmark_results.csv exist and are valid CSV

## Limitations

- Systematic biases emerge in the recovered effective radius and for low disc masses (M_disc < ~3 x 10^9 M_sun)
- Marginally resolved systems (PSF FWHM >= ~1.5 R_eff) show curve discrepancies up to ~15 per cent relative to dysmalpy
- Parameter recovery degrades at low S/N; excellent recovery requires S/N > ~25
- Restricted to axisymmetric, rotationally supported (V/sigma_0 >= 2) disks with well-defined kinematic axis and no strong non-circular motions

## Evidence

- [intro] Beam projection and convolution rationale: "We simulate the projected beam on the plane of the galaxy (i.e., "beam projection"), reconstructing the beam-smeared rotation curve and fitting it directly with the observational data."
- [abstract] Benchmark agreement: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
- [abstract] Runtime speedup: "The typical runtime with RotCurves is a few $\sim 10$ ms, a factor $\approx 250$ faster than dysmalpy for a single realization."
- [readme] Validity regime: "The code is uses a forward-modelling approach to model axisymmetric disk galaxies that are rotationally supported ($V/\sigma_0 \gtrsim 2$) and have a well-defined kinematic axis, with no strong"
- [readme] Resolution guidance: "The fitting procedure peforms best for resolved systems, with a beam size that is of the same size or smaller than the effective radius of the disk $PSF \lesssim R_{\rm eff,disk}$."
