---
name: lalsuite-public-api-introspection
description: Use when you need an exhaustive, machine-readable inventory of the public
  constants, functions, and classes exported by a LALSuite C library's header files,
  e.g. to automate SWIG interface annotation.
when_to_use_negative:
- The library's bindings are for a language other than Python or Octave — the workflow's
  second pass only supports SWIG's -python and -octave modes
- The input is already an annotated, hand-maintained SWIG interface with no LALSuite
  C headers to introspect — regenerating from XML would overwrite existing annotations
- The C code cannot be built via the GNU Autoconf/Automake build system used by LALSuite,
  since the workflow is implemented as macros and build rules in that system
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3489
tools:
- name: SWIG
  role: Used in -xml mode as a C header parser for the first pass (XML syntax tree),
    then a second time in -python or -octave mode to generate the wrappers
  repo: http://www.swig.org/
- name: Python
  role: Runs the generate_swig_iface.py script that parses the XML syntax tree and
    automates further annotation of the LALSuite C header files
  repo: https://www.python.org/
- name: SWIGLAL
  role: Collection of scripts implementing the two-pass SWIG workflow; provides SWIGCommon.i,
    SWIGLALAlpha.i, and SWIGLALOmega.i definitions used for annotation
  repo: ElsevierSoftwareX/SOFTX_2020_119
- name: LALSuite
  role: Source of the C header files whose public API is introspected; SWIGLAL is
    fully integrated into LALSuite
  repo: https://git.ligo.org/lscsoft/lalsuite/blob/master/CONTRIBUTING.md
- name: GNU Autoconf/Automake
  role: Hosts the workflow as a collection of macros and build rules in the LALSuite
    build system
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2020.100634
    title: 'SWIGLAL: Python and Octave interfaces to the LALSuite gravitational-wave
      data analysis libraries'
  source_content_hash: sha256:5f586fd29f07cd2c1426d5b7aa60aa8ff56fd8b9200ba078cb2750000973d503
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C
  - GNU Autoconf/Automake
  - LALSuite
  - Python
  - SWIG
  - SWIGLAL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lalsuite-public-api-introspection

## Summary

Introspects the public API of a LALSuite C library by running SWIG in -xml mode as a C header parser, yielding an XML syntax tree that enumerates every exported public constant, function, and class. This machine-readable API inventory then drives automated annotation of the SWIG interface (e.g. via generate_swig_iface.py).

## When to use

Use when you need an exhaustive, machine-readable inventory of the public constants, functions, and classes exported by a LALSuite C library's header files, e.g. to automate SWIG interface annotation.

## When NOT to use

- The library's bindings are for a language other than Python or Octave — the workflow's second pass only supports SWIG's -python and -octave modes
- The input is already an annotated, hand-maintained SWIG interface with no LALSuite C headers to introspect — regenerating from XML would overwrite existing annotations
- The C code cannot be built via the GNU Autoconf/Automake build system used by LALSuite, since the workflow is implemented as macros and build rules in that system

## Inputs

- All C header files of a LALSuite library (the basic SWIG interface simply incorporates these headers)
- Basic SWIG interface file incorporating the library's C headers
- SWIGCommon.i shared definitions, plus SWIGLALAlpha.i and SWIGLALOmega.i LAL-specific definitions
- generate_swig_iface.py custom Python script

## Outputs

- XML syntax tree describing the public constants, functions, and classes exported by the LALSuite library (SWIG -xml first pass)
- Fully annotated SWIG interface file (e.g. SWIGTestLALPython.i) ready for SWIG's second -python or -octave pass
- Python or Octave wrapper code (C for Python, C++ for Octave) after the second SWIG pass, compiled into dynamically loadable modules

## How to apply

Run SWIG in its -xml mode over all the C header files of the target LALSuite library; in this first pass SWIG acts purely as a C header parser and produces an XML syntax tree describing the public constants, functions, and classes exported by the library. Parse this XML syntax tree with the custom Python script generate_swig_iface.py, which enumerates every exported public constant, function, and class. Use the parsed information to insert SWIGLAL annotation directives into the interface definitions, drawing on the shared definitions in SWIGCommon.i and the LAL-specific definitions in SWIGLALAlpha.i and SWIGLALOmega.i. Write out the fully annotated SWIG interface file (e.g. SWIGTestLALPython.i for the test header) so it is ready for SWIG's second pass in -python or -octave mode. The rationale is that hand-maintaining interface annotations across a large C API is error-prone, so the XML syntax tree serves as the authoritative source of truth for the full public API surface.

## Related tools

- **SWIG** (Used in -xml mode as a C header parser for the first pass (XML syntax tree), then a second time in -python or -octave mode to generate the wrappers) — http://www.swig.org/
- **Python** (Runs the generate_swig_iface.py script that parses the XML syntax tree and automates further annotation of the LALSuite C header files) — https://www.python.org/
- **SWIGLAL** (Collection of scripts implementing the two-pass SWIG workflow; provides SWIGCommon.i, SWIGLALAlpha.i, and SWIGLALOmega.i definitions used for annotation) — ElsevierSoftwareX/SOFTX_2020_119
- **LALSuite** (Source of the C header files whose public API is introspected; SWIGLAL is fully integrated into LALSuite) — https://git.ligo.org/lscsoft/lalsuite/blob/master/CONTRIBUTING.md
- **GNU Autoconf/Automake** (Hosts the workflow as a collection of macros and build rules in the LALSuite build system)

## Examples

```
swig -xml -outdir xml lal.i && python generate_swig_iface.py --xml xml/lal.xml --output SWIGTestLALPython.i && swig -python lal.i
```

## Evaluation signals

- The annotated interface file (e.g. SWIGTestLALPython.i) exists, is non-empty, and contains SWIGLAL annotation directives
- The annotated interface covers every public constant, function, and class present in the XML syntax tree
- The XML syntax tree from the SWIG -xml first pass enumerates all exported public constants, functions, and classes of the LALSuite library
- The annotated interface passes cleanly through SWIG's second pass in -python or -octave mode and compiles into a dynamically loadable module (importable via 'import' in Python)
- SWIGLAL's extensive wrapper tests (SWIGLALTest.h, SWIGTest.c, SWIGTestLALPython, SWIGTestLALOctave.m) pass

## Limitations

- SWIGLAL is fully integrated into LALSuite and cannot be built directly from the extracted repository; obtaining the interfaces requires LALSuite itself (pre-built packages or building from source)
- The XML syntax tree only captures the public constants, functions, and classes exported by the C headers — it does not invent semantics, so further integration code must come from SWIGCommon.i and the LAL-specific headers
- Generated wrapper code is language-specific (C for Python, C++ for Octave), so a single introspection feeds two separate second-pass compilations

## Evidence

- [methods] SWIG first pass as C header parser: "The first pass uses the SWIG `-xml` mode to generate an XML syntax tree describing the public constants, functions, and classes exported by LALSuite"
- [methods] generate_swig_iface.py automates annotation: "used by the script `generate_swig_iface.py` to automate further annotation of the LALSuite C header files"
- [methods] second pass modes: "which are then passed through SWIG a second time, using either its `-python` or `-octave` mode to generate Python or Octave wrappers respectively"
- [readme] README: SWIG as C header parser: "The first pass uses the SWIG `-xml` mode to generate an XML syntax tree describing the public constants, functions, and classes exported by LALSuite; in essence, SWIG is used as a C header parser."
- [readme] README: header definitions used for annotation: "The headers `SWIGLALAlpha.i` and `SWIGLALOmega.i` provide definitions specific to the LAL library."
