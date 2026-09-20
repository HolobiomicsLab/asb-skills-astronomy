---
name: alp-parameter-bound-tabulation
description: Use when Fermi-LAT spectra of one or more ALP-sensitive sources are available
  and you need 95% CL exclusion contours in the (gaγ, ma) grid, especially combining
  sources for improved bounds.
when_to_use_negative:
- When the goal is spectral fitting of ALP models to an observed spectrum — this skill
  uses the irregularity-of-the-spectrum method explicitly as an alternative to that
  fitting approach.
- When the input is a single-source analysis only and no second source's irregularity
  results are available to form the joint statistic.
- When the target is a source with a detected spectral-oscillation signature rather
  than a null-oscillation constraint region — the article found no spectral oscillation
  for NGC 1275.
tools:
- name: alpconv
  role: Computes ALP-photon conversion probabilities along source sightlines so the
    irregularity statistic can be evaluated on combined datasets
  repo: github.com/lyf222/alpconv
- name: Fermi-LAT
  role: Provides the gamma-ray spectra of NGC 1275 and PKS 2155-304 that feed the
    irregularity-based ALP analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.physletb.2021.136611
    title: Revisiting the analysis of axion-like particles with the Fermi-LAT gamma-ray
      observation of NGC1275
  source_content_hash: sha256:d462803f5a5ef2f32169188e594393bd91354b74d1c246538950bb17b763ade3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fermi-LAT
  - alpconv
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# alp-parameter-bound-tabulation

## Summary

Tabulates axion-like particle (ALP) exclusion regions in the gaγ–ma plane by combining spectral-irregularity statistics from multiple Fermi-LAT gamma-ray sources (NGC 1275 + PKS 2155-304) using alpconv, yielding joint bounds stricter than single-source analyses.

## When to use

Use when Fermi-LAT spectra of one or more ALP-sensitive sources are available and you need 95% CL exclusion contours in the (gaγ, ma) grid, especially combining sources for improved bounds.

## When NOT to use

- When the goal is spectral fitting of ALP models to an observed spectrum — this skill uses the irregularity-of-the-spectrum method explicitly as an alternative to that fitting approach.
- When the input is a single-source analysis only and no second source's irregularity results are available to form the joint statistic.
- When the target is a source with a detected spectral-oscillation signature rather than a null-oscillation constraint region — the article found no spectral oscillation for NGC 1275.

## Inputs

- Fermi-LAT gamma-ray spectra of ALP-sensitive sources (e.g. NGC 1275, PKS 2155-304)
- Single-source exclusion regions in the gaγ–ma plane from irregularity analysis
- ALP-photon conversion probabilities along each source sightline computed with alpconv

## Outputs

- combined_exclusion_region.csv — joint exclusion-region grid over (gaγ, ma) at 95% CL
- combined_vs_single_bounds.csv — table comparing single-source vs combined bounds on gaγ (e.g. around ma ∼ 1 neV)

## How to apply

Load the single-source irregularity analysis results (exclusion regions in the gaγ–ma plane) previously derived from Fermi-LAT spectra of NGC 1275 and PKS 2155-304. Use alpconv to compute ALP-photon conversion probabilities along both sightlines so the spectral-irregularity statistic — chosen because it avoids fitting observed spectra with ALP models — can be evaluated on the combined dataset. Scan the coupling–mass grid (gaγ, ma) and merge the per-source irregularity statistics into a joint statistic, applying the 95% confidence level threshold to obtain the combined exclusion region. Verify the combined contour strictly enlarges both single-source regions; the expected improvement rules out couplings gaγ > 3×10⁻¹² GeV⁻¹ around ma ∼ 1 neV at 95% CL. Write the joint grid to combined_exclusion_region.csv and a single-source vs combined comparison to combined_vs_single_bounds.csv.

## Related tools

- **alpconv** (Computes ALP-photon conversion probabilities along source sightlines so the irregularity statistic can be evaluated on combined datasets) — github.com/lyf222/alpconv
- **Fermi-LAT** (Provides the gamma-ray spectra of NGC 1275 and PKS 2155-304 that feed the irregularity-based ALP analysis)

## Evaluation signals

- The combined exclusion region must strictly enlarge (contain) both the NGC 1275 and PKS 2155-304 single-source exclusion regions.
- The joint bounds must reproduce the ruling out of couplings gaγ > 3×10⁻¹² GeV⁻¹ around ALP mass ma ∼ 1 neV at 95% confidence level.
- combined_vs_single_bounds.csv must quantify the improvement of combined bounds over each single-source bound (e.g. around ma ∼ 1 neV).
- The exclusion-region grid must span the scanned (gaγ, ma) plane and be thresholded at the 95% confidence level.

## Limitations

- alpconv currently ships with no usage instructions and Chinese-character comments remain in the code, hindering reuse.
- No spectral oscillation signal was found for NGC 1275 itself — the constraint arises from irregularity statistics, not a detection.
- Wider exclusion regions require more sources taken into account; the two-source combination is a partial step toward that.

## Evidence

- [abstract] 95% CL bound: "rule out couplings gaγ>3×10−12GeV−1 around ALP mass of ma∼1 neV at 95% confidence level, which is more stringent than the previous results"
- [abstract] combining sources: "We also show that the constraints can be further improved by combining the observation of PKS 2155-304."
- [abstract] irregularity method: "Instead of fitting the observed spectrum with ALP models, we adopt an alternative method for the analysis of this source which calculates the irregularity of the spectrum."
- [readme] alpconv purpose: "calculating alp-photon conversion"
- [readme] alpconv caveat: "at present no intruction for use provides, chinese character comments remain in the code."
