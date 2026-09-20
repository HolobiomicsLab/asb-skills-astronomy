---
name: stellar-limb-darkening-parameter-fitting
description: Use when fitting transit light curves (e.g. Kepler long-cadence photometry)
  where limb darkening controls the transit shape and you want per-planet posterior
  constraints on limb-darkening coefficients alongside Period, Inclination, a/Rs,
  and Rp/Rs.
when_to_use_negative:
- When you need to phase-fold and rebin multiple light curves into a single stacked
  light curve — TAP is designed for simultaneous analysis of multiple light curves
  of varying cadence, filter, and noise conditions with no need to phase fold and
  rebin.
- When limb-darkening coefficients must be fixed from stellar atmosphere models rather
  than fitted (e.g. if the light curve quality is insufficient to constrain them).
- When the analysis goal is eclipse or secondary-transit modeling rather than primary
  transit light-curve fitting.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Transit Analysis Package (TAP)
  role: MCMC fitting of light curves with the Mandel and Agol (2002) analytic model,
    with limb-darkening parameters left free during sampling
  repo: https://github.com/zgazak/TAP
- name: IDL
  role: Runtime environment hosting the TAP graphical user interface used to configure
    and run the fit
- name: EXOFAST
  role: Provides the improved implementation of the Mandel and Agol (2002) analytic
    transit model used inside TAP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:aa60f0538d631d0d827296ccafc68b3d6f9783fa5df08278f4375bb83ea12be5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# stellar-limb-darkening-parameter-fitting

## Summary

Fit stellar limb-darkening parameters as free parameters within an MCMC transit fit using the Mandel and Agol (2002) analytic model, so that limb-darkening uncertainty propagates honestly into transit parameter posteriors instead of being fixed from stellar atmosphere models.

## When to use

Use when fitting transit light curves (e.g. Kepler long-cadence photometry) where limb darkening controls the transit shape and you want per-planet posterior constraints on limb-darkening coefficients alongside Period, Inclination, a/Rs, and Rp/Rs.

## When NOT to use

- When you need to phase-fold and rebin multiple light curves into a single stacked light curve — TAP is designed for simultaneous analysis of multiple light curves of varying cadence, filter, and noise conditions with no need to phase fold and rebin.
- When limb-darkening coefficients must be fixed from stellar atmosphere models rather than fitted (e.g. if the light curve quality is insufficient to constrain them).
- When the analysis goal is eclipse or secondary-transit modeling rather than primary transit light-curve fitting.

## Inputs

- Kepler long-cadence photometry (29.4-minute cadence) from the MAST archive
- Ground-based transit light curves (e.g. multi-filter light curves of WASP-10b)
- Configuration of transit model settings (circular orbits locked, limb-darkening parameters free)

## Outputs

- Posterior distributions of Period, Inclination, a/Rs, and Rp/Rs per planet
- Per-planet median and 1-sigma parameter summaries (e.g. tap_kepler_4_8_parameters.csv)
- Fractional-difference comparison versus published values (Eq. 5, KB10)

## How to apply

Configure the TAP MCMC so limb-darkening parameters are left free rather than locked, while the Mandel and Agol (2002) analytic transit model (implemented via EXOFAST) is evaluated at each chain step. Use the Carter and Winn (2009) wavelet-based likelihood in place of classic chi-squared so that both uncorrelated (white) and correlated (red) noise are parameterized, since limb-darkening degeneracies with impact parameter inflate uncertainties if noise is mis-modeled. For long-integration data such as Kepler 29.4-minute cadence light curves, compute the transit model on a 10x finer temporal scale and rebin to the observed cadence before comparing to data, because integration smearing distorts the ingress/egress shape that limb darkening constrains. Sample the joint posterior of Period, Inclination, a/Rs, Rp/Rs, and the limb-darkening coefficients, and report per-parameter medians with 1-sigma intervals. Judge success by statistical agreement with independent published analyses, e.g. a fractional difference of absolute value less than unity against the A.c method of Kipping and Bakos (2010) for Kepler-4b through -7b.

## Related tools

- **Transit Analysis Package (TAP)** (MCMC fitting of light curves with the Mandel and Agol (2002) analytic model, with limb-darkening parameters left free during sampling) — https://github.com/zgazak/TAP
- **IDL** (Runtime environment hosting the TAP graphical user interface used to configure and run the fit)
- **EXOFAST** (Provides the improved implementation of the Mandel and Agol (2002) analytic transit model used inside TAP)

## Evaluation signals

- Fractional difference (FD) versus independently published values (e.g. KB10 A.c method) satisfies |FD| < 1, as reproduced for Kepler-4b through Kepler-7b.
- Fitted values match the TAP column of Table 5 within the quoted 1-sigma uncertainty.
- Significant discrepancies are flagged rather than suppressed, e.g. the Kepler-8b period difference should surface as a flagged result.
- Wavelet-based likelihood yields parameter uncertainties that account for both white and red noise, so quoted 1-sigma intervals are wider than a classic chi-squared fit would give for correlated-noise data.

## Limitations

- Different fitting methods can lead to variations in derived quantities, and confidence quantification depends critically on the treatment of noise, stellar limb darkening, and cadence.
- Kepler-8b showed a significant period discrepancy in the TAP-vs-KB10 comparison, demonstrating that agreement is not guaranteed for all targets.
- Long-integration effects (e.g. Kepler 29.4-minute cadence) distort the transit model unless it is computed on a finer temporal scale and integrated to the observed cadence.

## Evidence

- [abstract] The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002).: "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [intro] For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
- [intro] Instead of the classic χ2, TAP uses the wavelet-based likelihood function of [17], which parameterizes both uncorrelated and correlated photometric noise allowing for accurate uncertainties: [evidence span withheld — anchor status: not_found]
- [abstract] Reanalysis of ground-based observations of TrES-1b, WASP-4b, and WASP-10b and space-based Kepler 4b-8b show good agreement between TAP and prior publications.: [evidence span withheld — anchor status: not_found]
