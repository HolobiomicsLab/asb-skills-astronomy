---
name: survey-classification-merging-rules
description: Decision support for assigning suggested quality-of-match (QOP) classes
  to automatic redshift matches from autoz-style cross-correlation peaks, or for deciding
  which classification disagreements require manual merging.
when_to_use_negative:
- Only a single correlation peak exists with no second peak v2 (the modified xcor
  normalisation sets a normalisation factor of 1 instead of NaN, but the FOM ratio
  v1/v2 is undefined in a meaningful way)
- The spectra were not redshifted via the autoz cross-correlation matcher, so v1/v2
  peak heights on the Marz normalised scale are unavailable
- The goal is manual redshifting by template comparison or feature marking rather
  than classifying automatic matches
tools:
- name: Marz
  role: Implements the modified autoz cross-correlation matcher, computes the FOM
    and assigns suggested QOP values; merging logic follows the manual OzDES algorithm
  repo: https://github.com/samreay/Marz
- name: autoz
  role: Underlying cross-correlation algorithm (modified) whose normalised peak heights
    v1 and v2 feed the FOM and QOP thresholds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:c04d7016381f39f9f29b9bc9cc82509950906f72a23c9e2de35c1fbbed2b5222
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# survey-classification-merging-rules

## Summary

Applies the Marz Figure-of-Merit quality classification, which converts cross-correlation peak strengths (v1, v2) into a suggested QOP class via the threshold ladder of Eq. 5, and encodes the OzDES merging rule that any QOP 3 vs QOP 4 disagreement must be merged.

## When to use

Decision support for assigning suggested quality-of-match (QOP) classes to automatic redshift matches from autoz-style cross-correlation peaks, or for deciding which classification disagreements require manual merging.

## When NOT to use

- Only a single correlation peak exists with no second peak v2 (the modified xcor normalisation sets a normalisation factor of 1 instead of NaN, but the FOM ratio v1/v2 is undefined in a meaningful way)
- The spectra were not redshifted via the autoz cross-correlation matcher, so v1/v2 peak heights on the Marz normalised scale are unavailable
- The goal is manual redshifting by template comparison or feature marking rather than classifying automatic matches

## Inputs

- Correlation peak pairs (v1, v2): normalised autoz-style peak heights from the Marz cross-correlation matcher
- Indicator of whether the best match is to a stellar template (controls the QOP6 branch)
- FITS spectra from the AAOmega spectrograph (or any spectrograph following Marz's input format) processed through the matching stage

## Outputs

- Suggested QOP class (QOP1–QOP6) per spectrum/match
- Table of every (v1, v2) pair with its assigned QOP (e.g. qop_assignments.csv)
- Flags marking QOP 3 / QOP 4 disagreements as requiring merging

## How to apply

Take the two highest normalised cross-correlation peak heights (v1 = best peak, v2 = second peak) emitted by the modified autoz matcher in Marz and compute the Figure of Merit FOM = (v1 - 2.5)^0.75 * v1/v2. Map FOM to a suggested quality operator using the Eq. 5 threshold ladder: QOP6 (stellar template fit) if FOM > 4.5 for stellar peaks, QOP4 if FOM > 8.5, QOP3 if FOM > 4.5, QOP2 if FOP threshold FOM > 3, and QOP1 otherwise. For each pair, record the assigned QOP alongside (v1, v2) so the ordering QOP4 > QOP3 > QOP2 > QOP1 can be verified. Apply the OzDES merging rule introduced in Marz 1.3.8: when classifiers (or automatic vs manual runs) disagree such that one assigns QOP 3 and another QOP 4, flag the object as requiring merging rather than accepting either label. The rationale is that the FOM folds both absolute peak strength and ratio-to-runner-up into one statistic, so the thresholds encode both match confidence and uniqueness of the correlation peak.

## Related tools

- **Marz** (Implements the modified autoz cross-correlation matcher, computes the FOM and assigns suggested QOP values; merging logic follows the manual OzDES algorithm) — https://github.com/samreay/Marz
- **autoz** (Underlying cross-correlation algorithm (modified) whose normalised peak heights v1 and v2 feed the FOM and QOP thresholds)

## Examples

```
./marz.sh <FITSFilePath>
```

## Evaluation signals

- Every emitted QOP is consistent with the threshold ordering QOP4 > QOP3 > QOP2 > QOP1 for the FOM cutoffs 8.5, 4.5, and 3
- Pairs where FOM > 4.5 and the best fit is a stellar template are assigned QOP6, not QOP3
- All QOP 3 vs QOP 4 disagreements are flagged as requiring merging, matching the 1.3.8 rule
- Recomputing FOM = (v1 - 2.5)^0.75 * v1/v2 from the tabulated (v1, v2) values reproduces the assigned class for each row
- Objects with only one correlation peak do not yield NaN classifications (normalisation factor of 1 applied)

## Limitations

- The FOM thresholds are calibrated for OzDES/2dFLenS spectra from the AAOmega spectrograph; other instruments or template sets may need recalibration
- The classifier only produces a *suggested* QOP; spectra not matched correctly still require manual redshifting (cycling automatic results, manual template comparison, or marking spectral features)
- Earlier xcor normalisation returned NaN for spectra with only one peak, an edge case fixed by setting the normalisation factor to 1

## Evidence

- [abstract] Automatic matching performance context: [evidence span withheld — anchor status: not_found]
