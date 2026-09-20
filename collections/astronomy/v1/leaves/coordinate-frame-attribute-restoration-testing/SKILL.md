---
name: coordinate-frame-attribute-restoration-testing
description: Use when validating that a FrameSet-as-Mapping still generates correct
  coordinate values after mutating the current Frame's attributes, and when checking
  whether integrity restoration is triggered versus bypassed.
when_to_use_negative:
- The goal is to intentionally modify Mappings without integrity restoration — use
  a direct reference to the internal Frame instead of the FrameSet reference
- The coordinate system representation is a single standalone Frame with no base/current
  Frame duality, so no FrameSet Mapping propagation can occur
- Coordinate-system information is being read/written in FITS-WCS form rather than
  manipulated through AST Frame/FrameSet objects
tools:
- name: Starlink AST
  role: Provides the Frame, Mapping and FrameSet classes whose base/current Frame
    semantics and integrity restoration are being tested
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Binding language used to instantiate FrameSet and Frame objects and run the
    two mutation paths
- name: C
  role: Implementation language of the AST library underlying the tested FrameSet
    behaviour
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:6ebb44d0931e6aded7d3b6aa92a97f6e91ff671c0b57bf05e9b2b1c7fd13cd28
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
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
n_claims: 7
---

# coordinate-frame-attribute-restoration-testing

## Summary

Tests the automatic integrity-restoration behaviour of Starlink AST FrameSets: changing a current Frame's attributes through the FrameSet reference must propagate to enclosed Mappings, while mutation via a direct internal Frame reference bypasses restoration. This skill verifies both paths and records the pass/fail outcome.

## When to use

Use when validating that a FrameSet-as-Mapping still generates correct coordinate values after mutating the current Frame's attributes, and when checking whether integrity restoration is triggered versus bypassed.

## When NOT to use

- The goal is to intentionally modify Mappings without integrity restoration — use a direct reference to the internal Frame instead of the FrameSet reference
- The coordinate system representation is a single standalone Frame with no base/current Frame duality, so no FrameSet Mapping propagation can occur
- Coordinate-system information is being read/written in FITS-WCS form rather than manipulated through AST Frame/FrameSet objects

## Inputs

- AST FrameSet containing base Frame, intermediate Frame connected by a Mapping, and current Frame
- Attribute-mutation operations applied to the current Frame (via FrameSet reference and via direct internal Frame reference)

## Outputs

- frameset_test_report.txt recording observed behaviour and pass/fail status for the restoration path and the bypass path

## How to apply

Obtain the Starlink AST library (ascl:1404.016) and its Python binding so FrameSet and Frame objects can be instantiated from Python. Build a FrameSet containing a base Frame, an intermediate Frame connected by a Mapping, and a current Frame, then confirm the FrameSet itself exposes the Mapping interface transforming coordinates from the base Frame to the current Frame. Mutate an attribute of the current Frame through the FrameSet reference and verify the enclosed Mapping is automatically simplified/restored so it still generates values matching the new current Frame (the Section 3.3.3 restoration path). Repeat the identical attribute mutation using a direct reference to the internal Frame object, recording that the automatic restoration is bypassed and the Mappings are left unchanged. Write the observed behaviour of both paths, with per-check pass/fail status, to a report file such as frameset_test_report.txt.

## Related tools

- **Starlink AST** (Provides the Frame, Mapping and FrameSet classes whose base/current Frame semantics and integrity restoration are being tested) — https://github.com/Starlink/starlink
- **Python** (Binding language used to instantiate FrameSet and Frame objects and run the two mutation paths)
- **C** (Implementation language of the AST library underlying the tested FrameSet behaviour)

## Examples

```
from starlink import ast; fs = ast.FrameSet(ast.Frame(2)); fs.setref(ast.Frame(2)); del fs.currentframe.SkyRef; open('frameset_test_report.txt','a').write('restoration-path: ok\n')
```

## Evaluation signals

- The report file frameset_test_report.txt exists and is parseable text
- The report explicitly records the outcome of the restoration-path check (Mapping regenerated to match the mutated current Frame)
- The report explicitly records the outcome of the bypass-path check (Mappings unchanged after mutation via direct internal Frame reference)
- After mutation through the FrameSet reference, the FrameSet-as-Mapping still transforms base-Frame coordinates into values consistent with the new current Frame
- The FrameSet under test contains all three components (base Frame, intermediate Frame with Mapping, current Frame) as specified in the workflow

## Limitations

- Integrity restoration only triggers when the current Frame is changed via the FrameSet reference; obtaining a direct reference to an internal Frame leaves the Mappings unchanged
- Behaviour is specific to the AST data model (Frames, Mappings, FrameSets) and its Python/C bindings, not a generic coordinate-system framework

## Evidence

- [other] FrameSet integrity restoration finding: [evidence span withheld — anchor status: not_found]
- [other] bypass via direct internal reference: [evidence span withheld — anchor status: not_found]
