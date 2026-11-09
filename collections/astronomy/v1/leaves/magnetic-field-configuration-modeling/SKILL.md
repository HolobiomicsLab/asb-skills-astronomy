---
name: magnetic-field-configuration-modeling
description: Use when ALP-photon conversion limits derived under a turbulent ICM field
  baseline must be checked for robustness against a regular magnetic field configuration,
  e.g. varying the assumed ICMF/GMF model in an alpconv-based analysis.
when_to_use_negative:
- When the goal is to fit the observed spectrum with ALP models rather than apply
  the irregularity-based method — the paper explicitly adopts the irregularity approach
  'instead of fitting the observed spectrum with ALP models'.
- When only the turbulent-field baseline limits are needed and no magnetic field model
  sensitivity check is requested.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: alpconv
  role: Calculates ALP-photon conversion probabilities; configured for the regular
    ICMF/GMF field variant instead of the turbulent configuration when running this
    sensitivity test.
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Source of the NGC 1275 gamma-ray observation that is re-analyzed to search
    for ALP effects and constrain ALP parameters.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:07dde370f8e7ab78640f8f86c7be71b444ae2c1e65c1f8e6ca510d7481bc7fba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# magnetic-field-configuration-modeling

## Summary

Sensitivity-test skill for ALP exclusion limits: recompute ALP-photon conversion probabilities under an alternative regular ICMF/GMF magnetic field configuration instead of the turbulent-field baseline, then quantify how the (ma, ga-gamma) exclusion region changes.

## When to use

Use when ALP-photon conversion limits derived under a turbulent ICM field baseline must be checked for robustness against a regular magnetic field configuration, e.g. varying the assumed ICMF/GMF model in an alpconv-based analysis.

## When NOT to use

- When the goal is to fit the observed spectrum with ALP models rather than apply the irregularity-based method — the paper explicitly adopts the irregularity approach 'instead of fitting the observed spectrum with ALP models'.
- When only the turbulent-field baseline limits are needed and no magnetic field model sensitivity check is requested.

## Inputs

- NGC 1275 Fermi-LAT spectral/likelihood data used in the paper's re-analysis
- Turbulent-field baseline exclusion limits in the (ma, ga-gamma) plane from a prior run
- Regular ICMF/GMF magnetic field configuration as described in the referenced magnetic field model

## Outputs

- regular_field_limits.csv (95% CL exclusion limits in (ma, ga-gamma) for the regular-field variant)
- regular_vs_turbulent_comparison.json (change in excluded region relative to the turbulent baseline)

## How to apply

Load the NGC 1275 Fermi-LAT spectral/likelihood data used in the re-analysis together with the turbulent-field baseline exclusion limits from the prior run. Configure the alpconv package (github.com/lyf222/alpconv), which calculates ALP-photon conversion, to use the regular ICMF/GMF field variant described in the referenced magnetic field model rather than the turbulent configuration. Compute the ALP-photon conversion probabilities over the ALP mass (ma) and coupling (ga-gamma) grid for the regular-field setup. Apply the paper's irregularity-based method — which calculates the irregularity of the spectrum instead of fitting the observed spectrum with ALP models — to derive 95% confidence-level exclusion limits in the (ma, ga-gamma) plane. Tabulate the regular-field limits and compare the excluded region against the turbulent-field baseline to quantify the change in stringency.

## Related tools

- **alpconv** (Calculates ALP-photon conversion probabilities; configured for the regular ICMF/GMF field variant instead of the turbulent configuration when running this sensitivity test.) — github.com/lyf222/alpconv
- **Fermi-LAT** (Source of the NGC 1275 gamma-ray observation that is re-analyzed to search for ALP effects and constrain ALP parameters.)

## Evaluation signals

- Derived limits are expressed as 95% confidence-level exclusions in the (ma, ga-gamma) plane.
- Comparison record captures that couplings ga-gamma > 3x10^-12 GeV^-1 around ALP mass ma ~ 1 neV are ruled out, matching the paper's reported stringency level.
- Conversion probabilities are computed over the full ALP mass (ma) and coupling (ga-gamma) grid, not a subsampled region.
- The regular-field setup is clearly distinguished from the turbulent-field baseline in the comparison output.
- The irregularity-based method (spectral irregularity calculation) is used rather than ALP model fitting.

## Limitations

- The alpconv repository provides no usage instructions at present, and code comments remain in Chinese.
- No discussion-section documentation is available for the alpconv package; the repo contains only a changelog placeholder and source reference.
- Findings are specific to NGC 1275 (no spectral oscillation detected); combining additional sources such as PKS 2155-304 is needed to further improve constraints.

## Evidence

- [abstract] Regular-field variant limits reproduce paper stringency: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [abstract] Irregularity-based method replaces ALP model fitting: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [intro] alpconv computes ALP-photon conversion: "alpconv calculating alp-photon conversion"
- [readme] alpconv repo lacks usage docs: "(at present no intruction for use provides, chinese character comments remain in the code.)"
- [abstract] Re-analysis targets NGC 1275 ALP constraints: "we re-analyze the Fermi-LAT observation of NGC 1275 to search for axion-like particle (ALP) effects and constrain ALP parameters"
