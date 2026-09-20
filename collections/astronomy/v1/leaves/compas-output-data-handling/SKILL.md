---
name: compas-output-data-handling
description: Use when the task requires extracting progenitor binary properties (e.g.
  ZAMS total mass m1+m2) from COMPAS population-synthesis output files by filtering
  systems that merge within a Hubble time and match the 90% credible intervals of
  a specific GW event.
when_to_use_negative:
- 'The task needs accurate cosmic merger rates: the paper notes an accurate calculation
  requires convolution of metallicity-specific redshift-dependent star formation rate
  with the time-delay distribution, which was not done in this dataset.'
- The analysis targets non-BBH outcomes (e.g. Galactic neutron stars, X-ray binaries)
  or non-Fiducial COMPAS model variations; this skill is built around the fiducial-model
  BBH output at Z=0.002 and Z=0.001.
- The event of interest is not GW150914, GW151226 or LVT151012, since the validation
  thresholds are specific to those three events' posteriors.
edam_operation: http://edamontology.org/operation_2422
tools:
- name: COMPAS
  role: Source of the fiducial double compact object population output (Z=0.002 and
    Z=0.001) whose binary black hole systems are filtered with the Peters-equation
    merger-time prescription; output data published via the COMPAS Zenodo community
  repo: http://github.com/TeamCOMPAS/COMPAS_
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:249abf130736120cd9537212e61a9ff60b28e33d291baee05ad95c36bd08121f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# compas-output-data-handling

## Summary

Loads fiducial COMPAS double compact object population output at Z=0.002 and Z=0.001 from the COMPAS Zenodo community deposit, filters merging binary black holes, and tabulates zero-age-main-sequence progenitor total masses matched to observed gravitational-wave events. It exists to reproduce the paper's progenitor initial-mass ranges for GW150914, GW151226 and LVT151012.

## When to use

Use when the task requires extracting progenitor binary properties (e.g. ZAMS total mass m1+m2) from COMPAS population-synthesis output files by filtering systems that merge within a Hubble time and match the 90% credible intervals of a specific GW event.

## When NOT to use

- The task needs accurate cosmic merger rates: the paper notes an accurate calculation requires convolution of metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which was not done in this dataset.
- The analysis targets non-BBH outcomes (e.g. Galactic neutron stars, X-ray binaries) or non-Fiducial COMPAS model variations; this skill is built around the fiducial-model BBH output at Z=0.002 and Z=0.001.
- The event of interest is not GW150914, GW151226 or LVT151012, since the validation thresholds are specific to those three events' posteriors.

## Inputs

- Fiducial-model COMPAS double compact object population output files at Z=0.002 and Z=0.001 (from the COMPAS Zenodo community deposit)
- 90% credible intervals on chirp mass, component masses and mass ratio for GW150914, GW151226 and LVT151012
- Peters-equation merger-time parameters (component masses, orbital separation/separation-derived quantities) to select systems merging within a Hubble time

## Outputs

- pandas DataFrame of minimum and typical initial total masses (m1+m2 at ZAMS) per event per metallicity
- progenitor_initial_masses.csv containing the per-event, per-metallicity table

## How to apply

Load the fiducial-model COMPAS double compact object output at Z=0.002 and Z=0.001 from the COMPAS Zenodo community deposit. Build the event-selection sample by keeping only binary black holes that merge within a Hubble time, using the COMPAS merger-time prescription based on the Peters equations. For GW150914, GW151226 and LVT151012, retain progenitor systems whose chirp mass and component masses fall within the observed 90% credible intervals (also constrained by mass ratio). For each selected progenitor compute the initial total mass as the sum of the two zero-age-main-sequence masses (m1+m2), tabulate the minimum and typical values per event per metallicity in a pandas DataFrame, and write the table to progenitor_initial_masses.csv. The fiducial model draws primary masses from the Kroupa IMF up to 100 Msun with a flat mass-ratio distribution and zero initial eccentricity, so quoted masses are ZAMS values from that initial-condition set.

## Related tools

- **COMPAS** (Source of the fiducial double compact object population output (Z=0.002 and Z=0.001) whose binary black hole systems are filtered with the Peters-equation merger-time prescription; output data published via the COMPAS Zenodo community) — http://github.com/TeamCOMPAS/COMPAS_

## Examples

```
df = pd.read_hdf('COMPAS_DCO_output.h5', key='DoubleCompactObjects'); bbh = df[df.merges_within_Hubble_time & (df.m1ZAMS+df.m2ZAMS>=60)]; bbh['M_ZAMS_total']=bbh.m1ZAMS+bbh.m2ZAMS; bbh.to_csv('progenitor_initial_masses.csv')
```

## Evaluation signals

- Minimum initial total masses at Z=0.001 are at least approximately 160, 60 and 90 solar masses for GW150914, GW151226 and LVT151012 respectively.
- Z=0.002 initial total masses fall within the reported ranges of 65-100 solar masses (GW150914) and 95-125 solar masses (LVT151012).
- Every system in the event-selection sample satisfies the merging-within-a-Hubble-time criterion under the COMPAS/Peters merger-time prescription.
- Retained progenitors have chirp mass and component masses inside the 90% credible intervals of the corresponding LIGO event.
- progenitor_initial_masses.csv is written with one row per event per metallicity and contains both minimum and typical m1+m2 columns.

## Limitations

- Only the Fiducial model output is used; the paper varied individual parameters (common-envelope alpha, LBV wind mass-loss rate, Fryer SN engine) independently and left full exploration of the multidimensional parameter space to future studies.
- The fiducial model excludes stellar rotation and tides, wind accretion by a companion, and extrapolates the SSE single-star grid above 50 solar masses, which affects the heaviest progenitors.
- The README's methods citation uses a version placeholder ('version X.X.X'), so 2017 fiducial output from the Zenodo deposit may differ in format and prescriptions from current COMPAS releases.

## Evidence

- [abstract] all three events could have formed in low-metallicity environments ( Z =0.001) from progenitor binaries with typical total masses ≳160 M ⊙ , ≳60 M ⊙ and ≳90 M ⊙: "all three events could have formed in low-metallicity environments ( Z =0.001) from progenitor binaries with typical total masses ≳160 M ⊙ , ≳60 M ⊙ and ≳90 M ⊙"
- [methods] Simulations in this paper made use of the COMPAS rapid binary population synthesis code (version X.X.X), which is freely available at http://github.com/TeamCOMPAS/COMPAS_: "Simulations in this paper made use of the COMPAS rapid binary population synthesis code (version X.X.X), which is freely available at http://github.com/TeamCOMPAS/COMPAS_"
- [discussion] using the Kroupa31 initial mass function (IMF), a uniform mass ratio distribution and assuming that all stars are in binaries: "using the Kroupa31 initial mass function (IMF), a uniform mass ratio distribution and assuming that all stars are in binaries"
- [readme] make your input settings and output data publicly available by uploading it to the [COMPAS Zenodo community](https://zenodo.org/communities/compas/): "make your input settings and output data publicly available by uploading it to the [COMPAS Zenodo community](https://zenodo.org/communities/compas/)"
