---
name: wavelet-likelihood-implementation
description: Use when fitting transit light curves whose parameter uncertainties would
  be underestimated by a classic χ² likelihood because photometric noise has a correlated
  (red) component, and you want σw and σr freed in the MCMC.
when_to_use_negative:
- The photometric noise is known to be purely uncorrelated Gaussian white noise, in
  which case the classic χ² likelihood's no-correlated-noise assumption is already
  satisfied
- The input has been phase folded and rebinned beforehand — TAP is designed for simultaneous
  analysis of multiple light curves of varying cadence without phase folding and rebinning,
  so pre-rebinning defeats its cadence/noise handling
- The task is a non-transit photometric fit where the 1/f red-noise parameterization
  does not model the actual correlated noise structure of the instrument or atmosphere
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Transit Analysis Package (TAP)
  role: Runs the MCMC that evaluates the wavelet-based likelihood against the Mandel
    and Agol analytic transit model; source code is the zgazak/TAP repository
  repo: zgazak/TAP
- name: IDL
  role: Language in which the TAP graphical user interface and the wavelet-likelihood
    MCMC are implemented
- name: Kepler mission
  role: Provides the high-precision photometric light curves (down to a few parts
    in 10^5) on which the wavelet-likelihood fit is exercised
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:41447b8170dc7da6040737db0e22b8a0b8e9b0b419b36ba296cf24dc99f444ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - IDL
  - Kepler mission
  - Transit Analysis Package (TAP)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# wavelet-likelihood-implementation

## Summary

Implements the Carter and Winn (2009) Daubechies-4 wavelet-based likelihood inside TAP's MCMC, replacing the classic chi-squared by letting uncorrelated white noise (σw) and 1/f correlated red noise (σr) evolve as free parameters during transit light-curve fitting.

## When to use

Use when fitting transit light curves whose parameter uncertainties would be underestimated by a classic χ² likelihood because photometric noise has a correlated (red) component, and you want σw and σr freed in the MCMC.

## When NOT to use

- The photometric noise is known to be purely uncorrelated Gaussian white noise, in which case the classic χ² likelihood's no-correlated-noise assumption is already satisfied
- The input has been phase folded and rebinned beforehand — TAP is designed for simultaneous analysis of multiple light curves of varying cadence without phase folding and rebinning, so pre-rebinning defeats its cadence/noise handling
- The task is a non-transit photometric fit where the 1/f red-noise parameterization does not model the actual correlated noise structure of the instrument or atmosphere

## Inputs

- Normalized photometric transit light curve (e.g., a single Kepler transit retrieved from MAST)
- Initial transit model parameters for the Mandel and Agol (2002) analytic model
- Starting guesses for σw (white noise scatter) and σr (red noise amplitude), with σr >= 0

## Outputs

- MCMC posterior samples including free σw and σr per transit
- Posterior summaries of σw and σr (e.g., written to wavelet_noise_fit.csv)
- Transit parameter estimates with wavelet-likelihood uncertainties, robust to correlated noise

## How to apply

Retrieve and normalize a transit light curve (e.g., a Kepler light curve from MAST), then fit it with TAP's MCMC, which uses the Mandel and Agol (2002) analytic transit model but replaces χ² with the wavelet-based likelihood of Carter and Winn (2009) per Eqs. (1)-(3). Hold the noise-model constants fixed at gamma = 1 and g(1) = (2 ln 2)^-1, and let two noise parameters — σw (uncorrelated Gaussian white noise) and σr (1/f correlated red noise) — evolve as free parameters during the chain. Update the noise parameters via Gibbs sampling within the Metropolis-Hastings MCMC, constraining σr to be non-negative. Because Kepler integration distorts transit models, set TAP to compute models on a finer temporal scale and integrate to the observed cadence rather than phase folding or rebinning the light curves.

## Related tools

- **Transit Analysis Package (TAP)** (Runs the MCMC that evaluates the wavelet-based likelihood against the Mandel and Agol analytic transit model; source code is the zgazak/TAP repository) — zgazak/TAP
- **IDL** (Language in which the TAP graphical user interface and the wavelet-likelihood MCMC are implemented)
- **Kepler mission** (Provides the high-precision photometric light curves (down to a few parts in 10^5) on which the wavelet-likelihood fit is exercised)

## Evaluation signals

- The likelihood module executes and the output contains both noise fields (σw and σr)
- σw agrees with the light curve's point-to-point scatter within roughly a factor of two
- σr is constrained to be non-negative throughout the chain and its posterior stays >= 0
- Fitted transit parameters show good agreement with previously published values for the same target (as TAP does for TrES-1b, WASP-4b, WASP-10b, and Kepler 4b-8b)
- Parameter uncertainties are wider/more robust than those obtained with a classic χ² fit on the same data when red noise is present

## Limitations

- The red noise is modeled specifically as 1/f correlated noise; other correlated-noise structures are not parameterized
- σr is constrained to be non-negative, so negative red-noise power cannot be fit; agreement for σw is only validated to within roughly a factor of two
- For long-integration observations such as Kepler light curves, the transit model must be calculated on a finer temporal scale and integrated to the observed cadence, or the fit is biased

## Evidence

- [intro] Wavelet likelihood parameterizes white and red noise: [evidence span withheld — anchor status: not_found]
- [abstract] Robust uncertainty assessment vs chi-squared: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [abstract] MCMC with Mandel and Agol model: "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [intro] Long-integration handling without rebinning: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
