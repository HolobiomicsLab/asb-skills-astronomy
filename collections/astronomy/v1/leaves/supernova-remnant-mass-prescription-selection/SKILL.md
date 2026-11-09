---
name: supernova-remnant-mass-prescription-selection
description: Use when a population-synthesis result (e.g. heavy BBH formation channels,
  fallback fractions, remnant masses) rests on a supernova engine choice and you need
  to demonstrate the pathway finding is robust to switching between the delayed and
  rapid Fryer prescriptions.
when_to_use_negative:
- When the scientific claim concerns total BBH merger rates already shown to be sensitive
  to other parameters (e.g. optimistic common-envelope treatment raising rates ~3x,
  or alpha_lambda 0.1→0.01 dropping rates ~2-4x) — vary those parameters instead;
  the rapid-vs-delayed check targets pathway robustness, not rate calibration.
- When studying sub-50 Msun progenitors where the single-star model grid is not extrapolated
  — the fallback-based heavy-BBH comparison is motivated by primaries up to 100 Msun
  where SSE grids extend only to 50 Msun.
- When the model already includes physics (stellar rotation, tides, wind accretion)
  outside the Fiducial scope, since prescription swaps should be tested one-at-a-time
  against a fixed Fiducial baseline.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: COMPAS
  role: Rapid binary population synthesis code in which the supernova remnant-mass
    prescription (delayed vs rapid Fryer) is toggled and binaries are evolved from
    ZAMS to two compact remnants
  repo: http://github.com/TeamCOMPAS/COMPAS_
- name: STROOPWAFEL
  role: COMPAS importance-sampling algorithm used to efficiently sample the merging-BBH
    channel at each metallicity so both prescriptions are compared on equivalent populations
  repo: https://compas.science/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:f720c0094616a7da8f39a81697020a4993070960791d2dc4762742b6745e9695
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# supernova-remnant-mass-prescription-selection

## Summary

A sensitivity-check workflow for binary population synthesis that swaps the supernova remnant-mass prescription (e.g. the Fiducial 'delayed' model for the 'rapid' model of Fryer et al.) in COMPAS and verifies whether evolutionary pathways to heavy BBH mergers are robust. High-mass BH formation proceeds through almost complete fallback under both prescriptions, so pathway conclusions should not hinge on this choice.

## When to use

Use when a population-synthesis result (e.g. heavy BBH formation channels, fallback fractions, remnant masses) rests on a supernova engine choice and you need to demonstrate the pathway finding is robust to switching between the delayed and rapid Fryer prescriptions.

## When NOT to use

- When the scientific claim concerns total BBH merger rates already shown to be sensitive to other parameters (e.g. optimistic common-envelope treatment raising rates ~3x, or alpha_lambda 0.1→0.01 dropping rates ~2-4x) — vary those parameters instead; the rapid-vs-delayed check targets pathway robustness, not rate calibration.
- When studying sub-50 Msun progenitors where the single-star model grid is not extrapolated — the fallback-based heavy-BBH comparison is motivated by primaries up to 100 Msun where SSE grids extend only to 50 Msun.
- When the model already includes physics (stellar rotation, tides, wind accretion) outside the Fiducial scope, since prescription swaps should be tested one-at-a-time against a fixed Fiducial baseline.

## Inputs

- COMPAS Fiducial-model configuration (Kroupa IMF to 100 Msun, uniform mass ratio, flat-in-log a, e=0, all stars in binaries)
- Supernova prescription setting: 'delayed' (fiducial) and 'rapid' (Fryer et al.)
- Metallicity grid: 10% and 5% solar metallicity (e.g. Z=0.001)
- STROOPWAFEL importance-sampling run targeting the merging-BBH channel

## Outputs

- Per-prescription set of double compact objects merging within a Hubble time, with component BH masses
- Fallback fractions for each compact-object formation event
- Cross-prescription comparison of BBH formation pathways (mass transfer, common envelope, remnant masses) and merger counts

