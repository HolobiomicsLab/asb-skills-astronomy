---
name: transit-model-cadence-integration
description: Use when fitting long-integration light curves (e.g. Kepler long-cadence
  photometry with ~30-min exposures) where the finite exposure smears ingress/egress
  and would bias transit parameters like Rp/Rs, a/Rs, i, and Tmid.
when_to_use_negative:
- Short-cadence or ground-based exposures where integration smearing is negligible
  relative to transit duration, so finer-scale evaluation plus integration adds cost
  without accuracy benefit
- Data that have already been phase folded and rebinned — TAP is explicitly designed
  for simultaneous analysis of multiple light curves 'with no need to phase fold and
  rebin', so folding destroys the per-curve cadence and noise information this skill
  relies on
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Transit Analysis Package (TAP)
  role: Provides the long-integration setting that computes the transit model on a
    finer temporal scale and integrates it to the observed cadence, driven through
    the IDL GUI and MCMC machinery
  repo: https://github.com/zgazak/TAP
- name: EXOFAST
  role: Improved implementation of the Mandel and Agol (2002) analytic transit model
    that TAP evaluates at sub-cadence time steps before integration
- name: IDL (Interactive Data Language)
  role: Runtime environment for the TAP GUI and MCMC pipeline that performs the cadence-integrated
    model fitting
- name: Kepler mission
  role: Source of the long-integration (long-cadence) photometry that motivates enabling
    the finer-scale integration handling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:5dc2b828dcc8a9db1ee4837dfb56504ec3e6983c613d59b973e10133ffc9c780
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - IDL (Interactive Data Language)
  - Kepler mission
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# transit-model-cadence-integration

## Summary

Corrects finite-exposure-time smearing in transit photometry by computing the Mandel and Agol (2002) analytic transit model (via the EXOFAST implementation in TAP) on a finer temporal scale and integrating it to the observed cadence, e.g. Kepler long-cadence data.

## When to use

Use when fitting long-integration light curves (e.g. Kepler long-cadence photometry with ~30-min exposures) where the finite exposure smears ingress/egress and would bias transit parameters like Rp/Rs, a/Rs, i, and Tmid.

## When NOT to use

- Short-cadence or ground-based exposures where integration smearing is negligible relative to transit duration, so finer-scale evaluation plus integration adds cost without accuracy benefit
- Data that have already been phase folded and rebinned — TAP is explicitly designed for simultaneous analysis of multiple light curves 'with no need to phase fold and rebin', so folding destroys the per-curve cadence and noise information this skill relies on

## Inputs

- Long-integration photometric time series, e.g. long-cadence Kepler light curves retrieved from the MAST archive, normalized per transit segment without phase folding or rebinning
- Initial transit parameter guesses (P, a/Rs, Rp/Rs, i, Tmid) and per-curve limb-darkening, wavelet noise, and trend parameters for the TAP parameter-locking matrix

## Outputs

- Cadence-integrated Mandel and Agol (2002) model fluxes matched to each observed Kepler exposure
- Per-transit Tmid values and combined (locked) system parameters with posterior uncertainties from the Carter and Winn (2009) wavelet-likelihood MCMC
- Comparison table of multicurve versus single-curve parameter uncertainties (e.g. written to multitransit_results.csv)

## How to apply

In TAP, enable the long-integration option so the Mandel and Agol (2002) analytic transit model, as implemented in EXOFAST, is evaluated on a finer temporal scale than the observations and then integrated to the observed Kepler cadence. Work directly on each transit segment without phase folding or rebinning, so per-curve conditions (limb darkening, noise, trend) are preserved. During the MCMC, run this cadence-integrated model inside the Carter and Winn (2009) wavelet-based likelihood, which parameterizes both uncorrelated (white) and correlated (red) noise rather than a classic chi-squared. Keep system parameters (P, a/Rs, Rp/Rs, i, Tmid) locked across curves while limb-darkening, wavelet noise, and trend parameters remain free per curve. The rationale is that a sharp model evaluated at cadence resolution underestimates transit duration distortions; integrating the finer-scale model reproduces the time-averaged flux actually recorded by the detector.

## Related tools

- **Transit Analysis Package (TAP)** (Provides the long-integration setting that computes the transit model on a finer temporal scale and integrates it to the observed cadence, driven through the IDL GUI and MCMC machinery) — https://github.com/zgazak/TAP
- **EXOFAST** (Improved implementation of the Mandel and Agol (2002) analytic transit model that TAP evaluates at sub-cadence time steps before integration)
- **IDL (Interactive Data Language)** (Runtime environment for the TAP GUI and MCMC pipeline that performs the cadence-integrated model fitting)
- **Kepler mission** (Source of the long-integration (long-cadence) photometry that motivates enabling the finer-scale integration handling)

## Evaluation signals

- The transit model is demonstrably evaluated at a finer temporal scale than the observations before comparison to the Kepler long-cadence fluxes, not sampled at cadence directly
- Residuals between the integrated model and the observed light curve are consistent with the Carter and Winn (2009) wavelet likelihood's white and red noise parameterization (posterior sigmaW/sigmaR well behaved), not the classic chi-squared
- Each transit segment is fit un-folded and un-rebinned; the TAP parameter matrix shows P, a/Rs, Rp/Rs, i, and Tmid evolving together while limb-darkening and trend parameters remain free per curve
- Derived parameters (e.g. Rp/Rs) for Kepler targets agree with published reanalyses such as Kepler 4b-8b (Kipping and Bakos 2010), as the paper reports for its validation runs
- Multicurve (locked-parameter) fit uncertainties for the combined system parameters are no larger than those from independent single-curve refits

## Limitations

- The correction only matters for long-integration observations (the paper cites most Kepler light curves as the case in point); applying it to well-sampled, short exposures yields no benefit
- Accuracy depends on the finer temporal scale being fine enough to resolve ingress/egress before integration; an insufficiently fine grid reintroduces smearing bias
- Derived transit parameters vary between fitting methods, and confidence in them depends critically on the treatment of noise, limb darkening, and cadence, so results are only comparable under a consistent configuration

## Evidence

- [abstract] MCMC with Mandel and Agol model in TAP: "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [intro] Long-integration handling on finer temporal scale: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
- [intro] No phase folding or rebinning across multiple light curves: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
