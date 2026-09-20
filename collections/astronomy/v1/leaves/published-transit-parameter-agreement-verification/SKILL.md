---
name: published-transit-parameter-agreement-verification
description: Use when ground-based or Kepler transit photometry from a published study
  is available and the goal is to verify that an independent pipeline (e.g. TAP) recovers
  the published inclination, a/Rs, Rp/Rs, and per-transit Tmid values within 1-sigma.
when_to_use_negative:
- The data are new observations whose parameters have no published counterpart to
  compare against — this skill is a reproduction/verification workflow, not a first-pass
  parameter derivation
- The light curves must be phase folded and rebinned into a single averaged transit
  before fitting — TAP is designed to fit multiple transits simultaneously without
  phase folding
- Orbital eccentricity is significant and cannot be locked at zero, since the verification
  fits fix eccentricity and periastron argument at zero
tools:
- name: Transit Analysis Package (TAP)
  role: IDL GUI that runs the MCMC light-curve fits using the Mandel and Agol (2002)
    analytic model and the Carter and Winn (2009) wavelet likelihood
  repo: https://github.com/zgazak/TAP
- name: IDL
  role: Runtime environment hosting the TAP graphical user interface used to configure
    and run the fits
- name: EXOFAST
  role: Provides the analytic transit model implementation used inside TAP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:ccfbf00cd75641dd5fc1d7d756c6ce7b0e41bfee97e77dea478fa57ad2939051
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Transit Analysis Package (TAP)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# published-transit-parameter-agreement-verification

## Summary

Reanalysis of published exoplanet transit light curves with TAP (an IDL GUI running MCMC fits with the Mandel and Agol 2002 model and Carter and Winn 2009 wavelet likelihood) to check whether a shared parameter set reproduces previously published system parameters such as inclination, a/Rs, and Rp/Rs within their quoted 1-sigma uncertainties.

## When to use

Use when ground-based or Kepler transit photometry from a published study is available and the goal is to verify that an independent pipeline (e.g. TAP) recovers the published inclination, a/Rs, Rp/Rs, and per-transit Tmid values within 1-sigma.

## When NOT to use

- The data are new observations whose parameters have no published counterpart to compare against — this skill is a reproduction/verification workflow, not a first-pass parameter derivation
- The light curves must be phase folded and rebinned into a single averaged transit before fitting — TAP is designed to fit multiple transits simultaneously without phase folding
- Orbital eccentricity is significant and cannot be locked at zero, since the verification fits fix eccentricity and periastron argument at zero

## Inputs

- Published photometric transit light curves (time series, one per transit; e.g. the three TrES-1b curves from Winn et al. 2007)
- Published system parameter values with 1-sigma uncertainties to compare against (inclination, a/Rs, Rp/Rs, per-transit Tmid)
- Limb-darkening / filter and cadence information for each light curve (transits may differ in instrument, filter, weather)

## Outputs

- Posterior medians and uncertainties for inclination, a/Rs, Rp/Rs, and per-transit Tmid from the MCMC fit
- A comparison table of reproduced vs. published parameters (e.g. a Table 2-style table written to res_tres1b_wasp4b_agreement.csv)
- A within-1-sigma agreement verdict for each parameter

## How to apply

Load the published per-transit photometric time series (e.g. the three TrES-1b light curves of Winn et al. 2007) into TAP, the IDL GUI package, and fit them simultaneously with the Mandel and Agol (2002) analytic transit model (implemented via EXOFAST). Fix eccentricity and argument of periastron at zero and share a single parameter set across all transits, leaving only Tmid free per transit. Run the TAP MCMC with the Carter and Winn (2009) wavelet-based likelihood, which parameterizes both uncorrelated (white) and correlated (red) noise for more robust uncertainties than classic chi-squared, without phase folding or rebinning the light curves. Extract posterior medians and uncertainties for inclination, a/Rs, Rp/Rs, and each Tmid, assemble them into a comparison table against the published values, and validate agreement within the quoted 1-sigma ranges (e.g. TrES-1b inclination 89.01 +/- 0.61 deg, a/Rs 10.46 +0.13/-0.23, Rp/Rs 0.1375 +/- 0.0012).

## Related tools

- **Transit Analysis Package (TAP)** (IDL GUI that runs the MCMC light-curve fits using the Mandel and Agol (2002) analytic model and the Carter and Winn (2009) wavelet likelihood) — https://github.com/zgazak/TAP
- **IDL** (Runtime environment hosting the TAP graphical user interface used to configure and run the fits)
- **EXOFAST** (Provides the analytic transit model implementation used inside TAP)

## Evaluation signals

- Reproduced inclination, a/Rs, and Rp/Rs agree with the published values (e.g. Winn et al. 2007 TrES-1b: inclination 89.01 +/- 0.61 deg, a/Rs 10.46 +0.13/-0.23, Rp/Rs 0.1375 +/- 0.0012) within the quoted 1-sigma uncertainties
- Each per-transit Tmid (targets ~0.84298/0.87341/0.90371) matches the published mid-transit times within 1-sigma
- Eccentricity and argument of periastron were locked at zero and all system parameters except per-transit Tmid shared one set across all transits, as configured
- The MCMC used the wavelet-based likelihood (white + red noise parameterization) rather than classic chi-squared, so reported uncertainties reflect correlated noise
- The comparison is recorded in a Table 2-style output (e.g. res_tres1b_wasp4b_agreement.csv) listing reproduced values against published ones

## Limitations

- Agreement was demonstrated for specific cases (TrES-1b, WASP-4b, WASP-10b, Kepler 4b-8b); different fitting methods can lead to variations in derived quantities, so agreement is not guaranteed for arbitrary systems
- Confidence in derived parameters depends critically on the treatment of noise, stellar limb darkening, and cadence
- Long-integration observations (e.g. Kepler) require the transit model to be computed on a finer temporal scale and integrated to the observed cadence; ignoring this biases parameters

## Evidence

- [abstract] Reanalysis of ground-based observations of TrES-1b, WASP-4b, and WASP-10b and space-based Kepler 4b-8b show good agreement between TAP and prior publications.: [evidence span withheld — anchor status: not_found]
- [abstract] MCMC fitting with the Mandel and Agol (2002) analytic model.: "uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)"
- [intro] Wavelet-based likelihood handles white and red noise for accurate uncertainties.: [evidence span withheld — anchor status: not_found]
- [abstract] TAP fits multiple transits simultaneously without phase folding or rebinning.: "The software is able to simultaneously analyze multiple transits observed in different conditions (instrument, filter, weather, etc.)."
- [intro] EXOFAST supplies the analytic transit model inside TAP.: "The TAP software package uses EXOFAST [23], an improved implementation of the [15] analytic transit model"
