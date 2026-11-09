---
name: fit-residual-scatter-analysis
description: Use when you have binned Fermi-LAT count/flux spectra of a point source
  and want to test for ALP-induced spectral oscillation via spectrum irregularity
  instead of direct ALP-model fitting of the observed spectrum.
when_to_use_negative:
- When the goal is to directly fit the observed spectrum with ALP models to obtain
  conversion-probability fits rather than an irregularity statistic — the paper explicitly
  adopts this method 'instead of fitting the observed spectrum with ALP models'.
- When spectra are unbinned or have too few energy bins for residuals per bin to define
  a meaningful scatter.
- When the source or analysis requires coupling/mass exclusion regions (e.g. gaγ >
  3×10⁻¹² GeV⁻¹ near ma ∼ 1 neV) — those come from the paper's separate constraint
  analysis, not from this irregularity statistic.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Fermi-LAT
  role: Source of the binned NGC 1275 gamma-ray spectra on which the null-oscillation
    model is fit and residuals are computed
- name: scipy.optimize
  role: Least-squares/minimization routine used to fit the best-fit null-oscillation
    model to each binned spectrum
- name: alpconv
  role: Companion code for calculating ALP-photon conversion probabilities in the
    broader re-analysis; irregularity results feed the ALP constraint interpretation
  repo: github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:2a678e294a4c53646e960bf3fc9797ecf4d00573c6862be7be1b44497c8b45d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  - scipy.optimize
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# fit-residual-scatter-analysis

## Summary

Quantifies spectral irregularity of the Fermi-LAT NGC 1275 observation as the scatter of residuals from a best-fit null-oscillation model, rather than fitting the spectrum directly with ALP models. It is the irregularity-based method this paper uses to test for ALP-induced spectral oscillation.

## When to use

Use when you have binned Fermi-LAT count/flux spectra of a point source and want to test for ALP-induced spectral oscillation via spectrum irregularity instead of direct ALP-model fitting of the observed spectrum.

## When NOT to use

- When the goal is to directly fit the observed spectrum with ALP models to obtain conversion-probability fits rather than an irregularity statistic — the paper explicitly adopts this method 'instead of fitting the observed spectrum with ALP models'.
- When spectra are unbinned or have too few energy bins for residuals per bin to define a meaningful scatter.
- When the source or analysis requires coupling/mass exclusion regions (e.g. gaγ > 3×10⁻¹² GeV⁻¹ near ma ∼ 1 neV) — those come from the paper's separate constraint analysis, not from this irregularity statistic.

## Inputs

- Binned Fermi-LAT count/flux spectra of NGC 1275 (one spectrum per observation/epoch)
- Best-fit null-oscillation (conventional, no-ALP) spectral model parameters

## Outputs

- irregularity_table.csv: one irregularity value (scatter of fit residuals) per input spectrum
- Per-spectrum null-oscillation model fits and residual scatter values

## How to apply

Load the binned NGC 1275 count/flux spectra from the Fermi-LAT observation. Fit the best-fit null-oscillation model (the conventional spectrum with no ALP-induced oscillation) to each spectrum using a least-squares minimization routine such as scipy.optimize. Compute residuals between the observed flux points and the fitted null model in each energy bin, then quantify irregularity as the scatter of these residuals (standard deviation / RMS deviation). The rationale, per the paper, is that ALP-photon conversion imprints an irregular, oscillatory pattern on the spectrum, so irregularity statistics can be evaluated against the null expectation without explicit ALP-model fits. Assemble one irregularity value per input spectrum into a table (irregularity_table.csv); absence of irregularity beyond the null expectation corresponds to the paper's finding of no spectral oscillation for NGC 1275.

## Related tools

- **Fermi-LAT** (Source of the binned NGC 1275 gamma-ray spectra on which the null-oscillation model is fit and residuals are computed)
- **scipy.optimize** (Least-squares/minimization routine used to fit the best-fit null-oscillation model to each binned spectrum)
- **alpconv** (Companion code for calculating ALP-photon conversion probabilities in the broader re-analysis; irregularity results feed the ALP constraint interpretation) — github.com/lyf222/alpconv

## Examples

```
from scipy.optimize import least_squares; fit = least_squares(resid_fn, p0, args=(E, flux)); sigma_irr = np.std(flux - model(fit.x, E))
```

## Evaluation signals

- irregularity_table.csv exists, is a valid CSV, and contains exactly one irregularity value per input spectrum
- All irregularity values are finite (no NaN/inf from the residual scatter computation)
- Null-oscillation fit converges for every spectrum before residuals are computed (scipy.optimize terminates without error)
- No irregularity signal beyond the null expectation is claimed, consistent with the paper's reported absence of spectral oscillation for NGC 1275
- Residuals are computed in each energy bin against the fitted null model, not against ALP models

## Limitations

- The method reports presence/absence and irregularity statistics; the ALP coupling constraint (gaγ > 3×10⁻¹² GeV⁻¹ around ma ∼ 1 neV at 95% CL) requires the separate conversion-probability analysis, not this skill alone.
- Constraints are source-limited: the paper notes improvement only by combining PKS 2155-304 or more sources ('with more sources taken into account, we could obtain a much wider exclusion region').
- The companion alpconv repository provides no usage instructions and retains Chinese-character comments in the code, so reproducing the pipeline may require reimplementation.

## Evidence

- [abstract] Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum.: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] With the newly used method, we find no spectral oscillation for the NGC 1275: "With the newly used method, we find no spectral oscillation for the NGC 1275"
- [methods] The degree of irregularity of a spectrum is estimated as the scatter of the fit residuals (of the best-fit null-oscillation model), as suggested in a referenced work.: [evidence span withheld — anchor status: not_found]
- [readme] (at present no intruction for use provides, chinese character comments remain in the code.): "(at present no intruction for use provides, chinese character comments remain in the code.)"
