---
name: multi-source-constraint-combination
description: Use when single-source ALP constraints from Fermi-LAT spectra exist for
  two or more blazars and the goal is a joint 95% CL exclusion region in the (gaγ,
  ma) plane that strictly enlarges each individual contour.
when_to_use_negative:
- Only one source observation is available — the irregularity method applied to a
  single source (e.g. NGC 1275 alone) is the appropriate single-source skill, not
  the combination procedure.
- The goal is model-fitting of the observed spectrum with ALP models rather than the
  irregularity-based statistical combination — the paper explicitly adopts the irregularity
  method 'instead of fitting the observed spectrum with ALP models'.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: alpconv
  role: Computes ALP-photon conversion probabilities along the sightlines of NGC 1275
    and PKS 2155-304 so the irregularity statistic can be evaluated on the combined
    dataset.
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Gamma-ray spectra of NGC 1275 and PKS 2155-304 that supply the irregularity
    statistics combined into the joint exclusion region.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:db05d4d2634f8f60a3c3bdfbd3dbae2b126515e8354f5f61e1bbc6a885e8d78a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multi-source-constraint-combination

## Summary

Combines ALP exclusion regions from multiple Fermi-LAT source observations (e.g. NGC 1275 and PKS 2155-304) by merging their spectrum-irregularity statistics into a joint statistic, yielding a combined gaγ–ma exclusion contour stronger than either single source.

## When to use

Use when single-source ALP constraints from Fermi-LAT spectra exist for two or more blazars and the goal is a joint 95% CL exclusion region in the (gaγ, ma) plane that strictly enlarges each individual contour.

## When NOT to use

- Only one source observation is available — the irregularity method applied to a single source (e.g. NGC 1275 alone) is the appropriate single-source skill, not the combination procedure.
- The goal is model-fitting of the observed spectrum with ALP models rather than the irregularity-based statistical combination — the paper explicitly adopts the irregularity method 'instead of fitting the observed spectrum with ALP models'.

## Inputs

- Single-source irregularity exclusion regions in the gaγ–ma plane for NGC 1275 and PKS 2155-304 (Fermi-LAT spectra)
- ALP-photon conversion probabilities along both sources' sightlines computed with alpconv
- Coupling–mass grid points (gaγ, ma) to scan

## Outputs

- combined_exclusion_region.csv — joint 95% CL exclusion region grid in the (gaγ, ma) plane
- combined_vs_single_bounds.csv — table of single-source vs combined bounds on gaγ (e.g. around ma ∼ 1 neV)

## How to apply

Load the single-source irregularity-based exclusion regions previously derived in the gaγ–ma plane for each source (NGC 1275 and PKS 2155-304). Use alpconv to compute ALP-photon conversion probabilities along both sightlines so the irregularity statistic can be evaluated per source. Scan the coupling–mass grid (gaγ, ma) and combine the two sources' irregularity statistics into a joint statistic, applying the 95% confidence level threshold to obtain the combined exclusion region. Compare the combined contour against each single-source contour to quantify the gaγ improvement (e.g. around ma ∼ 1 neV). Write the joint exclusion grid to combined_exclusion_region.csv and tabulate single-source vs combined bounds in combined_vs_single_bounds.csv. The rationale is that spectral irregularity induced by ALP conversion adds statistically across sightlines, so the combined constraint rules out couplings gaγ > 3×10⁻¹² GeV⁻¹ around ma ∼ 1 neV and enlarges the excluded region further than either observation alone.

## Related tools

- **alpconv** (Computes ALP-photon conversion probabilities along the sightlines of NGC 1275 and PKS 2155-304 so the irregularity statistic can be evaluated on the combined dataset.) — github.com/lyf222/alpconv
- **Fermi-LAT** (Gamma-ray spectra of NGC 1275 and PKS 2155-304 that supply the irregularity statistics combined into the joint exclusion region.)

## Examples

```
python alpconv.py --source NGC1275 PKS2155-304; # scan (gaγ, ma) grid, combine irregularity stats at 95% CL -> combined_exclusion_region.csv
```

## Evaluation signals

- The combined exclusion region must strictly enlarge (contain) both single-source exclusion regions in the gaγ–ma plane.
- Around ma ∼ 1 neV, the combined bound must rule out couplings gaγ > 3×10⁻¹² GeV⁻¹ at 95% confidence level.
- The combined bounds must be more stringent than each single-source bound (NGC 1275 alone or PKS 2155-304 alone) across the scanned mass range.
- combined_exclusion_region.csv must be a gridded (gaγ, ma) table at the 95% CL threshold, and combined_vs_single_bounds.csv must contain comparable single-source vs combined gaγ limits.
- Conversion probabilities used in the joint statistic must be reproducible with alpconv for both sightlines.

## Limitations

- alpconv presently provides no usage instructions and code comments remain in Chinese, complicating reuse.
- No spectral oscillation signal was found for NGC 1275 with the irregularity method; the improvement comes from statistical combination rather than a detection.
- The approach is limited to the irregularity statistic; wider exclusion regions require incorporating more sources, as the authors suggest.

## Evidence

- [abstract] combined improvement finding: "We also show that the constraints can be further improved by combining the observation of PKS 2155-304."
- [abstract] irregularity method rationale: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [abstract] combined 95% CL threshold: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [readme] alpconv purpose: "alpconv calculating alp-photon conversion"
- [readme] alpconv usability caveat: "(at present no intruction for use provides, chinese character comments remain in the code.)"
