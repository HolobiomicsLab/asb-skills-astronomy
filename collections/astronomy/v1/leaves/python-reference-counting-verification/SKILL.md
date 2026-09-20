---
name: python-reference-counting-verification
description: Use when validating that Python wrappers around single-owner C libraries
  (e.g. LALSuite via SWIGLAL) free child objects when parents are destroyed and that
  reference counts behave as expected.
when_to_use_negative:
- The wrappers are Octave rather than Python — ownership semantics should be checked
  via SWIGTestLALOctave.m and Octave's reference counting instead
- The goal is generating or annotating the SWIG interface itself (e.g. the generate_swig_iface.py
  XML-tree workflow) rather than verifying memory ownership behaviour
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3344
- http://edamontology.org/topic_3489
tools:
- name: SWIGLAL
  role: Provides the ownership-tracking layer and Python wrapper tests (SWIGTestLALPython)
    exercised by this verification skill
  repo: https://git.ligo.org/lscsoft/lalsuite
- name: SWIG
  role: Run in -python mode over annotated LALSuite C headers to generate the C wrapper
    code compiled into the testable Python module
- name: Python
  role: Host language whose native reference counting is used to verify object lifetime
    and parent/child ownership behaviour
- name: LALSuite
  role: C library whose single-owner memory paradigm is being reconciled with Python's
    garbage-collected semantics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2020.100634
    title: 'SWIGLAL: Python and Octave interfaces to the LALSuite gravitational-wave
      data analysis libraries'
  source_content_hash: sha256:e1650842cf23a8d3f9738f93279b3b5a6a6f56eb0dd6ad94c6a05e5c0a1b0991
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C
  - LALSuite
  - Python
  - SWIG
  - SWIGLAL
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3344
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# python-reference-counting-verification

## Summary

Verifies that SWIGLAL's memory-ownership tracking layer correctly manages LALSuite object lifetimes by exercising parent/child object destruction from Python and checking Python's native reference counting, ensuring users are freed from manual memory management.

## When to use

Use when validating that Python wrappers around single-owner C libraries (e.g. LALSuite via SWIGLAL) free child objects when parents are destroyed and that reference counts behave as expected.

## When NOT to use

- The wrappers are Octave rather than Python — ownership semantics should be checked via SWIGTestLALOctave.m and Octave's reference counting instead
- The goal is generating or annotating the SWIG interface itself (e.g. the generate_swig_iface.py XML-tree workflow) rather than verifying memory ownership behaviour

## Inputs

- SWIGTest.c test library compiled against SWIGLALTest.h
- Annotated LALSuite C headers passed through SWIG in -python mode
- SWIGTestLALPython test file with ownership-focused checks
- Generated, dynamically loadable Python wrapper module for LALSuite

## Outputs

- memory_ownership_report.txt with one pass/fail status per parent/child ownership and reference-counting check
- Per-check records of observed vs expected object lifetime behaviour

## How to apply

Build the SWIGLAL test library by compiling SWIGTest.c against the SWIGLALTest.h header, then generate the Python wrappers by running SWIG in -python mode over the annotated LALSuite C headers and compiling the resulting C wrapper code into a dynamically loadable module. Instantiate parent and child LALSuite objects from the generated Python module and use Python's reference counting (e.g. sys.getrefcount) together with the SWIGLAL ownership-tracking layer to confirm that destroying a parent object also frees its children. Exercise the ownership-focused checks from the SWIGTestLALPython test file, recording for each check whether the expected object lifetime behaviour was observed. Rationale: LALSuite assumes each class instance has exactly one owner, so SWIGLAL must reconcile this with Python's garbage-collected multiple-reference semantics; the tests demonstrate that the tracking layer plus native reference counting eliminates manual memory management.

## Related tools

- **SWIGLAL** (Provides the ownership-tracking layer and Python wrapper tests (SWIGTestLALPython) exercised by this verification skill) — https://git.ligo.org/lscsoft/lalsuite
- **SWIG** (Run in -python mode over annotated LALSuite C headers to generate the C wrapper code compiled into the testable Python module)
- **Python** (Host language whose native reference counting is used to verify object lifetime and parent/child ownership behaviour)
- **LALSuite** (C library whose single-owner memory paradigm is being reconciled with Python's garbage-collected semantics)

## Examples

```
swig -python -o SWIGTestLAL_wrap.c SWIGTestLAL.i && gcc -shared -fPIC SWIGTestLAL_wrap.c -o _SWIGTestLAL.so && python -c "import _SWIGTestLAL; p=_SWIGTestLAL.Parent(); c=p.child; del p; assert sys.getrefcount(c)==1"
```

## Evaluation signals

- Every parent/child ownership check in memory_ownership_report.txt reports PASS
- Every reference-counting check reports PASS
- Destroying a parent LALSuite object via the Python wrapper also frees its children (expected lifetime behaviour observed)
- User-level code requires no manual memory management calls while respecting LALSuite's memory management paradigm
- The wrapper module imports successfully (dynamically loadable via `import`)

## Limitations

- SWIGLAL cannot be built directly from the extracted repository; it must be built as part of LALSuite, so tests require the full LALSuite build system (GNU Autoconf/Automake macros)
- Only Python is covered by this verification; the parallel Octave path (SWIGTestLALOctave.m) needs separate exercise
- Results depend on the SWIGLAL wrapper version matching the tagged LAL library version

## Evidence

- [abstract] ownership tracking reconciling paradigms: "completely frees the user from any manual memory management, as is appropriate for a high-level language, while respecting the LALSuite memory management paradigm"
- [readme] two-pass SWIG wrapper generation: "The first pass uses the SWIG `-xml` mode to generate an XML syntax tree describing the public constants, functions, and classes exported by LALSuite"
- [readme] test files: "SWIGLAL includes extensive tests of the Python and Octave wrappers, provided by the files `SWIGLALTest.h`, `SWIGTest.c`, `SWIGTestLALPython`, and `SWIGTestLALOctave.m`."
- [readme] wrapper compilation: "The generated wrapper code (C for Python, C++ for Octave) is then compiled to create a dynamically loadable modules"
