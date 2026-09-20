---
name: wavelet-likelihood-noise-parameterization
description: Use when fitting photometric transit light curves whose uncertainties
  may be underestimated by classic chi-squared — especially ground-based data with
  correlated (red) noise or Kepler long-cadence photometry.
when_to_use_negative:
- When a pure classic chi-squared treatment is explicitly required as a controlled
  comparison (the locked red-noise-at-zero run), not the full noise-parameterized
  fit
- When the analysis goal is only a quick model fit without credible uncertainty quantification
  on correlated noise — the wavelet likelihood adds parameters and computation cost
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Transit Analysis Package (TAP)
  role: Runs the MCMC with the Carter and Winn (2009) wavelet-based likelihood over
    the fitted light curve(s)
- name: IDL (Interactive Data Language)
  role: Language/environment in which the TAP GUI and wavelet-likelihood MCMC execute
- name: EXOFAST
  role: Provides the Mandel and Agol (2002) analytic transit model evaluated inside
    the wavelet-likelihood MCMC
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  - task_006
  - task_004
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:6d1bd85c77ee9ce9424565a163dd911463de5ec3f5addc886f36b07632f46ed1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EXOFAST
  - IDL
  - IDL (Interactive Data Language)
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# wavelet-likelihood-noise-parameterization

## Summary

Replace the classic chi-squared likelihood in a transit MCMC with the Carter and Winn (2009) wavelet-based likelihood, which adds explicit white (uncorrelated) and red (correlated) noise amplitudes as free parameters so posterior uncertainties on Period, Inclination, a/Rs, Rp/Rs, and Tmid are robustly quantified.

## When to use

Use when fitting photometric transit light curves whose uncertainties may be underestimated by classic chi-squared — especially ground-based data with correlated (red) noise or Kepler long-cadence photometry.

## When NOT to use

- When a pure classic chi-squared treatment is explicitly required as a controlled comparison (the locked red-noise-at-zero run), not the full noise-parameterized fit
- When the analysis goal is only a quick model fit without credible uncertainty quantification on correlated noise — the wavelet likelihood adds parameters and computation cost

## Inputs

- Ground-based or space-based photometric transit light curves (e.g. Johnson et al. 2009 WASP-10b; Winn et al. 2007 TrES-1b; Winn et al. 2009 WASP-4b; Kepler long-cadence 29.4-min photometry of Kepler-4b through -8b from MAST)
- Initial transit parameter guesses (Period, Inclination, a/Rs, Rp/Rs, Tmid) and noise amplitude starting values for the wavelet likelihood

## Outputs

- MCMC posterior distributions (median and 1-sigma) for transit and per-transit noise parameters, including the correlated/red noise amplitude
- Uncertainty estimates robust to red noise, suitable for fractional-difference comparison against published values (e.g. KB10 Table 5, Eq. 5)

## How to apply

Instead of assuming a classic chi-squared residual, run the TAP MCMC with the Carter and Winn (2009) wavelet-based likelihood so both uncorrelated and correlated photometric noise are parameterized and marginalized alongside the transit parameters. Fit the Mandel and Agol (2002) analytic model (via EXOFAST) over the light curve(s) while the wavelet noise amplitudes evolve in the chain. For a diagnostic, run the fit twice: once with the red-noise amplitude locked at zero (emulating a classic chi-squared analysis) and once with it free — as done for the Johnson et al. (2009) WASP-10b light curve. Inspect the fitted red-noise amplitude: a negligible value (as found for WASP-10b, consistent with Johnson et al.) means the light curve carries no significant correlated noise, while a non-zero value means chi-squared error bars were too small.

## Related tools

- **Transit Analysis Package (TAP)** (Runs the MCMC with the Carter and Winn (2009) wavelet-based likelihood over the fitted light curve(s))
- **IDL (Interactive Data Language)** (Language/environment in which the TAP GUI and wavelet-likelihood MCMC execute)
- **EXOFAST** (Provides the Mandel and Agol (2002) analytic transit model evaluated inside the wavelet-likelihood MCMC)

## Evaluation signals

- Ablation check: fit with red-noise amplitude locked at zero and with it free; if the free fitted red-noise parameter is negligible (as for WASP-10b), the two runs should agree with published values within 1-sigma (e.g. inclination 88.44 vs 88.50 deg, a/Rs 11.60 vs 11.64, Rp/Rs 0.1593 vs 0.1588)
- Posterior uncertainty sanity: parameter uncertainties from the wavelet run should be equal to or larger than classic chi-squared errors when red noise is present
- Agreement validation: fitted parameters must match the corresponding published values (e.g. TrES-1b inclination 89.01 +/- 0.61 deg, Rp/Rs 0.1375 +/- 0.0012) within quoted 1-sigma
- Fractional-difference criterion: |FD| < 1 (Eq. 5) versus KB10 values for Kepler-4b through -7b system parameters

## Limitations

- Different fitting methods and noise treatment lead to variations in derived quantities; confidence quantification depends critically on the noise treatment, stellar limb darkening, and cadence
- The Kepler-8b period showed a significant difference versus KB10 even under this likelihood — agreement is not guaranteed for every parameter
- The wavelet likelihood is implemented in TAP/IDL; applying it requires the Carter and Winn (2009) formalism rather than off-the-shelf chi-squared minimizers

## Evidence

- [intro] wavelet likelihood replaces chi-squared: [evidence span withheld — anchor status: not_found]
- [abstract] robust uncertainty assessment: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing"
- [intro] noise treatment drives confidence: "quantification of confidence in derived parameters depends critically on the treatment of noise, stellar limb darkening,"
