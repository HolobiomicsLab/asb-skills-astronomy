---
name: lalsuite-api-surface-extraction
description: Use when you need a machine-readable enumeration of the public API surface
  (constants, functions, classes) of a LALSuite C library, e.g. as input to automated
  interface annotation or wrapper generation.
when_to_use_negative:
- You already have annotated header files and a complete SWIG interface — this skill
  covers only the first `-xml` parsing pass, not the second wrapper-generating pass
  (`-python`/`-octave`).
- The target codebase is not a LALSuite C library whose headers SWIG can parse as
  C — SWIG here is used strictly as a C header parser.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3344
tools:
- name: SWIG
  role: Run in `-xml` mode as a C header parser to emit the XML syntax tree of the
    library's exported API
  repo: http://www.swig.org/
- name: SWIGLAL
  role: Provides the scripts and build-system macros that incorporate all LALSuite
    C headers into a basic SWIG interface and consume the XML tree via generate_swig_iface.py
  repo: ElsevierSoftwareX/SOFTX_2020_119
- name: LALSuite
  role: Source of the C header files being parsed; SWIGLAL is fully integrated into
    LALSuite and its headers ship with that codebase
  repo: https://git.ligo.org/lscsoft/lalsuite/blob/master/CONTRIBUTING.md
- name: GNU Autoconf/Automake
  role: Build system in which the two-pass workflow, including this first pass, is
    implemented as macros and build rules
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2020.100634
    title: 'SWIGLAL: Python and Octave interfaces to the LALSuite gravitational-wave
      data analysis libraries'
  source_content_hash: sha256:8695e6580f933ff39d04b7362863c18150e25dd4e572cfc285575ec2b6d8c71e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNU Autoconf/Automake
  - LALSuite
  - SWIG
  - SWIGLAL
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3344
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# lalsuite-api-surface-extraction

## Summary

Runs SWIG in its `-xml` mode over a LALSuite library's C header files, using SWIG purely as a C header parser to emit an XML syntax tree of the library's public constants, functions, and classes. This is the first pass of the SWIGLAL interface-generation pipeline and feeds downstream interface annotation.

## When to use

Use when you need a machine-readable enumeration of the public API surface (constants, functions, classes) of a LALSuite C library, e.g. as input to automated interface annotation or wrapper generation.

## When NOT to use

- You already have annotated header files and a complete SWIG interface — this skill covers only the first `-xml` parsing pass, not the second wrapper-generating pass (`-python`/`-octave`).
- The target codebase is not a LALSuite C library whose headers SWIG can parse as C — SWIG here is used strictly as a C header parser.

## Inputs

- C header files of a target LALSuite library (from git.ligo.org/lscsoft/lalsuite), incorporating all headers provided by that library
- A basic SWIG interface that incorporates all the C header files of the library

## Outputs

- An XML syntax tree (well-formed XML) describing the public constants, functions, and classes exported by the LALSuite library's C headers

## How to apply

Obtain the target LALSuite library's C header files from the git.ligo.org/lscsoft/lalsuite repository, since SWIGLAL is fully integrated into LALSuite and cannot be built directly from the extracted SWIGLAL repository. Run SWIG (Simplified Wrapper and Interface Generator) in `-xml` mode over those headers; in this mode SWIG generates no wrapper code and instead emits an XML syntax tree describing the public constants, functions, and classes exported by the headers. In the SWIGLAL pipeline, information from this XML syntax tree is then consumed by the script `generate_swig_iface.py` to automate further annotation of the C header files before a second SWIG pass in `-python` or `-octave` mode. The whole workflow is implemented as macros and build rules in the GNU Autoconf/Automake build system used by LALSuite, so in practice this step is driven by the build system rather than invoked ad hoc.

## Related tools

- **SWIG** (Run in `-xml` mode as a C header parser to emit the XML syntax tree of the library's exported API) — http://www.swig.org/
- **SWIGLAL** (Provides the scripts and build-system macros that incorporate all LALSuite C headers into a basic SWIG interface and consume the XML tree via generate_swig_iface.py) — ElsevierSoftwareX/SOFTX_2020_119
- **LALSuite** (Source of the C header files being parsed; SWIGLAL is fully integrated into LALSuite and its headers ship with that codebase) — https://git.ligo.org/lscsoft/lalsuite/blob/master/CONTRIBUTING.md
- **GNU Autoconf/Automake** (Build system in which the two-pass workflow, including this first pass, is implemented as macros and build rules)

## Examples

```
swig -xml -o lalsuite_syntax_tree.xml lalsuite.i
```

## Evaluation signals

- The output XML file (e.g. lalsuite_syntax_tree.xml) exists and parses as valid, well-formed XML.
- The XML content enumerates the public constants, functions, and classes exported by the LALSuite library headers.
- SWIG was invoked in `-xml` mode (no wrapper code emitted), confirming it acted purely as a C header parser.
- The tree contains enough information for `generate_swig_iface.py` to automate further annotation of the C header files in the next pipeline stage.

## Limitations

- SWIGLAL is fully integrated into LALSuite and cannot be built directly from the extracted SWIGLAL repository; headers must be obtained from LALSuite itself.
- This pass yields only a raw syntax tree of the public API — it does not produce usable wrappers; wrappers require the second SWIG pass in `-python` or `-octave` mode.
- The XML tree describes only the public constants, functions, and classes exported by the headers; non-public internals are not captured.

## Evidence

- [methods] The first pass uses the SWIG `-xml` mode to generate an XML syntax tree describing the public constants, functions, and classes exported by LALSuite: "The first pass uses the SWIG `-xml` mode to generate an XML syntax tree describing the public constants, functions, and classes exported by LALSuite"
- [readme] README: two-pass SWIG operation: "The first pass uses the SWIG `-xml` mode to generate an XML syntax tree describing the public constants, functions, and classes exported by LALSuite; in essence, SWIG is used as a C header parser."
- [readme] README: consumption of XML tree by generate_swig_iface.py: "Information from the XML syntax tree is used by the script `generate_swig_iface.py` to automate further annotation of the LALSuite C header files"
- [abstract] abstract: basic SWIG interface incorporates all C header files: "SWIGLAL generates a basic SWIG interface which simply incorporates all the C header files provided by that library"
- [readme] README: SWIGLAL cannot be built standalone: "SWIGLAL is fully integrated to LALSuite, and as such cannot be built directly from this repository."
