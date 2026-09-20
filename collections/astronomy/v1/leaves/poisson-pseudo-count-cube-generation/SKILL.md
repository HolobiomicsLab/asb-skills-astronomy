---
name: poisson-pseudo-count-cube-generation
description: 'Use when an irregularity-based ALP spectral analysis needs null-distribution
  pseudo counts: input is a -fit ROI model counts map (FITS) and a scanned (ma, gaγ)
  grid, and output is 50,000 simulated counts cubes per parameter set plus a manifest.'
when_to_use_negative:
- The goal is to fit the observed spectrum with ALP models rather than to evaluate
  the spectrum irregularity statistic — the paper explicitly adopts the irregularity
  method 'instead of fitting the observed spectrum with ALP models'.
- No best-fit ROI model counts map exists yet; this skill depends on the upstream
  best-fit spectral-modeling step.
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: Fermi-LAT
  role: Source of the NGC 1275 gamma-ray observation; the best-fit ROI model map and
    pseudo counts cubes are built on Fermi-LAT counts data.
- name: alpconv
  role: Calculates ALP-photon conversion probabilities for each (ma, gaγ) parameter
    set on the scanned grid, defining the parameter sets for which pseudo counts cubes
    are generated.
  repo: https://github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:e3ec7ee77ee091aed526b1f3ef005c4dee491e74b86a3b35682d7ba5e56700d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lyf222/alpconv
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# poisson-pseudo-count-cube-generation

## Summary

Generates Monte Carlo pseudo counts cubes by adding Poisson fluctuations to the best-fit ROI model counts map for each ALP parameter set (ma, gaγ), providing the null-likelihood realizations needed to evaluate the spectrum irregularity statistic in the NGC 1275 Fermi-LAT ALP analysis.

## When to use

Use when an irregularity-based ALP spectral analysis needs null-distribution pseudo counts: input is a best-fit ROI model counts map (FITS) and a scanned (ma, gaγ) grid, and output is 50,000 simulated counts cubes per parameter set plus a manifest.

## When NOT to use

- The goal is to fit the observed spectrum with ALP models rather than to evaluate the spectrum irregularity statistic — the paper explicitly adopts the irregularity method 'instead of fitting the observed spectrum with ALP models'.
- No best-fit ROI model counts map exists yet; this skill depends on the upstream best-fit spectral-modeling step.

## Inputs

- Best-fit model counts map of the NGC 1275 ROI (FITS) from upstream best-fit spectral modeling
- ALP parameter grid of (ma, gaγ) pairs to scan

## Outputs

- 50,000 pseudo counts cube FITS files per ALP parameter set (500 realizations × 100 pseudo spectra)
- Manifest file enumerating the complete generated set of pseudo counts cubes

## How to apply

Load the best-fit model counts map of the NGC 1275 ROI produced by the upstream best-fit spectral-modeling step. For each ALP parameter set (ma, gaγ) on the scanned grid, add Poisson fluctuations to the best-fit model map to draw pseudo counts, assembling 500 realizations × 100 pseudo spectra, i.e. 50,000 pseudo counts cubes per parameter set. Serialize each pseudo counts cube to disk in FITS format, and write a manifest file enumerating the complete generated set. The large ensemble (50,000 cubes) is what lets the irregularity statistic be evaluated with proper statistical power for every point on the ALP parameter scan.

## Related tools

- **Fermi-LAT** (Source of the NGC 1275 gamma-ray observation; the best-fit ROI model map and pseudo counts cubes are built on Fermi-LAT counts data.)
- **alpconv** (Calculates ALP-photon conversion probabilities for each (ma, gaγ) parameter set on the scanned grid, defining the parameter sets for which pseudo counts cubes are generated.) — https://github.com/lyf222/alpconv

## Evaluation signals

- Exactly 50,000 pseudo counts cube FITS files exist per ALP parameter set.
- The manifest file lists the full generated set: file count in the manifest equals the number of cubes on disk.
- Ensemble structure per parameter set is 500 realizations × 100 pseudo spectra.
- Each pseudo counts cube is a valid FITS file that can be loaded by downstream irregularity-statistic evaluation.
- Pseudo counts are Poisson fluctuations of the best-fit model map, not of the raw observed counts.

## Limitations

- The alpconv repository provides no usage instructions and code comments remain in Chinese: '(at present no intruction for use provides, chinese character comments remain in the code.)'
- The method is tied to the NGC 1275 Fermi-LAT ROI analysis; no spectral oscillation was found with it for this source, and constraints rely on the irregularity statistic rather than direct spectral fitting.
- Validation of the cube set is purely file-count/manifest-based; the described workflow does not include statistical goodness-of-fit checks of the pseudo counts distribution.

## Evidence

- [methods] 50,000 counts cubes are simulated from Poisson fluctuations added to the best-fit ROI model map: [evidence span withheld — anchor status: not_found]
- [methods] verify that exactly 50,000 pseudo counts cube FITS files exist: [evidence span withheld — anchor status: not_found]
- [abstract] alternative method calculating the irregularity of the spectrum: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [readme] alpconv calculating alp-photon conversion: "calculating alp-photon conversion"
- [readme] no usage instructions, Chinese comments: "(at present no intruction for use provides, chinese character comments remain in the code.)"
