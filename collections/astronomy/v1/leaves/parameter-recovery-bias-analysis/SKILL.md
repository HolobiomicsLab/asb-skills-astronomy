---
name: parameter-recovery-bias-analysis
description: 'Use when validating a parametric fitting procedure: synthetic data with
  known ground-truth parameters are available, and you need to quantify fractional
  bias (recovered minus intrinsic) in parameters like R_eff and M_disc across S/N,
  resolution, or mass regimes.'
when_to_use_negative:
- The galaxy is not a rotationally supported axisymmetric disk (V/sigma_0 < ~2) or
  has strong deviations from circular motions / no well-defined kinematic axis
- The system is unresolved, with beam size larger than the disk effective radius (PSF
  >> R_eff), where the fitting procedure does not perform best
- Inclination is outside the 10-80 degree range where RotCurves is best suited
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RotCurves
  role: Builds galaxy/mass models, generates beam-smeared mock rotation curves, and
    provides the built-in emcee-based MCMC fitter for parameter recovery
  repo: https://github.com/nestoramit/RotCurves
- name: emcee
  role: Monte-Carlo Markov-Chain sampler used to sample the posterior parameter distribution
    during fitting
- name: Python
  role: Implementation language for the modelling, mock-observation, fitting, and
    bias-aggregation scripts
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:9c650304b0f79c2b3b679987891729baff1489e9872313e07eaa7bafc4b9457d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nestoramit/RotCurves
  tools:
  - Python
  - RotCurves
  - emcee
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# parameter-recovery-bias-analysis

## Summary

Assess whether a forward-modelling + MCMC fitting pipeline recovers intrinsic model parameters without systematic biases, by fitting beam-smeared synthetic rotation curves with known ground truth and comparing recovered versus true parameters across galaxy-property regimes.

## When to use

Use when validating a parametric fitting procedure: synthetic data with known ground-truth parameters are available, and you need to quantify fractional bias (recovered minus intrinsic) in parameters like R_eff and M_disc across S/N, resolution, or mass regimes.

## When NOT to use

- The galaxy is not a rotationally supported axisymmetric disk (V/sigma_0 < ~2) or has strong deviations from circular motions / no well-defined kinematic axis
- The system is unresolved, with beam size larger than the disk effective radius (PSF >> R_eff), where the fitting procedure does not perform best
- Inclination is outside the 10-80 degree range where RotCurves is best suited

## Inputs

- Synthetic axisymmetric galaxy mass models (disk, bulge, dark-matter halo components with known ground-truth parameters)
- Intrinsic 1D rotation curves from RotCurves.rotation_curve
- Sky-orientation and instrumental PSF specifications (beam FWHM, inclination 10-80 degrees)
- Noise realizations at defined S/N levels (e.g. S/N ~ 25 and lower)

## Outputs

- Per-realization recovery table of true versus recovered parameters with posterior medians and uncertainties (e.g. mcmc_recovery_table.csv)
- Fractional-bias summary per parameter and regime (e.g. bias_table.csv)

## How to apply

Build axisymmetric galaxy mass models (disk + bulge + dark-matter halo components) with known ground-truth parameters using RotCurves.galaxy_model and RotCurves.mass_model, defining a low-mass set (M_disc <= 3e9 Msun) and a higher-mass control set. Mock-observe the equilibrium rotation velocity given sky-orientation and instrumental PSF via RotCurves.rotation_curve, reconstructing beam-smeared rotation curves for well-resolved (PSF FWHM < R_eff) and marginally resolved systems at inclinations 10-80 degrees. Add Gaussian noise at several signal-to-noise conditions including S/N ~ 25 and lower, then fit each synthetic curve with RotCurves.mcmc_fitter (emcee-based Bayesian posterior sampling). Compute fractional bias (recovered minus intrinsic) per realization — expect excellent recovery at S/N >= 25 with deviations at lower S/N, and systematic biases for effective radius and low disc masses (M_disc < ~3e9 Msun) while higher-mass controls remain unbiased.

## Related tools

- **RotCurves** (Builds galaxy/mass models, generates beam-smeared mock rotation curves, and provides the built-in emcee-based MCMC fitter for parameter recovery) — https://github.com/nestoramit/RotCurves
- **emcee** (Monte-Carlo Markov-Chain sampler used to sample the posterior parameter distribution during fitting)
- **Python** (Implementation language for the modelling, mock-observation, fitting, and bias-aggregation scripts)

## Examples

```
python -m rotcurves.mcmc_fitter --model mock_galaxy.yaml --snr 25 --fit disk,bulge,halo --output mcmc_recovery_table.csv
```

## Evaluation signals

- Recovery at S/N >= 25 conditions shows recovered parameters consistent with intrinsic values within reported posterior uncertainties
- Low-mass set (M_disc <= ~3x10^9 Msun) reproduces systematic biases in effective radius and disc mass recovery, while the higher-mass control set shows unbiased recovery
- Well-resolved systems (PSF FWHM < R_eff) show mock rotation/dispersion curves agreeing with reference (dysmalpy) within 5 per cent up to 3 R_eff
- Recovery table is valid CSV with posterior medians and uncertainties for disk, bulge, and halo parameters against true values per condition

## Limitations

- Systematic biases emerge in the recovered effective radius and for low disc masses (M_disc < ~3 x 10^9 Msun); recovery is not unbiased in all regimes
- At lower S/N (below ~25), recovered parameters show increasing deviations from intrinsic values
- Marginally resolved systems show mock-curve discrepancies up to 15 per cent, degrading recovery fidelity
- RotCurves is best suited for inclinations of 10-80 degrees

## Evidence

- [abstract] Excellent recovery at high S/N with low-mass bias: "We show excellent parameter recovery at high signal-to-noise ratios (S/N $\gtrsim 25$ ), with increasing deviations in parameter recovery at lower S/N."
- [abstract] Systematic biases for R_eff and low disc masses: "Systematic biases emerge for the effective radius and for low disc masses ($M_{\rm disc} \lesssim 3 \times 10^{9} \, \rm M_\odot$)."
- [intro] MCMC posterior sampling approach: "We perform the fit with a Bayesian inference approach, using a Monte-Carlo Markov-Chain (MCMC, with $\texttt{emcee}$) algorithm to sample the posterior parameters distribution."
- [intro] Beam projection / beam smearing reconstruction: "We simulate the projected beam on the plane of the galaxy (i.e., "beam projection"), reconstructing the beam-smeared rotation curve"
- [readme] Best performance for resolved systems: "The fitting procedure peforms best for resolved systems, with a beam size that is of the same size or smaller than the effective radius of the disk"
