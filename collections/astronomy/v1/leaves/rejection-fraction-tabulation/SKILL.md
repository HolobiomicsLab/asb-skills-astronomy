---
name: rejection-fraction-tabulation
description: Use when you have per-star rejection tables from a BeyonCE gradient-rejection
  run over azimuthally symmetric disc geometries and need to quantify, per star, what
  fraction of mapped disc orientations remain viable.
when_to_use_negative:
- The disc optical-depth profile is not azimuthally symmetric — the gradient-rejection
  test and its fraction estimates are derived for azimuthally symmetric, non-uniform
  profiles
- No gradient-rejection analysis has yet been run (i.e. only raw light curves exist
  and disc geometries have not been mapped and tested)
- The system's eclipse is attributed to a circular, transiting body rather than a
  circumsecondary disc transit such as those hypothesised for J1407 and PDS 110
edam_operation: http://edamontology.org/operation_2945
tools:
- name: BeyonCE
  role: Produces the mapped disc geometries and per-configuration gradient accept/reject
    results that this skill tabulates into rejected/retained fractions
  repo: github.com/dmvandamt/beyonce
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:b2eb8987489498cf143e879d0674b8ebb2c20a8afb70bd962e45705ae16a465a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BeyonCE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# rejection-fraction-tabulation

## Summary

Tabulates how much of a circumsecondary disc (CSD) parameter space is eliminated by BeyonCE's gradient-rejection test: for each star (e.g. J1407, PDS 110), it counts total mapped disc geometries versus configurations rejected where the measured light-curve gradient exceeds the theoretical gradient, and reports retained and rejected fractions per star.

## When to use

Use when you have per-star rejection tables from a BeyonCE gradient-rejection run over azimuthally symmetric disc geometries and need to quantify, per star, what fraction of mapped disc orientations remain viable.

## When NOT to use

- The disc optical-depth profile is not azimuthally symmetric — the gradient-rejection test and its fraction estimates are derived for azimuthally symmetric, non-uniform profiles
- No gradient-rejection analysis has yet been run (i.e. only raw light curves exist and disc geometries have not been mapped and tested)
- The system's eclipse is attributed to a circular, transiting body rather than a circumsecondary disc transit such as those hypothesised for J1407 and PDS 110

## Inputs

- Per-star BeyonCE rejection tables (J1407, PDS 110) from gradient-rejection analysis, with per-configuration accept/reject status and disc orientation
- Count of total mapped disc geometries per star

## Outputs

- parameter_space_reduction_summary.csv with one row per star and columns: star, total configurations, retained fraction, rejected fraction

## How to apply

Load the per-star rejection tables for J1407 and PDS 110 produced by BeyonCE's gradient-rejection analysis. For each star, count the total number of mapped disc geometries and the subset rejected where the measured light-curve gradient is greater than the theoretical gradient for the given disc orientation. Compute the rejected fraction (rejected / total) and the retained fraction (1 − rejected fraction) per star. Assemble these into a summary table with one row per star and columns: star, total configurations, retained fraction, rejected fraction. Write the table to parameter_space_reduction_summary.csv; the rationale is that the ratio of rejected to mapped geometries directly measures how effectively the gradient test condenses the CSD parameter space, as demonstrated for azimuthally symmetric, non-uniform optical-depth profiles.

## Related tools

- **BeyonCE** (Produces the mapped disc geometries and per-configuration gradient accept/reject results that this skill tabulates into rejected/retained fractions) — github.com/dmvandamt/beyonce

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('rejection_table_J1407.csv'); frac=t.groupby('star').agg(total=('rejected','size'), rejected=('rejected','sum')); frac['rejected_fraction']=frac.rejected/frac.total; frac['retained_fraction']=1-frac.rejected_fraction; frac.reset_index().to_csv('parameter_space_reduction_summary.csv', index=False)"
```

## Evaluation signals

- Each star row reports retained_fraction + rejected_fraction == 1 (within floating-point tolerance)
- total configurations equals the count of mapped disc geometries for that star, and rejected count ≤ total
- Summary CSV has exactly the columns star, total configurations, retained fraction, rejected fraction, one row per star analysed (J1407, PDS 110)
- Rejected fractions are in [0, 1]; a rejected fraction near 0 would contradict the reported 'considerable' parameter-space reduction and warrants re-checking the gradient comparison

## Limitations

- Fractions are only meaningful for discs with azimuthally symmetric, non-uniform optical-depth profiles; extension to other disc structures is not established
- The test compares measured light-curve gradients to theoretical gradients per orientation, so gradient measurement noise in the light curve can misclassify configurations as rejected or retained
- Results are specific to the analysed systems (J1407, PDS 110) whose light curves are attributed to CSD transits; applicability to other eclipsers is unverified

## Evidence

- [abstract] gradient rejection rule: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
- [abstract] mapping of disc geometries: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] effectiveness of parameter-space reduction: "demonstrate its effectiveness in considerably reducing the parameter space of discs that contain an azimuthally symmetric structure"
- [abstract] applied systems: "We used the code to analyse the light curves seen towards J1407 and PDS 110, attributed to CSD transits"
