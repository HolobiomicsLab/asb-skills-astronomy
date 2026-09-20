---
name: fermi-lat-spectral-data-handling
description: Use when re-analyzing a Fermi-LAT gamma-ray observation (e.g. NGC 1275)
  to search for axion-like particle effects, and binned spectral data must be prepared
  for spectral-irregularity computation or ALP-photon conversion runs.
when_to_use_negative:
- When the analysis goal is direct fitting of the observed spectrum with ALP models
  — the paper explicitly adopts an alternative irregularity-based method instead
- When the source observation is not Fermi-LAT spectral data (e.g. X-ray or optical
  spectra), since the binned count/flux spectral handling is tied to Fermi-LAT gamma-ray
  observations
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Fermi-LAT
  role: Instrument providing the gamma-ray observation of NGC 1275 whose binned spectral
    data are handled and re-analyzed
- name: alpconv
  role: Python package for calculating ALP-photon conversion probabilities, consuming
    the prepared spectral data and field configuration
  repo: github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:a304a1d9fea032f751135f3c33253efc5cfe3470a614463aaa4812f15b16c0b8
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
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fermi-lat-spectral-data-handling

## Summary

Handling of binned Fermi-LAT count/flux spectra of NGC 1275 as the input stage for ALP-photon conversion analysis, feeding both the irregularity-based exclusion method and the alpconv propagation code.

## When to use

Use when re-analyzing a Fermi-LAT gamma-ray observation (e.g. NGC 1275) to search for axion-like particle effects, and binned spectral data must be prepared for spectral-irregularity computation or ALP-photon conversion runs.

## When NOT to use

- When the analysis goal is direct fitting of the observed spectrum with ALP models — the paper explicitly adopts an alternative irregularity-based method instead
- When the source observation is not Fermi-LAT spectral data (e.g. X-ray or optical spectra), since the binned count/flux spectral handling is tied to Fermi-LAT gamma-ray observations

## Inputs

- Binned NGC 1275 count/flux spectra from the Fermi-LAT observation
- ALP mass (ma) and coupling (ga-gamma) parameter grid
- Magnetic field configuration (turbulent ICMF baseline or regular-field variant)

## Outputs

- irregularity_table.csv (one irregularity value per input spectrum)
- Null-oscillation model fit residuals per energy bin
- ALP-photon conversion probabilities over the (ma, ga-gamma) grid
- 95% confidence-level exclusion limits in the (ma, ga-gamma) plane

## How to apply

Load the binned NGC 1275 count/flux spectra from the Fermi-LAT observation, since the paper re-analyzes this dataset to search for ALP effects and constrain ALP parameters. These spectra serve two downstream uses: (1) fitting a best-fit null-oscillation model and quantifying the degree of irregularity as the scatter of the fit residuals (the alternative method used instead of fitting the observed spectrum with ALP models), and (2) serving as input to the alpconv package for calculating ALP-photon conversion probabilities over the (ma, ga-gamma) grid. The irregularity values for each spectrum are assembled into a single table, and the derived limits are expressed as 95% confidence-level exclusions in the (ma, ga-gamma) plane, benchmarked against the reported rule-out of ga_gamma > 3x10^-12 GeV^-1 around ma ~ 1 neV.

## Related tools

- **Fermi-LAT** (Instrument providing the gamma-ray observation of NGC 1275 whose binned spectral data are handled and re-analyzed)
- **alpconv** (Python package for calculating ALP-photon conversion probabilities, consuming the prepared spectral data and field configuration) — github.com/lyf222/alpconv

## Examples

```
python -m alpconv --source NGC1275 --field turbulent --ma-grid 1e-11:1e-8 --gag-grid 1e-13:1e-11 --output exclusion_limits.csv
```

## Evaluation signals

- irregularity_table.csv exists, is a valid CSV with one finite irregularity value per input spectrum
- No irregularity signal beyond the null expectation is claimed, consistent with the reported absence of spectral oscillation for NGC 1275
- Derived exclusion limits are expressed at 95% confidence level in the (ma, ga-gamma) plane
- Exclusion region reproduces the reported rule-out of couplings ga_gamma > 3x10^-12 GeV^-1 around ALP mass ma ~ 1 neV
- Conversion probabilities are computed over the full (ma, ga-gamma) grid for the chosen magnetic-field configuration (turbulent baseline or regular-field variant)

## Limitations

- The alpconv repository currently provides no usage instructions, and code comments remain in Chinese, so setup requires reading the referenced paper (arXiv:1802.08420) for method details
- The irregularity-based method quantifies spectral irregularity via residual scatter rather than fitting ALP models directly, so it yields exclusion regions from irregularity rather than likelihood-based ALP fits
- No changelog/discussion documentation is available in the extracted article text, so nuances of the spectral handling must be inferred from the referenced PKS 2155-304 paper

## Evidence

- [abstract] Re-analyze Fermi-LAT observation of NGC 1275: "we re-analyze the Fermi-LAT observation of NGC 1275 to search for axion-like particle (ALP) effects"
- [abstract] Irregularity-based method: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [intro] alpconv tool: "alpconv calculating alp-photon conversion"
- [abstract] 95% CL exclusion benchmark: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [readme] alpconv README: "(at present no intruction for use provides, chinese character comments remain in the code.)"
