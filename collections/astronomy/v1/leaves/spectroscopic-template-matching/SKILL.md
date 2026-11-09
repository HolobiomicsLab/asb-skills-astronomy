---
name: spectroscopic-template-matching
description: Use when FITS spectra from a fibre-fed spectrograph (e.g. AAOmega/2dF)
  need automated redshifts and per-match confidence labels, or when reconstructing/QA-ing
  an autoz-style cross-correlation matcher and its QOP decision boundaries.
when_to_use_negative:
- The spectrum is not a FITS file in the well-defined input format Marz expects (e.g.
  already-reduced catalogues or non-spectral tables)
- The task requires per-object human classification/QOP verification rather than bulk
  automatic matching — Marz's automatic path is meant to be supplemented by manual
  redshifting for unmatched spectra
- A redshifting pipeline with different template sets or FOM conventions is already
  validated for the survey; QOP thresholds (4.5, 8.5, 3) are Marz/OzDES-specific
tools:
- name: Marz
  role: Web application and CLI that runs the modified autoz cross-correlation matcher
    against stellar and galaxy templates and assigns QOP labels; run via marz.sh on
    FITS files
  repo: https://github.com/samreay/Marz
- name: autoz
  role: Underlying cross-correlation algorithm (modified) used to normalise peak heights
    and match input spectra against templates
- name: node.js
  role: Runtime required to execute the Marz command-line interface (marz.sh) for
    batch matching of FITS files
  repo: https://nodejs.org
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:553c33c780b0dc3cd250ff91ee821d8f4da88df97763cf8fc9c9c42137cc5602
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  - node.js
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# spectroscopic-template-matching

## Summary

Redshift spectra by cross-correlating them against a set of stellar and galaxy templates with a modified autoz algorithm (as implemented in Marz), then rank matches via a Figure of Merit computed from the two highest correlation peaks (v1, v2) and assign a quality operator (QOP). This replaces manual redshifting (e.g. runz) for AAOmega/2dF-style survey spectra.

## When to use

Use when FITS spectra from a fibre-fed spectrograph (e.g. AAOmega/2dF) need automated redshifts and per-match confidence labels, or when reconstructing/QA-ing an autoz-style cross-correlation matcher and its QOP decision boundaries.

## When NOT to use

- The spectrum is not a FITS file in the well-defined input format Marz expects (e.g. already-reduced catalogues or non-spectral tables)
- The task requires per-object human classification/QOP verification rather than bulk automatic matching — Marz's automatic path is meant to be supplemented by manual redshifting for unmatched spectra
- A redshifting pipeline with different template sets or FOM conventions is already validated for the survey; QOP thresholds (4.5, 8.5, 3) are Marz/OzDES-specific

## Inputs

- Generic FITS spectra (drag-and-drop single files, folders of *.fits, from AAOmega/2dF or any spectrograph with a well-defined input format)
- Template catalogue (WiggleZ and SDSS template samples, including stellar and galaxy templates)
- Correlation peak pairs (v1, v2) from the cross-correlation matcher

## Outputs

- Redshift estimates with heliocentric and CMB corrections applied
- Suggested QOP labels per match (QOP1–QOP6) derived from FOM thresholds
- Merge flags for QOP 3 / QOP 4 classification disagreements
- Match results saved to .mz files, or stdout output via the CLI

## How to apply

Cross-correlate each input spectrum against the template catalogue (Marz uses WiggleZ and SDSS template samples, including stellar templates) with the autoz-style normalised peak-height algorithm, handling the special case of a single peak by assigning a normalisation factor of 1 rather than NaN. For each spectrum take the two highest cross-correlation peak strengths (v1 best, v2 second) and compute the Figure of Merit FOM = (v1 - 2.5)^0.75 * v1/v2. Assign the suggested quality operator from the Eq. 5 threshold ordering: QOP6 (stellar) if FOM > 4.5 for a stellar template fit, QOP4 if FOM > 8.5, QOP3 if FOM > 4.5, QOP2 if FOM > 3, and QOP1 otherwise. Flag spectra whose merging-classification assignment yields a QOP 3 vs QOP 4 disagreement as requiring merging, consistent with the manual OzDES merging rule. Apply heliocentric and CMB corrections to the derived redshift before emitting results.

## Related tools

- **Marz** (Web application and CLI that runs the modified autoz cross-correlation matcher against stellar and galaxy templates and assigns QOP labels; run via marz.sh on FITS files) — https://github.com/samreay/Marz
- **autoz** (Underlying cross-correlation algorithm (modified) used to normalise peak heights and match input spectra against templates)
- **node.js** (Runtime required to execute the Marz command-line interface (marz.sh) for batch matching of FITS files) — https://nodejs.org

## Examples

```
./marz.sh <FITSFilePath>
```

## Evaluation signals

- Threshold ordering check: for every (v1, v2) pair, the assigned QOP is monotone with FOM (QOP4 if FOM > 8.5, QOP3 if FOM > 4.5, QOP2 if FOM > 3, QOP1 otherwise; QOP6 only for stellar template fits with FOM > 4.5)
- Single-peak robustness: spectra with only one correlation peak receive a normalisation factor of 1 and a finite FOM, never NaN
- Merging-flag invariant: every match whose classification yields a QOP 3 vs QOP 4 disagreement is flagged as needing merging
- Benchmark comparison: automatic matching performance on OzDES spectra should approach Marz's reported 91%, versus 54% for runz
- Output round-trip: .mz files save without error (comments with commas are stripped), and CLI output matches the interactive results

## Limitations

- Single-peak spectra previously returned NaN from xcor normalisation; this edge case requires the normalisation factor of 1 fix
- Mathematical library support for Javascript is still in its infancy, and Marz was developed without significant supporting scientific libraries
- No version numbers are stated for external libraries (AngularJS, Bootstrap, AngularUI, fitsjs), so reproducing the exact client stack is difficult
- QOP thresholds and merging rules are tuned for the OzDES manual algorithm; other surveys may need recalibration

## Evidence

- [abstract] automatic matching performance benchmark: [evidence span withheld — anchor status: not_found]
