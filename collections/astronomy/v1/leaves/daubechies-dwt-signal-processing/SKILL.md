---
name: daubechies-dwt-signal-processing
description: Use when fitting transit light curves (e.g., Kepler photometry) whose
  residuals show correlated (red) noise, and you need MCMC parameter uncertainties
  that marginalize over free σw and σr instead of fixed χ² error bars.
when_to_use_negative:
- The analysis target is not photometric exoplanet transit light-curve fitting (TAP
  is purpose-built for transit photometry of ground-based and Kepler data)
- The time series cannot be treated as evenly sampled per cadence, or you have not
  accounted for long-integration effects by computing models on a finer temporal scale
  before comparing to the observed cadence
- You require point estimates only with no need for correlated-noise-aware uncertainty
  quantification (e.g., a quick χ² reduction where red noise is demonstrably negligible)
edam_operation: http://edamontology.org/operation_2239
tools:
- name: Transit Analysis Package (TAP)
  role: Hosts the Metropolis-Hastings MCMC with the Carter and Winn (2009) wavelet-based
    likelihood that parameterizes uncorrelated and correlated photometric noise for
    transit fitting
  repo: zgazak/TAP
- name: IDL
  role: Implementation language of TAP's graphical-user-interface-driven transit analysis
    pipeline
- name: EXOFAST
  role: Improved implementation of the Mandel and Agol (2002) analytic transit model
    used inside TAP's MCMC fitting
- name: Kepler mission
  role: Source of high-precision (few parts in 10^5) photometric light curves on which
    the DWT likelihood is applied
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:46df5227b4e485059b8980ec1aba75570c429d9a80a06b52f68029e7ae0f002e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Kepler mission
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# daubechies-dwt-signal-processing

## Summary

Wavelet-based likelihood using the Daubechies-4 discrete wavelet transform (Carter and Winn 2009) that parameterizes uncorrelated white noise (σw) and 1/f correlated red noise (σr) for exoplanet transit light-curve MCMC fitting, replacing the classic χ² assumption of noiseless correlation.

## When to use

Use when fitting transit light curves (e.g., Kepler photometry) whose residuals show correlated (red) noise, and you need MCMC parameter uncertainties that marginalize over free σw and σr instead of fixed χ² error bars.

## When NOT to use

- The analysis target is not photometric exoplanet transit light-curve fitting (TAP is purpose-built for transit photometry of ground-based and Kepler data)
- The time series cannot be treated as evenly sampled per cadence, or you have not accounted for long-integration effects by computing models on a finer temporal scale before comparing to the observed cadence
- You require point estimates only with no need for correlated-noise-aware uncertainty quantification (e.g., a quick χ² reduction where red noise is demonstrably negligible)

## Inputs

- Normalized photometric transit light curve (e.g., Kepler mission time series retrieved from MAST), one or more transits without phase folding/rebinning
- Transit model parameters (Mandel and Agol 2002 analytic model) with priors/initial guesses for the MCMC

## Outputs

- Per-transit posterior summaries of white-noise scatter σw and red-noise scatter σr (e.g., wavelet_noise_fit.csv with both noise fields)
- MCMC posterior distributions of transit parameters with robust (correlated-noise-aware) uncertainties

## How to apply

Retrieve and normalize one transit light curve (e.g., from the Kepler mission via MAST) into an evenly sampled photometric time series. Implement the Daubechies 4th-order DWT likelihood following Carter and Winn (2009) Eqs. (1)-(3), holding γ = 1 and g(1) = (2 ln 2)^-1, with σw and σr as free parameters. Wrap the likelihood in the Metropolis-Hastings MCMC used by TAP (built on the Mandel and Agol 2002 analytic model via EXOFAST) and use Gibbs sampling to update the noise parameters, constraining σr ≥ 0. Run the chain per transit and record posterior summaries of σw and σr. For long-integration data such as Kepler light curves, calculate transit models on a finer temporal scale and integrate to the observed cadence. Write the fitted (σw, σr) pair to an output CSV (e.g., wavelet_noise_fit.csv) per transit.

## Related tools

- **Transit Analysis Package (TAP)** (Hosts the Metropolis-Hastings MCMC with the Carter and Winn (2009) wavelet-based likelihood that parameterizes uncorrelated and correlated photometric noise for transit fitting) — zgazak/TAP
- **IDL** (Implementation language of TAP's graphical-user-interface-driven transit analysis pipeline)
- **EXOFAST** (Improved implementation of the Mandel and Agol (2002) analytic transit model used inside TAP's MCMC fitting)
- **Kepler mission** (Source of high-precision (few parts in 10^5) photometric light curves on which the DWT likelihood is applied)

## Evaluation signals

- The DWT likelihood module executes end-to-end on the normalized light curve
- Output (e.g., wavelet_noise_fit.csv) contains both σw and σr fields for each fitted transit
- Posterior σw agrees with the light curve's point-to-point scatter within roughly a factor of two
- Posterior σr is non-negative (enforced constraint of the Gibbs step)
- Posterior uncertainty intervals on transit parameters are consistent with (typically wider than) classic χ² errors when red noise is present

## Limitations

- σr is constrained to be non-negative during Gibbs sampling, so negative red-noise trials cannot be represented
- Long integration observations (most Kepler light curves) distort transit shapes unless models are calculated on a finer temporal scale and integrated to the observed cadence
- Confidence in derived transit parameters depends critically on noise treatment, limb darkening, and cadence; different fitting methods can yield varying derived quantities
- TAP analyzes multiple light curves of varying cadence, filter (limb darkening), and noise conditions simultaneously, so per-transit noise parameters must be tracked separately per condition

## Evidence

- [intro] TAP uses the wavelet-based likelihood of Carter and Winn (2009) to marginalize over uncorrelated and correlated photometric noise: [evidence span withheld — anchor status: not_found]
- [abstract] TAP fits light curves via MCMC with the Mandel and Agol analytic model: "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
