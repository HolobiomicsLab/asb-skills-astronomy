---
name: astronomical-coordinate-systems-model-construction
description: Use when a task must model, manipulate, or verify inter-related coordinate
  systems (base/intermediate/current Frames connected by Mappings) and needs to confirm
  that mutating a current Frame through a FrameSet keeps enclosed Mappings consistent.
when_to_use_negative:
- Input is already a simple single-frame coordinate description with no inter-related
  Frames — AST FrameSets add integrity machinery you will not exercise
- You intend to mutate a Frame while deliberately bypassing FrameSet integrity restoration
  (you must take a direct reference to the internal Frame; the FrameSet path will
  auto-restore)
- You only need FITS-WCS header writing/parsing and not the data model of inter-related
  coordinate systems
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Starlink AST
  role: Provides the Frame, Mapping and FrameSet classes whose dual Mapping/Frame
    behaviour and integrity restoration are the object of the skill
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Binding language used to instantiate FrameSet and Frame objects and drive
    the restoration/bypass checks
- name: C
  role: Implementation language of the AST library built from source before the Python
    binding can be exercised
  repo: https://github.com/Starlink/starlink
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:3aa89dd480d1b58e6be4e8c145452befe85404c2f081a877807458d891ac0a77
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - C
  - Python
  - Starlink AST
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# astronomical-coordinate-systems-model-construction

## Summary

Builds and validates inter-related coordinate-system models using the Starlink AST library, specifically exercising FrameSet behaviour as both a Mapping and a Frame, including automatic integrity restoration when the current Frame's attributes are changed via the FrameSet reference.

## When to use

Use when a task must model, manipulate, or verify inter-related coordinate systems (base/intermediate/current Frames connected by Mappings) and needs to confirm that mutating a current Frame through a FrameSet keeps enclosed Mappings consistent.

## When NOT to use

- Input is already a simple single-frame coordinate description with no inter-related Frames — AST FrameSets add integrity machinery you will not exercise
- You intend to mutate a Frame while deliberately bypassing FrameSet integrity restoration (you must take a direct reference to the internal Frame; the FrameSet path will auto-restore)
- You only need FITS-WCS header writing/parsing and not the data model of inter-related coordinate systems

## Inputs

- FrameSet definition (base Frame, intermediate Frame connected by a Mapping, current Frame)
- Frame attribute mutation (e.g. an attribute change applied via the FrameSet reference or via a direct internal Frame reference)
- Starlink AST library build from the Starlink/starlink source tree

## Outputs

- FrameSet-to-Mapping coordinate transformation behaviour for base-to-current Frame conversion
- Pass/fail record of the restoration-path check and the bypass-path check
- frameset_test_report.txt (parseable text report of both checks)

## How to apply

Obtain the Starlink AST library (ascl:1404.016) via the Starlink/starlink repository and build it with the documented sequence (./bootstrap, make configure-deps, ./configure -C, make world), since AST is written in C but has Python bindings. In Python, instantiate Frame and FrameSet objects and construct a FrameSet containing a base Frame, an intermediate Frame connected by a Mapping, and a current Frame, then verify the FrameSet itself exposes the Mapping interface transforming base-Frame coordinates to current-Frame coordinates. Mutate an attribute of the current Frame through the FrameSet reference and confirm the enclosed Mappings are automatically modified so they still generate values matching the new current Frame (the Section 3.3.3 integrity-restoration path). Repeat the same mutation via a direct reference to the internal Frame object to record that automatic restoration is bypassed and the Mappings are left unchanged. Write both observed behaviours with pass/fail status per check to a report file (e.g. frameset_test_report.txt).

## Related tools

- **Starlink AST** (Provides the Frame, Mapping and FrameSet classes whose dual Mapping/Frame behaviour and integrity restoration are the object of the skill) — https://github.com/Starlink/starlink
- **Python** (Binding language used to instantiate FrameSet and Frame objects and drive the restoration/bypass checks)
- **C** (Implementation language of the AST library built from source before the Python binding can be exercised) — https://github.com/Starlink/starlink

## Examples

```
% ./bootstrap; % make configure-deps; % ./configure -C; % make world
```

## Evaluation signals

- The FrameSet itself exposes the Mapping interface and transforms coordinates correctly from the base Frame to the current Frame
- After mutating a current-Frame attribute via the FrameSet reference, enclosed Mappings still generate values matching the new current Frame (restoration path passes)
- After the same mutation via a direct reference to the internal Frame, the Mappings are left unchanged (bypass path passes)
- frameset_test_report.txt exists, is parseable text, and explicitly records outcomes of both the restoration-path and bypass-path checks
- The library was built successfully from source with the documented ./bootstrap && make configure-deps && ./configure -C && make world sequence

## Limitations

- Integrity restoration only fires when the current Frame is changed through a FrameSet reference; a direct reference to an internal Frame bypasses it, leaving Mappings stale
- Building the full classic source set requires patched autoconf/automake/libtool installed by ./bootstrap and careful environment setup (STARCONF_DEFAULT_PREFIX, unsetting stale INSTALL/STARLINK variables); Java applications are built separately via Starlink/starjava

## Evidence

- [other] FrameSet integrity restoration finding: [evidence span withheld — anchor status: not_found]
