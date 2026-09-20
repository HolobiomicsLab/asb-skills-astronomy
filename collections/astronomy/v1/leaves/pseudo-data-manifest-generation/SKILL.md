---
name: pseudo-data-manifest-generation
description: Use when an irregularity-statistic ALP analysis needs Monte Carlo pseudo
  counts cubes sampled from a -fit ROI model map, plus a manifest tracking the full
  generated set per ALP parameter point.
when_to_use_negative:
- When the observed spectrum is to be fitted directly with ALP models instead of the
  irregularity-based method (the paper explicitly adopts the irregularity approach
  instead of such fitting)
- When pseudo counts are needed for a source other than the NGC 1275 Fermi-LAT ROI
  without an upstream best-fit ROI model map to fluctuate
- When raw event-level Fermi-LAT data still need likelihood-based spectral modeling
  — this skill consumes the best-fit ROI model map, it does not produce it
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Fermi-LAT
  role: Source of the NGC 1275 gamma-ray observation from which the best-fit ROI model
    map (the basis for Poisson pseudo-count sampling) is derived
- name: alpconv
  role: Companion tool for calculating ALP-photon conversion across the scanned (ma,
    gaγ) parameter grid; README notes no usage instructions exist and code comments
    remain in Chinese
  repo: github.com/lyf222/alpconv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:814125a6c81a71d95177df2bbfe239dcbf667ad128577a3b5f13aea0706edb5c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pseudo-data-manifest-generation

## Summary

Generates Monte Carlo pseudo-count cubes for the NGC 1275 Fermi-LAT ALP analysis by adding Poisson fluctuations to the best-fit ROI model map, and writes a manifest enumerating every generated FITS file.

## When to use

Use when an irregularity-statistic ALP analysis needs Monte Carlo pseudo counts cubes sampled from a best-fit ROI model map, plus a manifest tracking the full generated set per ALP parameter point.

## When NOT to use

- When the observed spectrum is to be fitted directly with ALP models instead of the irregularity-based method (the paper explicitly adopts the irregularity approach instead of such fitting)
- When pseudo counts are needed for a source other than the NGC 1275 Fermi-LAT ROI without an upstream best-fit ROI model map to fluctuate
- When raw event-level Fermi-LAT data still need likelihood-based spectral modeling — this skill consumes the best-fit ROI model map, it does not produce it

## Inputs

- Best-fit model counts map of the NGC 1275 ROI from the upstream best-fit spectral-modeling step
- Scanned grid of ALP parameters (ma, gaγ)

## Outputs

- 50,000 pseudo counts cube FITS files per ALP parameter set
- Manifest file enumerating the complete generated set of pseudo counts cubes

## How to apply

Load the best-fit model counts map of the NGC 1275 ROI produced by the upstream best-fit spectral-modeling step. For each (ma, gaγ) point on the scanned ALP parameter grid, add Poisson fluctuations to the best-fit model map to draw pseudo counts, assembling 500 realizations × 100 pseudo spectra per parameter set (50,000 pseudo counts cubes total). Serialize each pseudo counts cube to disk in FITS format. Finally, write a manifest file enumerating the complete set of generated pseudo counts cubes so downstream irregularity-statistic evaluation can be validated against the full sample.

## Related tools

- **Fermi-LAT** (Source of the NGC 1275 gamma-ray observation from which the best-fit ROI model map (the basis for Poisson pseudo-count sampling) is derived)
- **alpconv** (Companion tool for calculating ALP-photon conversion across the scanned (ma, gaγ) parameter grid; README notes no usage instructions exist and code comments remain in Chinese) — github.com/lyf222/alpconv

## Evaluation signals

- Exactly 50,000 pseudo counts cube FITS files exist on disk per ALP parameter set
- The manifest lists the full generated set: file count in the manifest equals the number of cubes on disk
- Each cube can be traced to one of 500 realizations × 100 pseudo spectra per (ma, gaγ) point
- Serialized cubes parse as valid FITS files
- Pseudo counts are consistent with Poisson fluctuations drawn from the best-fit model map (statistical distribution check)

## Limitations

- alpconv provides no usage instructions and its code comments remain in Chinese (per README), so adapting it requires reading the source
- The pseudo-count generation is tied to the best-fit ROI model map of the specific NGC 1275 ROI; reusing it for another source requires an equivalent upstream best-fit model
- Cost scales with the ALP parameter grid: 50,000 cubes per (ma, gaγ) point can be substantial for dense scans

## Evidence

- [methods] Poisson fluctuations on best-fit ROI model map: [evidence span withheld — anchor status: not_found]
- [abstract] Irregularity-based method rationale: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [readme] alpconv tool and README caveat: "(at present no intruction for use provides, chinese character comments remain in the code.)"
- [intro] ALP-photon conversion calculation: "alpconv calculating alp-photon conversion"
