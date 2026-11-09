---
name: compact-object-merger-time-evaluation
description: Use when a COMPAS (or BSE-style) population synthesis run has produced
  double compact objects with component masses and separations, and you must compute
  GW merger times and select merging BBHs within a Hubble time.
when_to_use_negative:
- When an accurate LIGO merger-rate comparison is required that convolves the metallicity-specific
  redshift-dependent star formation rate with the time delay distribution — the paper
  explicitly did not perform this convolution, so its ~300 Gpc^-3/yr figure is approximate
- When the formation channel depends on stellar rotation, tides, or wind accretion
  onto the companion — the Fiducial model excludes all three, so merger-time-selected
  yields from this setup will not capture those physics
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to evolve progenitor binaries
    to double compact objects and evaluate which merge within a Hubble time under
    the Fiducial model
  repo: https://github.com/TeamCOMPAS/COMPAS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:e99b4d9dd6b8524983c88fc8f2b9706ab51bcd3dfd324f05dd355dada4594839
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamCOMPAS/COMPAS
  tools:
  - COMPAS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# compact-object-merger-time-evaluation

## Summary

Computes the gravitational-wave inspiral merger time for every double compact object produced by COMPAS binary population synthesis and retains only binary black holes that merge within a Hubble time, yielding per-metallicity merging-BBH yields and rescaled volumetric merger rates (as done to reproduce GW150914, GW151226 and LVT151012).

## When to use

Use when a COMPAS (or BSE-style) population synthesis run has produced double compact objects with component masses and separations, and you must compute GW merger times and select merging BBHs within a Hubble time.

## When NOT to use

- When an accurate LIGO merger-rate comparison is required that convolves the metallicity-specific redshift-dependent star formation rate with the time delay distribution — the paper explicitly did not perform this convolution, so its ~300 Gpc^-3/yr figure is approximate
- When the formation channel depends on stellar rotation, tides, or wind accretion onto the companion — the Fiducial model excludes all three, so merger-time-selected yields from this setup will not capture those physics

## Inputs

- COMPAS binary population evolved to the double compact object stage at Z=0.002 and Z=0.001 (component masses, final separations, remnant types)
- Initial population sampling settings: Kroupa IMF primaries up to 100 Msun, uniform mass-ratio distribution, flat-in-the-log semimajor-axis distribution, zero initial eccentricity, all stars in binaries

## Outputs

- List of double compact objects with computed gravitational-wave merger times, filtered to BBHs merging within a Hubble time
- Per-metallicity merging-BBH yields per 1e9 solar masses of star formation (e.g. bbh_merger_rates.csv)
- Rescaled volumetric BBH merger rate at z=0 (Gpc^-3 per year)

## How to apply

Evolve the initial binary population with COMPAS under the Fiducial physics — delayed Fryer supernova model (alpha=1), common-envelope alpha_lambda=0.1, fLBV=1.5, Vink winds, no stellar rotation, tides or wind accretion — drawing primaries from the Kroupa IMF up to 100 Msun, flat mass ratio, flat-in-the-log semimajor axis, zero eccentricity, at metallicities of 10% and 5% solar (Z=0.002 and Z=0.001). For each double compact object, compute the gravitational-wave merger time from the compact-object masses and final separation, then select only binary black holes with merger times shorter than a Hubble time. Normalize the count of merging BBHs per 1e9 solar masses of star formation at each metallicity and rescale to a volumetric BBH merger rate at z=0, writing per-metallicity yields (e.g. to bbh_merger_rates.csv). Validate against the fiducial benchmarks of ~2e4 merging BBHs per 1e9 Msun at 10% solar metallicity, ~3e4 at 5%, and a rescaled rate of ~300 Gpc^-3 per year, against the LIGO empirical estimate of 9-240 Gpc^-3 per year.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to evolve progenitor binaries to double compact objects and evaluate which merge within a Hubble time under the Fiducial model) — https://github.com/TeamCOMPAS/COMPAS

## Evaluation signals

- Merging-BBH yield per 1e9 solar masses of star formation is ~2e4 at 10% solar metallicity and ~3e4 at 5% solar metallicity
- Rescaled rate is consistent with ~300 Gpc^-3 per year, comparable to the LIGO empirical estimate of 9-240 Gpc^-3 per year
- At Z=0.001, all three events (GW150914, GW151226, LVT151012) are reproduced from progenitor total masses ≳160, ≳60 and ≳90 Msun respectively
- Output table contains one yield entry per simulated metallicity and counts only BBHs with merger time below a Hubble time
- Toggling to the 'rapid' Fryer supernova model does not significantly change the typical pathways for forming heavy BBHs, serving as a robustness check

## Limitations

- The merger-rate comparison to LIGO is approximate; an accurate rate requires the convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution, which was not done
- The single-star (SSE) model grid extends only to 50 solar masses and is extrapolated above that limit, affecting the most massive progenitors
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model
- Only individual parameters (e.g. alpha_lambda from 0.1 to 0.01, fLBV from 1.5e-4 to 1e-3 Msun/yr, rapid vs delayed Fryer) were varied independently; full exploration of the multidimensional parameter space was deferred to future studies

## Evidence

- [other] merger-time selection of double compact objects: [evidence span withheld — anchor status: not_found]
- [discussion] fiducial yield benchmark: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [abstract] progenitor masses at low metallicity: "We show all three events could have formed in low-metallicity environments ( Z =0.001) from progenitor binaries with typical total masses ≳160 M ⊙"
- [discussion] rate-convolution caveat: "An accurate calculation of the merger rate requires the convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution"
- [readme] COMPAS scope: "It has been used for inference from observations of gravitational-wave mergers, Galactic neutron stars, X-ray binaries, and luminous red novae."
