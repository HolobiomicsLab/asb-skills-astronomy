---
name: coordinate-frame-identity-verification
description: Use when a FITS header buffer contains WCS cards (CTYPE/CRVAL/CRPIX-type)
  that must be read into an AST FrameSet, and the pixel-vs-world coordinate frame
  identity of the base and current Frames must be confirmed.
when_to_use_negative:
- The header uses a non-FITS-WCS encoding (e.g. native or FITS-IRAF encodings) where
  the base/current Frame mapping being verified does not apply
- The FITS cards must be preserved after reading — the FitsChan Read is destructive
  and deletes the relevant cards from the buffer
- The coordinate system description is not stored in FITS header cards (e.g. already
  an AST Channel text dump), in which case a Channel read, not a FitsChan read, is
  appropriate
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Starlink AST
  role: Provides FitsChan, the destructive astRead operation, FrameSet base/current
    Frame model, and Channel serialisation used to reconstruct and verify the WCS
    FrameSet
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Language binding used to construct the FitsChan and perform the astRead on
    FITS header card buffers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:b372b3d011de05c90bcaa2458018c207dad5cf2fe2d1d3385d11bf206e70316c
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# coordinate-frame-identity-verification

## Summary

Reconstructs an AST FrameSet from FITS-WCS header cards via a destructive FitsChan read (astRead) and verifies that the base Frame represents pixel coordinates while the current Frame represents the FITS primary WCS frame.

## When to use

Use when a FITS header buffer contains WCS cards (CTYPE/CRVAL/CRPIX-type) that must be read into an AST FrameSet, and the pixel-vs-world coordinate frame identity of the base and current Frames must be confirmed.

## When NOT to use

- The header uses a non-FITS-WCS encoding (e.g. native or FITS-IRAF encodings) where the base/current Frame mapping being verified does not apply
- The FITS cards must be preserved after reading — the FitsChan Read is destructive and deletes the relevant cards from the buffer
- The coordinate system description is not stored in FITS header cards (e.g. already an AST Channel text dump), in which case a Channel read, not a FitsChan read, is appropriate

## Inputs

- Buffer of FITS header cards encoding WCS information (CTYPE/CRVAL/CRPIX-type cards)
- FITS-WCS encoding specification for the FitsChan

## Outputs

- AST FrameSet with base Frame = pixel coordinates and current Frame = FITS primary WCS frame
- Residual FitsChan header buffer with WCS cards removed
- Text serialisation of the FrameSet written via an AST Channel

## How to apply

Construct an AST FitsChan through the Python bindings (the AST library is written in C but has Python bindings) and feed it the buffer of FITS header cards, specifying the FITS-WCS encoding. Perform the destructive read (astRead) to obtain the FrameSet that encodes the FITS primary WCS representation; AST's Read method both reads the object described by the header cards and deletes the consumed cards from the FitsChan buffer. Verify the returned FrameSet's base Frame is set to pixel coordinates and its current Frame is set to the FITS primary WCS frame. Confirm destructiveness by inspecting the FitsChan for the consumed WCS cards (e.g. CTYPE/CRVAL/CRPIX-type) and checking they are gone from the residual buffer. Finally, serialise the FrameSet to a text file via an AST Channel and write the residual header buffer to a second file so both artefacts can be diffed against expectations.

## Related tools

- **Starlink AST** (Provides FitsChan, the destructive astRead operation, FrameSet base/current Frame model, and Channel serialisation used to reconstruct and verify the WCS FrameSet) — https://github.com/Starlink/starlink
- **Python** (Language binding used to construct the FitsChan and perform the astRead on FITS header card buffers)

## Examples

```
from starlink import ast; fs = ast.FitsChan(cards, encoding='FITS-WCS').read(); print(fs.getframe(ast.BASE), fs.getframe(ast.CURRENT))
```

## Evaluation signals

- Returned object is an AST FrameSet whose base Frame is set to pixel coordinates
- Returned FrameSet's current Frame is set to the FITS primary WCS frame
- The residual FitsChan buffer contains none of the WCS cards (CTYPE/CRVAL/CRPIX-type) supplied on input, confirming the read was destructive
- The FrameSet serialises cleanly to a text file via an AST Channel without read errors
- The residual header buffer written to file matches the expected set of non-WCS cards remaining after the read

## Limitations

- The FitsChan Read is destructive: all relevant FITS cards are deleted from the buffer, so the original header cannot be re-read from the same FitsChan instance
- Correct base/current Frame identity is tied to the FITS-WCS encoding of the primary WCS representation; other encodings may yield different Frame arrangements
- Only cards recognised as describing the AST object are removed; unrecognised cards remain in the residual buffer

## Evidence

- [full_text] destructive FitsChan read semantics: [evidence span withheld — anchor status: not_found]
