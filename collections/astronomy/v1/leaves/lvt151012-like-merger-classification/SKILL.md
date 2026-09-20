---
name: lvt151012-like-merger-classification
description: Use when counting COMPAS BBH mergers resembling LVT151012 (progenitor
  total mass ~90 M⊙ regime at Z=0.001) to test how a single parameter change alters
  the yield of one event class.
when_to_use_negative:
- When the question is the total BBH merger rate alone — the paper finds the LBV variation
  does not significantly change it, so the informative quantity is the event-class
  yield, not the total.
- When classifying GW150914-like (progenitor total mass ≳160 M⊙) or GW151226-like
  (≳60 M⊙) systems — those occupy different progenitor mass regimes and need different
  cuts.
- When a joint/multidimensional parameter-space exploration (e.g. common-envelope
  treatment together with fLBV) is required — this approach varies individual parameters
  independently only.
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to generate the Fiducial and LBV-varied
    BBH populations whose mergers are classified as LVT151012-like
  repo: http://github.com/TeamCOMPAS/COMPAS_
- name: STROOPWAFEL
  role: COMPAS's adaptive importance-sampling algorithm used to draw enough rare heavy
    BBH mergers for statistically meaningful LVT151012-like counts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:31d8faaec6ef607ce0527b082e69e8e9fbf7fdf0b8d1687f21d9e39e7010d2f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# lvt151012-like-merger-classification

## Summary

Classifies and counts COMPAS binary-black-hole mergers similar to LVT151012 (progenitor binaries with total mass in the ≳90 M⊙ regime at Z=0.001) and quantifies how a single-parameter change, e.g. raising the LBV wind mass-loss rate from 1.5×10⁻⁴ to 1×10⁻³ M⊙ yr⁻¹, collapses this event-class yield by a factor of ~10 while leaving the total BBH merger rate unchanged.

## When to use

Use when counting COMPAS BBH mergers resembling LVT151012 (progenitor total mass ~90 M⊙ regime at Z=0.001) to test how a single parameter change alters the yield of one event class.

## When NOT to use

- When the question is the total BBH merger rate alone — the paper finds the LBV variation does not significantly change it, so the informative quantity is the event-class yield, not the total.
- When classifying GW150914-like (progenitor total mass ≳160 M⊙) or GW151226-like (≳60 M⊙) systems — those occupy different progenitor mass regimes and need different cuts.
- When a joint/multidimensional parameter-space exploration (e.g. common-envelope treatment together with fLBV) is required — this approach varies individual parameters independently only.

## Inputs

- COMPAS Fiducial model input settings and reference results at Z=0.001 (LBV mass-loss rate 1.5e-4 M⊙/yr) from the COMPAS Zenodo community
- Modified COMPAS run output with the LBV mass-loss rate set to 1e-3 M⊙/yr, varying only this parameter relative to Fiducial
- COMPAS post-processing scripts for computing merger rates and remnant/progenitor masses
- LVT151012 reference properties: progenitor binary total mass ≳90 M⊙ at Z=0.001

## Outputs

- lbv_variation_results.csv comparison table (fiducial vs modified-run total rates and LVT151012-like counts)
- Count of LVT151012-like BBH mergers per run
- Total BBH merger rate (mergers within a Hubble time per formed population) per run

## How to apply

Run the Fiducial COMPAS configuration at Z=0.001 (5% solar metallicity, LBV mass-loss rate 1.5×10⁻⁴ M⊙ yr⁻¹) and the variant in which only the LBV rate is raised to 1×10⁻³ M⊙ yr⁻¹, drawing m1 from a Kroupa IMF up to 100 M⊙, flat mass ratios, flat-in-log semimajor axes, and zero eccentricity, with STROOPWAFEL importance sampling so the rare heavy channel is well sampled. In each run's post-processed output, select BBHs that merge within a Hubble time whose progenitor binaries have total masses in the ≳90 M⊙ regime characterising LVT151012 (versus ≳160 M⊙ for GW150914 and ≳60 M⊙ for GW151226). Compute the total BBH merger rate (mergers within a Hubble time per formed population; the Fiducial model forms ~3×10⁴ per 10⁹ M⊙ of star formation at 5% solar metallicity) as the control quantity. The decision point is the contrast: the total rate is not significantly changed, while the number of LVT151012-like mergers is reduced by a factor of ~10. Write the fiducial-vs-modified comparison (total rate plus event-class counts) to lbv_variation_results.csv.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to generate the Fiducial and LBV-varied BBH populations whose mergers are classified as LVT151012-like) — http://github.com/TeamCOMPAS/COMPAS_
- **STROOPWAFEL** (COMPAS's adaptive importance-sampling algorithm used to draw enough rare heavy BBH mergers for statistically meaningful LVT151012-like counts)

## Evaluation signals

- Total BBH merger rate in the modified run is not significantly changed relative to Fiducial (the paper's stated invariant).
- Number of LVT151012-like mergers (progenitor total mass ~90 M⊙ regime) is reduced by a factor of ~10 at 5% solar metallicity.
- Only the LBV mass-loss parameter differs between the two runs; all other Fiducial settings (Kroupa IMF m1 ≤ 100 M⊙, flat q, flat-in-log a, e = 0) are identical.
- All counted systems merge within a Hubble time and fall in the ≳90 M⊙ progenitor total-mass regime, not the GW150914 (≳160 M⊙) or GW151226 (≳60 M⊙) regimes.
- lbv_variation_results.csv exists and contains paired fiducial/modified counts for both the total rate and the event-class yield.

## Limitations

- The single-star model grid underlying SSE extends only to 50 M⊙ and is extrapolated above that limit, affecting massive progenitors like those of LVT151012.
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model.
- A full exploration of the multidimensional parameter space (e.g. αλ, fLBV jointly) is deferred to future studies; only individual parameters are varied independently.
- Merger-rate comparisons to LIGO are approximate; an accurate rate requires convolving metallicity-specific, redshift-dependent star formation with the time-delay distribution, which was not done.

## Evidence

- [abstract] Progenitor mass regime defining LVT151012-like systems: "from progenitor binaries with typical total masses ≳160 M ⊙ , ≳60 M ⊙ and ≳90 M ⊙ , for GW150914, GW151226 and"
- [discussion] Expected outcome of the LBV variation on the event class versus the total rate: "does not significantly change the total BBHs merger rate; nevertheless, the number of BBHs mergers similar to LVT151012 was reduced by a factor of ∼10"
- [discussion] Initial conditions under which the classification counts are made: "the primary mass m1 drawn from the Kroupa IMF31 up to m1≤100M⊙"
- [discussion] Why only one parameter is varied at a time: "We leave a full exploration of this parameter space for future studies with COMPAS; here we follow the common approach"
- [readme] What COMPAS evolves to produce the classified remnant populations: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact"