## How to apply

Configure COMPAS with the Fiducial-model setup — primary masses drawn from a Kroupa IMF up to 100 Msun, uniform mass ratio, flat-in-the-log semi-major axis, initial eccentricity of zero, all stars in binaries — then toggle the supernova prescription from 'delayed' to the 'rapid' model of Fryer et al. Run the population at low metallicity (10% and 5% solar, e.g. Z=0.001) using the STROOPWAFEL importance-sampling algorithm restricted to the merging-BBH channel, so each prescription is compared on identical initial distributions. Extract all double compact objects that merge within a Hubble time, recording component BH masses and fallback fractions, and compare the distributions of mass transfer, common-envelope and remnant-mass outcomes between the two runs. The decision criterion is whether typical heavy-BBH evolutionary pathways and BBH merger counts change materially; in this study both prescriptions predict high-mass BH formation through almost complete fallback, so the Fiducial (delayed) pathway was retained.

## Related tools

- **COMPAS** (Rapid binary population synthesis code in which the supernova remnant-mass prescription (delayed vs rapid Fryer) is toggled and binaries are evolved from ZAMS to two compact remnants) — http://github.com/TeamCOMPAS/COMPAS_
- **STROOPWAFEL** (COMPAS importance-sampling algorithm used to efficiently sample the merging-BBH channel at each metallicity so both prescriptions are compared on equivalent populations) — https://compas.science/

## Evaluation signals

- Both the delayed and rapid runs must recover high-mass BH formation through almost complete fallback for the heavy-BBH channel.
- Typical evolutionary pathways (mass transfer stage, common-envelope occurrence, remnant-mass distribution) should match between the two prescriptions for heavy BBHs.
- BBH merger counts per 1e9 solar masses of star formation should be comparable between prescriptions (Fiducial reference: ~2e4 at 10% solar metallicity, ~3e4 at 5%).
- Confirm only the supernova prescription was varied: all other Fiducial settings (IMF, mass-ratio, semi-major-axis and eccentricity distributions, metallicity, STROOPWAFEL seeds) must be identical across runs.
- Verify the extracted DCO sample is restricted to systems merging within a Hubble time.

## Limitations

- Only individual parameters were varied against the Fiducial model; a full exploration of the multidimensional parameter space (e.g. alpha, fLBV) is deferred to future studies.
- The original grid of single star models underlying SSE extends only to 50 solar masses and is extrapolated above that limit, which may affect the most massive fallback-dominated progenitors.
- The comparison is pathway-level, not a rate prediction: an accurate merger rate requires convolving metallicity-specific redshift-dependent star formation with the time-delay distribution, which was not done.
- Stellar rotation, tides and wind accretion by a companion are not included in the Fiducial model, so prescription robustness is conditional on those omissions.

## Evidence

- [discussion] using the ‘rapid' model of Fryer et al.38 does not significantly alter the typical evolutionary pathways for forming heavy BBHs: "using the ‘rapid' model of Fryer et al.38 does not significantly alter the typical evolutionary pathways for forming heavy BBHs"
- [discussion] the primary mass m1 drawn from the Kroupa IMF31 up to m1≤100M⊙: [evidence span withheld — anchor status: not_found]
- [discussion] using the Kroupa31 initial mass function (IMF), a uniform mass ratio distribution and assuming that all stars are in binaries: "using the Kroupa31 initial mass function (IMF), a uniform mass ratio distribution and assuming that all stars are in binaries"
- [readme] COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants.: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants."
- [readme] Simulations in this paper made use of the COMPAS rapid binary population synthesis code (version X.X.X), which is freely available at http://github.com/TeamCOMPAS/COMPAS_.: "Simulations in this paper made use of the COMPAS rapid binary population synthesis code (version X.X.X), which is freely available at http://github.com/TeamCOMPAS/COMPAS_."
