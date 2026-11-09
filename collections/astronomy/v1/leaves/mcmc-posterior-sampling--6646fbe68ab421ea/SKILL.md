---
name: mcmc-posterior-sampling
description: Use when fitting a 1D (optionally beam-smeared) galaxy rotation curve
  to recover physical mass-model parameters, or when validating MCMC parameter recovery
  on synthetic rotation curves at known S/N levels.
when_to_use_negative:
- The system is unresolved or marginally resolved with beam size larger than the disk
  effective radius (fitting performs best for PSF ≲ R_eff,disk)
- The galaxy is not rotationally supported (V/sigma_0 ≲ 2), lacks a well-defined kinematic
  axis, or shows strong deviations from circular motions
- The inclination is outside 10° < i < 80°, where RotCurves is not best suited
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RotCurves
  role: Forward-modelling package providing galaxy_model, mass_model, rotation_curve
    (with beam-smearing correction), and the built-in mcmc_fitter module
  repo: github.com/nestoramit/RotCurves
- name: emcee
  role: Monte-Carlo Markov-Chain sampler used to sample the posterior parameter distribution
    during the Bayesian fit
- name: dysmalpy
  role: Established parametric benchmark code used to validate RotCurves' synthetic
    observations and MCMC recovery
- name: Python
  role: Implementation language of RotCurves and the surrounding fitting/analysis
    workflow
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:2996e9a3470427bf7019d57bb8f59ed5406ea55457b0d25ffdecfae1f3984a3e
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
yes_rate: 1.0
n_claims: 10
---

# mcmc-posterior-sampling

## Summary

Bayesian inference skill that recovers intrinsic galaxy mass-model parameters (disk, bulge, halo, effective radius, disc mass) from 1D rotation curves by sampling the posterior with an emcee-based MCMC inside the RotCurves forward-modelling package.

## When to use

Use when fitting a 1D (optionally beam-smeared) galaxy rotation curve to recover physical mass-model parameters, or when validating MCMC parameter recovery on synthetic rotation curves at known S/N levels.

## When NOT to use

- The system is unresolved or marginally resolved with beam size larger than the disk effective radius (fitting performs best for PSF ≲ R_eff,disk)
- The galaxy is not rotationally supported (V/sigma_0 ≲ 2), lacks a well-defined kinematic axis, or shows strong deviations from circular motions
- The inclination is outside 10° < i < 80°, where RotCurves is not best suited

## Inputs

- 1D rotation curve (observed or synthetic, beam-smeared)
- Galaxy mass-model configuration (disk, bulge, dark-matter halo with selectable analytical density profiles)
- Sky-orientation and instrumental PSF parameters for beam projection
- Noise level / signal-to-noise condition of the data

## Outputs

- Posterior parameter distributions (emcee chains) for disk, bulge, and halo parameters
- Posterior medians and uncertainties per fitted parameter
- True-vs-recovered parameter comparison table (e.g. mcmc_recovery_table.csv)

## How to apply

Build an axisymmetric galaxy model with RotCurves.galaxy_model and RotCurves.mass_model using disk, bulge, and dark-matter-halo components (RotCurves.baryons, RotCurves.dm_halos) with known analytical density profiles. Mock-observe the equilibrium rotation velocity given sky-orientation and instrumental PSF so that beam smearing is corrected by projecting and convolving the beam PSF in the plane of the galaxy. Fit the beam-smeared rotation curve directly with the built-in RotCurves.mcmc_fitter, using emcee to sample the posterior parameter distribution, then compare posterior medians and uncertainties against true values. Expect excellent recovery at S/N ≳ 25; be aware that systematic biases emerge for the effective radius and for low disc masses (M_disc ≲ 3×10^9 M_sun), so flag those regimes in the recovery table.

## Related tools

- **RotCurves** (Forward-modelling package providing galaxy_model, mass_model, rotation_curve (with beam-smearing correction), and the built-in mcmc_fitter module) — github.com/nestoramit/RotCurves
- **emcee** (Monte-Carlo Markov-Chain sampler used to sample the posterior parameter distribution during the Bayesian fit)
- **dysmalpy** (Established parametric benchmark code used to validate RotCurves' synthetic observations and MCMC recovery)
- **Python** (Implementation language of RotCurves and the surrounding fitting/analysis workflow)

## Examples

```
pip install rotcurves
```

## Evaluation signals

- Recovered parameters at S/N ≳ 25 conditions are consistent with intrinsic values within reported posterior uncertainties
- MCMC chains converge with well-defined posterior distributions (corner plots inspected via the corner dependency)
- Fractional bias (recovered minus intrinsic) of R_eff and M_disc shows systematic biases only for M_disc ≲ 3×10^9 M_sun, matching the paper's finding
- Output recovery table exists and is valid CSV with true and recovered parameters plus posterior uncertainties per condition
- Benchmark agreement: mock rotation and dispersion curves agree with dysmalpy within 5 per cent up to 3 R_eff for well-resolved systems

## Limitations

- Systematic biases emerge for the effective radius and for low disc masses (M_disc ≲ 3×10^9 M_sun)
- Parameter recovery degrades with increasing deviations at lower S/N (below S/N ≳ 25)
- Fitting performs best for resolved systems with PSF ≲ R_eff,disk; marginally resolved systems show discrepancies up to 15 per cent versus dysmalpy
- Scope restricted to axisymmetric, rotationally supported (V/sigma_0 ≳ 2) disks with a well-defined kinematic axis and no strong deviations from circular motions

## Evidence

- [abstract] MCMC fitting procedure: "Using a built-in MCMC fitting procedure, RotCurves recovers well the intrinsic model parameters across a wide range of galaxy properties"
- [abstract] Systematic biases: "Systematic biases emerge for the effective radius and for low disc masses ($M_{\rm disc} \lesssim 3 \times 10^{9} \, \rm M_\odot$)."
- [abstract] S/N recovery: "We show excellent parameter recovery at high signal-to-noise ratios (S/N $\gtrsim 25$ ), with increasing deviations in parameter recovery at lower S/N."
- [intro] emcee posterior sampling: "We perform the fit with a Bayesian inference approach, using a Monte-Carlo Markov-Chain (MCMC, with $\texttt{emcee}$) algorithm to sample the posterior parameters distribution."
- [readme] README install: "The package can be installed simply by the following pip command:
``npip install rotcurves
"
