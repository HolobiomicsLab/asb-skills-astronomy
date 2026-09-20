---
name: fitschan-destructive-read-verification
description: Use when a buffer of FITS header cards encoding WCS information must
  be converted into an AST FrameSet (e.g. from a FITS file's WCS representation),
  and the destructive-read semantics of the FitsChan need to be exercised and verified.
when_to_use_negative:
- Input WCS information is not encoded as FITS header cards (e.g. it is already an
  AST object or a non-FITS-WCS format), since the FitsChan read consumes FITS-WCS-encoded
  cards
- The header cards must be preserved intact for downstream reuse, because the astRead
  read is destructive and deletes the relevant FITS cards from the buffer
- Only a coordinate transformation needs to be evaluated numerically without round-tripping
  through FITS-WCS header card representation
tools:
- name: Starlink AST
  role: Provides the FitsChan (astRead destructive read), FrameSet, and Channel objects
    used to parse FITS-WCS headers into coordinate-system Frames
- name: Python
  role: Language binding through which the AST FitsChan read and FrameSet serialisation
    are invoked
- name: C
  role: Native implementation language of the AST library providing the FitsChan/astRead
    API
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:c4f1dc1732b7b4dbebbd20be8f65d2c6afdda7db22f992d41b7a7fe9fdd7bf72
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C
  - Python
  - Starlink AST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# fitschan-destructive-read-verification

## Summary

Reconstructs a Starlink AST FrameSet from FITS-WCS header cards by performing a destructive read (astRead) on an AST FitsChan, and verifies that the returned FrameSet has its base Frame set to pixel coordinates, its current Frame set to the FITS primary WCS frame, and that the consumed WCS cards (CTYPE/CRVAL/CRPIX-type) have been removed from the FitsChan buffer.

## When to use

Use when a buffer of FITS header cards encoding WCS information must be converted into an AST FrameSet (e.g. from a FITS file's WCS representation), and the destructive-read semantics of the FitsChan need to be exercised and verified.

## When NOT to use

- Input WCS information is not encoded as FITS header cards (e.g. it is already an AST object or a non-FITS-WCS format), since the FitsChan read consumes FITS-WCS-encoded cards
- The header cards must be preserved intact for downstream reuse, because the astRead read is destructive and deletes the relevant FITS cards from the buffer
- Only a coordinate transformation needs to be evaluated numerically without round-tripping through FITS-WCS header card representation

## Inputs

- Buffer of FITS header cards encoding WCS information (FITS-WCS encoding)
- FITS-WCS cards such as CTYPE/CRVAL/CRPIX-type entries
- Starlink AST FitsChan object constructed via the AST Python (or C) bindings

## Outputs

- AST FrameSet with base Frame = pixel coordinates and current Frame = FITS primary WCS frame
- Text serialisation of the FrameSet written via an AST Channel
- Residual header buffer file containing none of the supplied WCS cards

## How to apply

Construct an AST FitsChan in Python (via the AST bindings) and feed it the buffer of FITS header cards using the FITS-WCS encoding. Perform the destructive read (astRead) to obtain a FrameSet encoding the FITS primary WCS representation. Verify the returned FrameSet has its base Frame set to pixel coordinates and its current Frame set to the FITS primary WCS frame. Confirm the read was destructive by inspecting the FitsChan for the consumed WCS cards (e.g. CTYPE/CRVAL/CRPIX-type cards) and checking they have been removed from the buffer. Finally serialise the FrameSet to a text file via an AST Channel and write the residual header buffer to a second file.

## Related tools

- **Starlink AST** (Provides the FitsChan (astRead destructive read), FrameSet, and Channel objects used to parse FITS-WCS headers into coordinate-system Frames)
- **Python** (Language binding through which the AST FitsChan read and FrameSet serialisation are invoked)
- **C** (Native implementation language of the AST library providing the FitsChan/astRead API)

## Examples

```
from starlink import ast; fitschan = ast.FitsChan(source=header_cards); frame_set = fitschan.read()
```

## Evaluation signals

- The returned FrameSet has its base Frame set to pixel coordinates
- The returned FrameSet has its current Frame set to the FITS primary WCS frame
- The residual header buffer contains none of the WCS cards supplied on input (CTYPE/CRVAL/CRPIX-type cards removed)
- The FrameSet serialises successfully to a text file via an AST Channel
- The serialised FrameSet round-trips as the FITS primary WCS representation

## Limitations

- The FitsChan Read method is destructive: after a successful read the relevant FITS cards are deleted from the buffer, so the original card set is no longer recoverable from the FitsChan
- AST must be obtained and built (e.g. as part of the Starlink Software Collection), and language bindings other than C (Python, Java, Perl, Fortran) rely on the AST bindings

## Evidence
