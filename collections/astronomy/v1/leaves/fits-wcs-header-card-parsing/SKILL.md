---
name: fits-wcs-header-card-parsing
description: Use when a FITS file's header cards (CTYPE/CRVAL/CRPIX-type WCS cards)
  must be converted into an AST FrameSet, and the consumed cards must be verified
  as removed from the residual header buffer.
when_to_use_negative:
- The coordinate system description is not stored as FITS header cards (e.g. it already
  exists as an AST object or Channel text), so no FitsChan destructive read is needed
- The header contains WCS cards that AST cannot interpret — these remain unconsumed
  in the FitsChan, so a successful parse of the full WCS cannot be assumed from a
  non-null read result alone
- You only need to preserve or modify raw header cards without interpreting them as
  coordinate systems
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Starlink AST
  role: Provides the FitsChan class whose astRead method destructively parses FITS-WCS
    header cards into a FrameSet, and the Channel class for serialising the FrameSet
  repo: Starlink/starlink
- name: Python
  role: Language binding used to construct the FitsChan and perform the destructive
    read
- name: C
  role: Core implementation language of the AST library executing the FITS-WCS card
    parsing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:3a39f739cb7a9c6fe73f61ed3ccf15c2f005225c08b1ea7e2244a58acb64162d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - C
  - Python
  - Starlink AST
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fits-wcs-header-card-parsing

## Summary

Parses a buffer of FITS header cards encoding FITS-WCS information into an AST FrameSet using the Starlink AST FitsChan's destructive astRead method. The resulting FrameSet maps pixel coordinates (base Frame) to the FITS primary WCS frame (current Frame).

## When to use

Use when a FITS file's header cards (CTYPE/CRVAL/CRPIX-type WCS cards) must be converted into an AST FrameSet, and the consumed cards must be verified as removed from the residual header buffer.

## When NOT to use

- The coordinate system description is not stored as FITS header cards (e.g. it already exists as an AST object or Channel text), so no FitsChan destructive read is needed
- The header contains WCS cards that AST cannot interpret — these remain unconsumed in the FitsChan, so a successful parse of the full WCS cannot be assumed from a non-null read result alone
- You only need to preserve or modify raw header cards without interpreting them as coordinate systems

## Inputs

- Buffer or list of FITS header cards encoding WCS information (CTYPE, CRVAL, CRPIX-type cards)
- FITS-WCS encoding designation for the FitsChan

## Outputs

- AST FrameSet with base Frame = pixel coordinates and current Frame = FITS primary WCS frame
- Residual FitsChan header buffer (WCS cards consumed/removed)
- Text serialisation of the FrameSet via an AST Channel

## How to apply

Construct an AST FitsChan (via the Python bindings to the C library) and feed it the buffer of FITS header cards, using the FITS-WCS encoding. Perform the destructive read (astRead), which parses the FrameSet described by the cards and then deletes all relevant WCS cards from the FitsChan buffer. Inspect the returned FrameSet and set/check its Frame selection: the base Frame must represent pixel coordinates and the current Frame the FITS primary WCS representation. Confirm destructiveness by checking that no CTYPE/CRVAL/CRPIX-type cards remain in the FitsChan. Finally serialise the FrameSet to a text file through an AST Channel and write the residual header buffer to a second file for diff-based validation.

## Related tools

- **Starlink AST** (Provides the FitsChan class whose astRead method destructively parses FITS-WCS header cards into a FrameSet, and the Channel class for serialising the FrameSet) — Starlink/starlink
- **Python** (Language binding used to construct the FitsChan and perform the destructive read)
- **C** (Core implementation language of the AST library executing the FITS-WCS card parsing)

## Examples

```
import starlink.ast as ast; fitschan = ast.FitsChan(cards=header_cards, encoding='FITS-WCS'); frameset = fitschan.read()  # base Frame = pixel coords, current Frame = FITS primary WCS
```

## Evaluation signals

- The returned FrameSet has its base Frame set to pixel coordinates
- The returned FrameSet has its current Frame set to the FITS primary WCS frame
- The residual FitsChan buffer contains none of the WCS cards supplied on input (e.g. no CTYPE/CRVAL/CRPIX-type cards)
- The FrameSet serialises to a readable text file via an AST Channel without errors
- A diff of the residual header buffer against the input cards shows only the WCS cards removed

## Limitations

- The FitsChan read is destructive: WCS cards are deleted from the buffer once consumed, so the original header must be retained elsewhere if re-use is needed
- Cards that AST cannot interpret into the FrameSet remain in the FitsChan, so an incomplete read still leaves a non-empty buffer
- Behaviour is specific to the FITS-WCS encoding; other encodings of WCS information are not covered by this skill

## Evidence
