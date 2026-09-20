---
name: gamma-ray-spectral-fitting
description: Use when re-analyzing binned Fermi-LAT gamma-ray spectra of a blazar
  or AGN (e.g. NGC 1275) to search for ALP-induced spectral oscillations via an irregularity
  statistic rather than ALP model fitting.
when_to_use_negative:
- When the goal is to directly fit the observed spectrum with ALP models to obtain
  conversion-probability likelihoods — the paper explicitly adopts the irregularity
  method instead of that approach
- When the ALP-photon conversion probability itself must be computed (use the alpconv
  tool for that calculation, not this residual-scatter skill)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Fermi-LAT
  role: Source instrument providing the binned NGC 1275 gamma-ray spectra that are
    fit and analyzed for irregularity
- name: alpconv
  role: Companion code for calculating ALP-photon conversion, cited as the detailed
    reference for the ALP analysis context
  repo: github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:5241e2c4209cc98afa2fa7498d7e3f21da754334fdb291a399860b9114517c0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# gamma-ray-spectral-fitting

## Summary

Method for quantifying the irregularity of a gamma-ray spectrum (e.g. the Fermi-LAT observation of NGC 1275) by fitting a null-oscillation model and measuring the scatter of fit residuals, instead of directly fitting the spectrum with axion-like particle (ALP) models.

## When to use

Use when re-analyzing binned Fermi-LAT gamma-ray spectra of a blazar or AGN (e.g. NGC 1275) to search for ALP-induced spectral oscillations via an irregularity statistic rather than ALP model fitting.

## When NOT to use

- When the goal is to directly fit the observed spectrum with ALP models to obtain conversion-probability likelihoods — the paper explicitly adopts the irregularity method instead of that approach
- When the ALP-photon conversion probability itself must be computed (use the alpconv tool for that calculation, not this residual-scatter skill)

## Inputs

- Binned NGC 1275 count/flux spectra from the Fermi-LAT observation of the source (one spectrum per time/energy binning choice)
- Best-fit null-oscillation (conventional, no-ALP) spectrum model parameters

## Outputs

- irregularity_table.csv with one irregularity value (residual scatter) per input spectrum
- Per-bin residuals between observed flux points and the fitted null-oscillation model

## How to apply

Load the binned NGC 1275 count/flux spectra from the Fermi-LAT observation, then fit the best-fit null-oscillation model (the conventional spectrum model with no ALP-induced oscillation) to each spectrum using a least-squares minimization routine such as scipy.optimize. Compute the residuals between the observed flux points and the fitted null-oscillation model in each energy bin. Quantify the irregularity of each spectrum as the scatter of these residuals (e.g. their standard deviation or root-mean-square deviation), following the irregularity-based approach adopted instead of fitting the observed spectrum with ALP models. Assemble one irregularity value per input spectrum into a single table (irregularity_table.csv) and interpret absence of excess irregularity as 'no spectral oscillation', consistent with the paper's null result for NGC 1275.

## Related tools

- **Fermi-LAT** (Source instrument providing the binned NGC 1275 gamma-ray spectra that are fit and analyzed for irregularity)
- **alpconv** (Companion code for calculating ALP-photon conversion, cited as the detailed reference for the ALP analysis context) — github.com/lyf222/alpconv

## Examples

```
from scipy.optimize import least_squares; fit = least_squares(null_model_resid, p0, args=(E, flux)); resid = flux - null_model(E, fit.x); irregularity = resid.std()
```

## Evaluation signals

- irregularity_table.csv exists and parses as a valid CSV with exactly one irregularity value per input spectrum
- All irregularity values in the table are finite (no NaN/inf)
- Residuals are computed against the best-fit null-oscillation model, not an ALP model, verified by inspecting the fitted model used per spectrum
- No irregularity signal beyond the null expectation is claimed, consistent with the paper's reported absence of spectral oscillation for NGC 1275
- scipy.optimize (least-squares/minimization) is used to obtain the null-oscillation best fit before residuals are taken

## Limitations

- No spectral oscillation was found for NGC 1275 with this method, so the irregularity statistic is a null-result test rather than a detection pipeline
- The companion alpconv repository provides no usage instructions and Chinese-character comments remain in the code, complicating reproduction of the full ALP conversion pipeline
- Constraints are strongest only around ALP mass ma ~ 1 neV (ruling out gaγ > 3×10⁻¹² GeV⁻¹ at 95% CL); broader exclusion regions require combining additional sources such as PKS 2155-304

## Evidence

- [abstract] Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum.: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] With the newly used method, we find no spectral oscillation for the NGC 1275: "With the newly used method, we find no spectral oscillation for the NGC 1275"
- [other] The degree of irregularity of a spectrum is estimated as the scatter of the fit residuals (of the best-fit null-oscillation model): [evidence span withheld — anchor status: not_found]
- [readme] at present no intruction for use provides, chinese character comments remain in the code.: "(at present no intruction for use provides, chinese character comments remain in the code.)"
