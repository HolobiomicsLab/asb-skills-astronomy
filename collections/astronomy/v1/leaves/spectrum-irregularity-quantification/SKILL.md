---
name: spectrum-irregularity-quantification
description: Use when you have binned count/flux spectra (e.g. from Fermi-LAT) and
  need to test for ALP-induced spectral irregularity without directly fitting ALP
  models to the observed spectrum.
when_to_use_negative:
- You intend to directly fit ALP models to the observed spectrum to obtain conversion-probability
  constraints — the article's method explicitly replaces this with an irregularity
  calculation
- The input is not a binned spectrum (e.g. unbinned event lists) or you lack a fitted
  null-oscillation model to compute residuals against
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: Fermi-LAT
  role: Provides the binned gamma-ray count/flux spectra of NGC 1275 whose residual
    scatter quantifies spectral irregularity
- name: scipy.optimize
  role: Least-squares/minimization routine used to fit the best-fit null-oscillation
    model to each spectrum before residual computation
- name: alpconv
  role: Companion code for calculating ALP-photon conversion in the broader analysis;
    the null-oscillation fit itself uses no ALP conversion
  repo: github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:2eca8e97857ba3cc6f287efd62aa56bb193134ad0b335325759afe914c9417b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  - scipy.optimize
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spectrum-irregularity-quantification

## Summary

Quantifies the irregularity of an observed gamma-ray spectrum as the scatter (standard deviation / RMS) of fit residuals from the best-fit null-oscillation model, rather than directly fitting ALP models. Used to test for ALP-induced spectral oscillation in the Fermi-LAT observation of NGC 1275.

## When to use

Use when you have binned count/flux spectra (e.g. from Fermi-LAT) and need to test for ALP-induced spectral irregularity without directly fitting ALP models to the observed spectrum.

## When NOT to use

- You intend to directly fit ALP models to the observed spectrum to obtain conversion-probability constraints — the article's method explicitly replaces this with an irregularity calculation
- The input is not a binned spectrum (e.g. unbinned event lists) or you lack a fitted null-oscillation model to compute residuals against

## Inputs

- Binned NGC 1275 count/flux spectra from the Fermi-LAT observation (one spectrum per energy binning/configuration)
- Best-fit null-oscillation (conventional, no-ALP) spectral model parameters

## Outputs

- irregularity_table.csv with one irregularity value (residual scatter) per input spectrum

## How to apply

Load the binned NGC 1275 count/flux spectra from the Fermi-LAT observation, then fit the best-fit null-oscillation model (the conventional spectrum model with no ALP-induced oscillation) to each spectrum using a least-squares/minimization routine such as scipy.optimize. Compute the residuals between the observed flux points and the fitted null-oscillation model in each energy bin, and quantify the irregularity of each spectrum as the scatter of these residuals (standard deviation / root-mean-square deviation). Assemble one irregularity value per input spectrum into a single table. The rationale is the article's alternative approach: instead of fitting the observed spectrum with ALP models, the analysis calculates the irregularity of the spectrum; no irregularity signal beyond the null expectation should be claimed, consistent with the paper's reported absence of spectral oscillation for NGC 1275.

## Related tools

- **Fermi-LAT** (Provides the binned gamma-ray count/flux spectra of NGC 1275 whose residual scatter quantifies spectral irregularity)
- **scipy.optimize** (Least-squares/minimization routine used to fit the best-fit null-oscillation model to each spectrum before residual computation)
- **alpconv** (Companion code for calculating ALP-photon conversion in the broader analysis; the null-oscillation fit itself uses no ALP conversion) — github.com/lyf222/alpconv

## Examples

```
import numpy as np; from scipy.optimize import curve_fit; popt, _ = curve_fit(null_model, energy, flux, sigma=flux_err); resid = flux - null_model(energy, *popt); irregularity = np.std(resid)
```

## Evaluation signals

- irregularity_table.csv exists, is a valid CSV, and contains exactly one irregularity value per input spectrum
- All irregularity values in irregularity_table.csv are finite (no NaN/inf)
- Residuals are computed against the best-fit null-oscillation model, not against any ALP-oscillation model
- No irregularity signal beyond the null expectation is claimed, consistent with the paper's reported absence of spectral oscillation for NGC 1275
- Each irregularity value equals the scatter (standard deviation / RMS deviation) of the flux residuals within the corresponding spectrum

## Limitations

- The alpconv repository provides no usage instructions and code comments remain in Chinese, complicating reproducibility of the wider analysis pipeline
- The method quantifies irregularity but does not directly fit ALP models; coupling constraints (e.g. gaγ > 3×10⁻¹² GeV⁻¹ ruled out at ma ∼ 1 neV, 95% CL) come from the surrounding analysis, not the irregularity statistic alone
- Constraints on ALP parameters are only strengthened by combining additional sources (e.g. PKS 2155-304); a single-source irregularity analysis has limited exclusion power

## Evidence

- [abstract] Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum.: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] No spectral oscillation found for NGC 1275 using the irregularity-based method: "With the newly used method, we find no spectral oscillation for the NGC 1275"
- [methods] Residual scatter is computed against the conventional no-ALP model per the task workflow: [evidence span withheld — anchor status: not_found]
- [methods] Output contract for the irregularity table: [evidence span withheld — anchor status: not_found]
- [readme] README caveat on the companion code: "(at present no intruction for use provides, chinese character comments remain in the code.)"
