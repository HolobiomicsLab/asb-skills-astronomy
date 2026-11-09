---
name: redshift-quality-of-match-grading
description: Use when assigning suggested quality operators (QOP) to automatic spectroscopic
  redshift matches, e.g. OzDES/AAOmega spectra scored by cross-correlation peak heights,
  or when reproducing Marz's autoz-style FOM thresholds.
when_to_use_negative:
- Spectra require manual redshifting (cycling automatic results, manual template comparison,
  or marking spectral features) rather than automatic QOP grading
- Input peak heights are not normalised autoz-style cross-correlation peak strengths,
  so the FOM = (v1 - 2.5)^0.75 * v1/v2 thresholds do not apply
- Goal is template construction or variance culling rather than grading existing automatic
  matches
edam_operation: http://edamontology.org/operation_2990
tools:
- name: Marz
  role: Hosts the modified autoz cross-correlation matcher and the FOM/QOP assignment
    logic; can grade matches via its web interface or command line
  repo: https://github.com/samreay/Marz
- name: autoz
  role: Cross-correlation algorithm (modified) that produces the peak heights (v1,
    v2) against stellar and galaxy templates that feed the FOM classifier
- name: kriskowal/q
  role: JavaScript promise library used in Marz's client-side codebase (npm dependency)
  repo: https://github.com/kriskowal/q
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:63ec51e4a77b1589eb64d1b24a50d16674a461f7a841804dff85752c6d68e88f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  - kriskowal/q
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# redshift-quality-of-match-grading

## Summary

Implements the Marz Figure of Merit (FOM) quality classifier that converts the two highest cross-correlation peak strengths (v1, v2) from the modified autoz matcher into a suggested Quality of Match (QOP 1-6) grade for automatic redshifts, using the paper's FOM thresholds (alpha=2.5, beta=0.75).

## When to use

Use when assigning suggested quality operators (QOP) to automatic spectroscopic redshift matches, e.g. OzDES/AAOmega spectra scored by cross-correlation peak heights, or when reproducing Marz's autoz-style FOM thresholds.

## When NOT to use

- Spectra require manual redshifting (cycling automatic results, manual template comparison, or marking spectral features) rather than automatic QOP grading
- Input peak heights are not normalised autoz-style cross-correlation peak strengths, so the FOM = (v1 - 2.5)^0.75 * v1/v2 thresholds do not apply
- Goal is template construction or variance culling rather than grading existing automatic matches

## Inputs

- Correlation peak pairs (v1, v2) from Marz's modified autoz cross-correlation matcher (normalised peak heights)
- Template identity of the best match (galaxy vs stellar) to resolve the QOP6 stellar case
- FITS spectra from AAOmega or any spectrograph following Marz's well-defined input format

## Outputs

- Suggested QOP (1-6) per spectrum / per (v1, v2) peak pair
- Table of every (v1, v2) pair with its assigned QOP (e.g. qop_assignments.csv)
- Merging flags for QOP 3 vs QOP 4 disagreements requiring manual merging

## How to apply

Take the normalised peak heights of the two best cross-correlation matches produced by the modified autoz algorithm (v1 = best peak, v2 = second peak); results with only one peak use a normalisation factor of 1 instead of returning NaN. For each (v1, v2) pair compute FOM = (v1 - 2.5)^0.75 * v1/v2. Assign QOP from the threshold ordering: QOP6 (stellar template match) if FOM > 4.5 with a stellar template, QOP4 if FOM > 8.5, QOP3 if FOM > 4.5, QOP2 if FOM > 3, and QOP1 otherwise. Flag pairs whose individual classifications disagree as QOP 3 vs QOP 4 as requiring merging, matching the manual OzDES merging rule implemented in Marz 1.3.8. Emit each (v1, v2) pair with its assigned QOP (e.g. to qop_assignments.csv) and verify the threshold ordering QOP4 > QOP3 > QOP2 > QOP1 is respected.

## Related tools

- **Marz** (Hosts the modified autoz cross-correlation matcher and the FOM/QOP assignment logic; can grade matches via its web interface or command line) — https://github.com/samreay/Marz
- **autoz** (Cross-correlation algorithm (modified) that produces the peak heights (v1, v2) against stellar and galaxy templates that feed the FOM classifier)
- **kriskowal/q** (JavaScript promise library used in Marz's client-side codebase (npm dependency)) — https://github.com/kriskowal/q

## Examples

```
./marz.sh path/to/spectra.fits > marz_output.txt
```

## Evaluation signals

- Emitted QOP for each (v1, v2) pair is consistent with the threshold ordering (QOP4 > QOP3 > QOP2 > QOP1) given FOM = (v1 - 2.5)^0.75 * v1/v2
- QOP6 is only assigned when FOM > 4.5 AND the best match is to a stellar template
- All QOP 3 vs QOP 4 disagreements are flagged as requiring merging, per the manual OzDES rule
- Peak pairs with only one peak use a normalisation factor of 1 rather than producing NaN
- Graded automatic matches reproduce Marz's reported behaviour, e.g. improved automatic matching on OzDES spectra relative to runz

## Limitations

- Thresholds are calibrated for Marz's modified autoz normalisation of AAOmega/OzDES spectra; other peak-height conventions will misgrade
- QOP grades are suggested values — spectra not matched correctly by the automatic algorithm still require manual redshifting (cycling results, manual template comparison, or marking features)
- FOM-based grading is a single scalar summary of the top two peaks and inherits the sensitivity of the cross-correlation to noise and template coverage (Marz uses WiggleZ and SDSS template samples)

## Evidence

- [other] QOP thresholds and FOM formula from task card: [evidence span withheld — anchor status: not_found]
