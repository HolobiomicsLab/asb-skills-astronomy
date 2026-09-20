---
name: mcmc-posterior-parameter-estimation
description: Use when reanalyzing Kepler long-cadence (29.4 min) transit photometry
  to derive Period, Inclination, a/Rs, and Rp/Rs posteriors via MCMC, or when comparing
  independent transit fits with a fractional-difference significance test.
when_to_use_negative:
- When the light curves are ground-based with different cadence/noise conditions that
  require simultaneous multi-transit analysis rather than a single locked-orbit fit
- When eccentric orbits or fixed limb-darkening priors are scientifically required,
  since this workflow locks circular orbits and leaves limb darkening free
- When the goal is discovery of new transits rather than refinement of known system
  parameters, since this skill presumes known planets and periods
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3315
tools:
- name: Transit Analysis Package (TAP)
  role: Runs the MCMC chains that fit light curves using the Mandel & Agol (2002)
    analytic model with the wavelet-based likelihood
  repo: https://github.com/zgazak/TAP
- name: IDL
  role: Language environment in which TAP's graphical user interface and analysis
    pipeline are executed
- name: EXOFAST
  role: Provides the improved implementation of the Mandel & Agol (2002) analytic
    transit model used inside TAP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:0c805dbce3a52dc2176c9dd76ed6fade6fb83698f3546f81967547e9adeafd76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mcmc-posterior-parameter-estimation

## Summary

Estimate exoplanet transit system parameters (Period, Inclination, a/Rs, Rp/Rs) by running MCMC chains over the Mandel & Agol (2002) analytic transit model in TAP, using the Carter & Winn (2009) wavelet-based likelihood to capture white and red photometric noise. Validated by comparing posterior medians to published values within 1-sigma and computing Eq. (5) fractional differences (|FD| < 1 indicates agreement).

## When to use

Use when reanalyzing Kepler long-cadence (29.4 min) transit photometry to derive Period, Inclination, a/Rs, and Rp/Rs posteriors via MCMC, or when comparing independent transit fits with a fractional-difference significance test.

## When NOT to use

- When the light curves are ground-based with different cadence/noise conditions that require simultaneous multi-transit analysis rather than a single locked-orbit fit
- When eccentric orbits or fixed limb-darkening priors are scientifically required, since this workflow locks circular orbits and leaves limb darkening free
- When the goal is discovery of new transits rather than refinement of known system parameters, since this skill presumes known planets and periods

## Inputs

- Kepler long-cadence photometry (29.4-minute cadence) from the MAST archive for Kepler-4b through Kepler-8b
- TAP configuration: circular orbits locked, limb-darkening parameters free, Carter & Winn (2009) wavelet likelihood enabled
- Published reference parameter values (e.g. Kipping and Bakos 2010 Table 5) for fractional-difference comparison

## Outputs

- Per-planet posterior summaries (median and 1-sigma) of Period, Inclination, a/Rs, and Rp/Rs in tap_kepler_4_8_parameters.csv
- Fractional-difference (FD) statistics per parameter against published values, with |FD| < 1 flagging statistical agreement
- Agreement results: res_kepler_4_7_agreement (Kepler-4b to 7b agree) and res_kepler8_discrepancy (Kepler-8b period difference significant)

## How to apply

Retrieve Kepler long-cadence photometry (29.4-minute cadence) for the target planets from MAST and configure TAP in IDL with circular orbits locked and limb-darkening parameters left free. Enable the wavelet-based likelihood of Carter & Winn (2009) so the MCMC parameterizes both uncorrelated (white) and correlated (red) noise instead of classic chi-squared, which yields more accurate parameter uncertainties. Because long integration smears transits, compute the Mandel & Agol (2002) analytic model (via EXOFAST) on a 10x finer temporal scale and rebin it to the observed 29.4-minute cadence. Run the MCMC chains to sample posteriors of Period, Inclination, a/Rs, and Rp/Rs, and assemble per-planet posterior summaries (median and 1-sigma). Validate by checking each fitted value against published table values within 1-sigma and computing the Eq. (5) fractional difference versus published values, confirming |FD| < 1 for statistical agreement.

## Related tools

- **Transit Analysis Package (TAP)** (Runs the MCMC chains that fit light curves using the Mandel & Agol (2002) analytic model with the wavelet-based likelihood) — https://github.com/zgazak/TAP
- **IDL** (Language environment in which TAP's graphical user interface and analysis pipeline are executed)
- **EXOFAST** (Provides the improved implementation of the Mandel & Agol (2002) analytic transit model used inside TAP)

## Examples

```
idl> tap_fit --photometry kepler_4b_8b_lc.fits --orbit circular --limb_darkening free --wavelet_likelihood carter_winn2009 --finer_cadence 10x --output tap_kepler_4_8_parameters.csv
```

## Evaluation signals

- Each fitted posterior median matches the corresponding TAP column of Table 5 within the quoted 1-sigma uncertainty
- Eq. (5) fractional difference computed against Kipping and Bakos (2010) published values satisfies |FD| < 1 for Kepler-4b through Kepler-7b
- Kepler-8b period difference is flagged as significant (res_kepler8_discrepancy), reproducing the expected discrepancy rather than forcing agreement
- Posterior summaries are emitted as medians with 1-sigma widths in tap_kepler_4_8_parameters.csv with columns for Period, Inclination, a/Rs, and Rp/Rs
- Transit models computed on a 10x finer temporal scale and rebinned to 29.4-minute cadence show no residual long-integration smearing in the fit residuals

## Limitations

- Long-cadence Kepler integration smears transit shape, requiring model computation on a finer temporal scale with rebinning; omitting this biases fitted parameters
- Circular orbits are locked, so eccentricity-driven parameter degeneracies are not explored in this workflow
- Fitting-method sensitivity: different fitting methods can lead to variations in derived quantities, and confidence depends critically on noise treatment, limb darkening, and cadence
- Kepler-8b period shows a significant fractional difference versus the comparison method, indicating not all parameters reproduce across methods

## Evidence

- [abstract] MCMC fitting with wavelet likelihood: "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [abstract] Wavelet likelihood for robust uncertainties: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [intro] Long-integration rebinning: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
- [abstract] Good agreement with prior publications: [evidence span withheld — anchor status: not_found]
- [intro] Method sensitivity caveat: [evidence span withheld — anchor status: not_found]
