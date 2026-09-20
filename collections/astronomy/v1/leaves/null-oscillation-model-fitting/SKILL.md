---
name: null-oscillation-model-fitting
description: Use when analyzing binned count/flux spectra from a Fermi-LAT source
  for axion-like particle (ALP) effects, and you need an irregularity statistic (residual
  scatter) rather than a direct ALP-model spectral fit.
when_to_use_negative:
- When the analysis goal is to fit the observed spectrum directly with ALP models
  (the paper explicitly avoids this in favor of the irregularity-based method).
- When spectra are not binned count/flux spectra and no null-oscillation model can
  be fit to define residuals.
- When a conversion-probability calculation is needed (use the alpconv tool for calculating
  ALP-photon conversion instead).
tools:
- name: Fermi-LAT
  role: Source instrument supplying the binned NGC 1275 spectra that the null-oscillation
    model is fit to
- name: alpconv
  role: Companion code for calculating ALP-photon conversion in the broader re-analysis;
    no usage instructions provided (Chinese comments remain in the code)
  repo: https://github.com/lyf222/alpconv
- name: scipy.optimize
  role: Least-squares/minimization routine used to fit the best-fit null-oscillation
    model to each spectrum
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:1b97de9e348208b0ec26c854b1ff4fa434b218dd90c80fb9d94fc75b47ada716
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lyf222/alpconv
  tools:
  - Fermi-LAT
  - alpconv
  - scipy.optimize
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# null-oscillation-model-fitting

## Summary

Quantifies spectral irregularity in Fermi-LAT gamma-ray spectra (e.g. NGC 1275) by fitting a conventional null-oscillation model with no ALP-induced oscillation and using the scatter of fit residuals as the irregularity statistic, instead of fitting the observed spectrum directly with ALP models.

## When to use

Use when analyzing binned count/flux spectra from a Fermi-LAT source for axion-like particle (ALP) effects, and you need an irregularity statistic (residual scatter) rather than a direct ALP-model spectral fit.

## When NOT to use

- When the analysis goal is to fit the observed spectrum directly with ALP models (the paper explicitly avoids this in favor of the irregularity-based method).
- When spectra are not binned count/flux spectra and no null-oscillation model can be fit to define residuals.
- When a conversion-probability calculation is needed (use the alpconv tool for calculating ALP-photon conversion instead).

## Inputs

- Binned NGC 1275 count/flux spectra from the Fermi-LAT observation (one spectrum per observation/analysis configuration)
- Best-fit null-oscillation (conventional, ALP-free) spectrum model parameters

## Outputs

- irregularity_table.csv — one irregularity value (scatter of fit residuals, e.g. standard deviation/RMSD) per input spectrum

## How to apply

Load the binned NGC 1275 count/flux spectra obtained from the Fermi-LAT observation, then fit the best-fit null-oscillation model — the conventional spectrum model with no ALP-induced oscillation — to each spectrum using a least-squares/minimization routine such as scipy.optimize. Compute residuals between the observed flux points and the fitted null-oscillation model in each energy bin. Quantify the irregularity of each spectrum as the scatter of these residuals (e.g. their standard deviation / root-mean-square deviation). Assemble one irregularity value per input spectrum into a single table written to irregularity_table.csv; the absence of irregularity signal beyond the null expectation is consistent with the paper's reported 'no spectral oscillation for the NGC 1275'.

## Related tools

- **Fermi-LAT** (Source instrument supplying the binned NGC 1275 spectra that the null-oscillation model is fit to)
- **alpconv** (Companion code for calculating ALP-photon conversion in the broader re-analysis; no usage instructions provided (Chinese comments remain in the code)) — https://github.com/lyf222/alpconv
- **scipy.optimize** (Least-squares/minimization routine used to fit the best-fit null-oscillation model to each spectrum)

## Examples

```
from scipy.optimize import least_squares; import numpy as np; fit = least_squares(lambda p: obs_flux - null_model(p, energy), p0); irregularity = np.std(obs_flux - null_model(fit.x, energy))
```

## Evaluation signals

- irregularity_table.csv exists and is a valid CSV with exactly one irregularity value per input spectrum
- All irregularity values are finite (no NaN/inf residuals from a failed fit)
- Residuals are computed against the best-fit null-oscillation model, not an ALP-model fit
- No irregularity signal beyond the null expectation is claimed, consistent with the paper's reported absence of spectral oscillation for NGC 1275

## Limitations

- The alpconv repository provides no usage instructions; Chinese character comments remain in the code.
- The method quantifies irregularity via residual scatter of the null model rather than a direct ALP-model fit, so it cannot by itself yield per-model likelihoods; coupling exclusion (e.g. gaγ > 3×10⁻¹² GeV⁻¹ at ma ∼ 1 neV, 95% CL) comes from the broader analysis.
- No discussion-section text was available in the indexed document to further contextualize method caveats.

## Evidence

- [abstract] Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum.: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] With the newly used method, we find no spectral oscillation for the NGC 1275: "With the newly used method, we find no spectral oscillation for the NGC 1275"
- [methods] The degree of irregularity of a spectrum is estimated as the scatter of the fit residuals (of the best-fit null-oscillation model): [evidence span withheld — anchor status: not_found]
- [readme] at present no intruction for use provides, chinese character comments remain in the code.: "(at present no intruction for use provides, chinese character comments remain in the code.)"
