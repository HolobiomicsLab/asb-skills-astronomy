---
name: cross-correlation-peak-quality-assignment
description: Use when you have ranked cross-correlation peak heights (v1, v2) from
  an autoz-style matcher and need to convert them into discrete match-quality classes
  (QOP1–QOP6) or flag matches needing manual merging.
when_to_use_negative:
- Spectra with only one correlation peak handled by a different path — apply the special
  normalisation factor of 1, not the standard FOM peak-ratio logic
- Manual redshifting tasks where the user cycles automatic results, compares templates
  manually, or marks spectral features instead of relying on automatic QOPs
- Redshift fitting against templates or instruments outside the autoz/Marz normalisation
  scheme, where peak heights are not autoz-style normalised
tools:
- name: Marz
  role: Provides the autoz-based cross-correlation matcher whose peak heights feed
    the FOM/QOP classifier; runnable interactively or via the command line
  repo: https://github.com/samreay/Marz
- name: autoz
  role: Modified cross-correlation algorithm inside Marz that normalises peak heights
    (v1, v2) used by the FOM thresholds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:b10978ba6bf1782e0b4c1b38d06e80321abce80a260909352f83c94b50ac52fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-correlation-peak-quality-assignment

## Summary

Assigns a suggested quality operator (QOP) to automatic spectral matches by computing Marz's Figure of Merit (FOM = (v1 - 2.5)^0.75 * v1/v2) from the two highest normalised cross-correlation peak heights, then thresholding FOM into QOP classes.

## When to use

Use when you have ranked cross-correlation peak heights (v1, v2) from an autoz-style matcher and need to convert them into discrete match-quality classes (QOP1–QOP6) or flag matches needing manual merging.

## When NOT to use

- Spectra with only one correlation peak handled by a different path — apply the special normalisation factor of 1, not the standard FOM peak-ratio logic
- Manual redshifting tasks where the user cycles automatic results, compares templates manually, or marks spectral features instead of relying on automatic QOPs
- Redshift fitting against templates or instruments outside the autoz/Marz normalisation scheme, where peak heights are not autoz-style normalised

## Inputs

- Ranked cross-correlation peak heights (v1, v2) from autoz-style normalised matching of spectra against Marz's WiggleZ/SDSS stellar and galaxy templates
- FITS spectra files in the Marz well-defined input format (AAOmega or generic)

## Outputs

- Suggested QOP (1–6) per automatic match
- FOM value per (v1, v2) peak pair
- Flags marking QOP 3 vs QOP 4 disagreements as requiring merging
- Table of every (v1, v2) pair with its assigned QOP (e.g. qop_assignments.csv)

## How to apply

Load the correlation peak pairs produced by the reconstructed Marz cross-correlation matcher, which uses a modified autoz algorithm with normalised peak heights. For each (v1, v2) pair compute the Figure of Merit FOM = (v1 - 2.5)^0.75 * v1/v2, where v1 is the best peak height and v2 the second peak. Assign QOPs via the Eq. 5 thresholds: QOP6 (stellar template) if FOM > 4.5, QOP4 if FOM > 8.5, QOP3 if FOM > 4.5, QOP2 if FOM > 3, and QOP1 otherwise. Flag pairs where a QOP 3 / QOP 4 disagreement occurs as requiring merging, matching the manual OzDES rule adopted in Marz v1.3.1+. Handle single-peak spectra by using a normalisation factor of 1 rather than NaN, per the xcor normalisation update.

## Related tools

- **Marz** (Provides the autoz-based cross-correlation matcher whose peak heights feed the FOM/QOP classifier; runnable interactively or via the command line) — https://github.com/samreay/Marz
- **autoz** (Modified cross-correlation algorithm inside Marz that normalises peak heights (v1, v2) used by the FOM thresholds)

## Examples

```
./marz.sh spectra.fits
```

## Evaluation signals

- Every (v1, v2) pair's emitted QOP matches the threshold ordering QOP4 > QOP3 > QOP2 > QOP1 for FOM = (v1-2.5)^0.75 * v1/v2
- Stellar-template matches with FOM > 4.5 receive QOP6, not QOP3
- All QOP 3 / QOP 4 disagreements are flagged for merging, consistent with the manual OzDES rule
- Single-peak spectra produce a normalisation factor of 1 and a finite FOM, never NaN
- Boundary checks: FOM exactly at 3, 4.5, and 8.5 falls to the lower QOP class (strict > comparisons)

## Limitations

- QOP thresholds are calibrated for OzDES/2dFLenS AAOmega spectra; performance on other instruments' data is unvalidated
- The merging rule (any QOP 3 and QOP 4 disagreement needs merging) only applies from Marz v1.3.8 onward; earlier versions used a different merging algorithm
- Prior to the xcor normalisation fix, results with only one peak returned NaN instead of a normalisation factor of 1
- QOP is a suggested quality only — Marz's own workflow still expects manual checking, correction, or verification by the user

## Evidence

- [abstract] autoz modified cross-correlation: "Behind the scenes, a modified version of the autoz cross-correlation algorithm is used to match input spectra against a variety of stellar and galaxy templates"
- [methods] xcor normalisation single-peak fix: "Update to xcor normalisation, such that results with only one peak have a normalisation factor of 1, instead of returning NaN."
- [readme] QOP 3/QOP 4 merging rule: "Updating merging so that any QOP 3 and QOP 4 disagreement needs merging."
- [readme] OzDES merging alignment: "Updating merging algorithm to match manual OzDES algorithm."
