---
name: hds-api-dispatch-wrapping
description: Use when you must support both legacy HDS v4 files and new HDF5-backed
  HDS v5 files behind one drop-in HDS API, and need a dispatch mechanism that routes
  each API call based on the locator's file version.
when_to_use_negative:
- When migrating HDSv4 files to HDF5 is the goal — the README states 'I'm not yet
  worrying about migration of HDS v4 files to HDF5'; a conversion tool is a separate
  path
- When you only need to read/write HDF5-backed files directly (use the hds-v5 library
  itself rather than a dispatch wrapper)
- When the routines query low-level implementation details — per the article, a handful
  of such routines were not implemented
tools:
- name: HDS
  role: Legacy HDSv4 backend whose public API the wrapper reimplements and forwards
    to when a locator's version field indicates HDSv4
  repo: https://github.com/Starlink
- name: HDF5
  role: Underlying storage backend for HDSv5; the dispatch wrapper forwards calls
    to HDF5-backed routines when the locator version indicates HDSv5
- name: EMS
  role: 'Starlink Error Message Service receiving backend errors: every forwarded
    HDF5 call is wrapped with an EMS status check that copies the HDF5 error stack
    into EMS on failure'
- name: uthash
  role: Hash-table macros used within the re-implemented library to track HDS group
    membership (relevant to wrapper-level bookkeeping such as hdsLink/hdsFlush)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:7615bd832fa29c6c098644d35ab0709adc6dff5497c06660ed23ef8f32aeb36d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EMS
  - HDF5
  - HDS
  - uthash
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hds-api-dispatch-wrapping

## Summary

Wraps the ~150 public HDS API routines so that each call is dispatched at runtime to the correct backend (native HDSv4 or the HDF5-based HDSv5) by inspecting a version integer stored as the first member of the shared HDSLoc locator struct. This lets the Starlink software run unmodified over either file format.

## When to use

Use when you must support both legacy HDS v4 files and new HDF5-backed HDS v5 files behind one drop-in HDS API, and need a dispatch mechanism that routes each API call based on the locator's file version.

## When NOT to use

- When migrating HDSv4 files to HDF5 is the goal — the README states 'I'm not yet worrying about migration of HDS v4 files to HDF5'; a conversion tool is a separate path
- When you only need to read/write HDF5-backed files directly (use the hds-v5 library itself rather than a dispatch wrapper)
- When the routines query low-level implementation details — per the article, a handful of such routines were not implemented

## Inputs

- HDSv4 and HDSv5 library sources and headers from the Starlink GitHub repository
- HDSLoc locator structure definitions (HDSv4 and HDSv5 variants)
- HDS files being opened (must be auto-detected as HDSv4 or HDSv5)
- Tuning parameter controlling which backend creates new files

## Outputs

- Wrapper library exposing the full public HDS API (~150 routines) that dispatches to HDSv4 or HDSv5 per locator
- EMS error stack entries populated from the HDF5 error stack on backend failures
- Newly created files in either HDSv4 or HDSv5 format depending on the tuning parameter

## How to apply

Extend the HDSLoc locator struct so that a version integer is its first member, and initialize that field whenever any locator is created in either the HDSv4 or HDSv5 library. Write wrapper functions for the ~150 public HDS API routines that read the locator's version field and forward each call to the routine of the matching backend library. On the open path, attempt HDSv4 first and fall back to HDSv5 when the file is not recognized as HDSv4; on the create path, control the output format (HDSv4 vs HDSv5) via a tuning parameter. Forward HDF5 backend errors into the EMS error stack by wrapping all HDF5 calls with an EMS status check that reads the HDF5 error stack on failure. Validate by compiling the wrapper and confirming each public routine dispatches correctly and that files created under each tuning setting re-open with the intended backend.

## Related tools

- **HDS** (Legacy HDSv4 backend whose public API the wrapper reimplements and forwards to when a locator's version field indicates HDSv4) — https://github.com/Starlink
- **HDF5** (Underlying storage backend for HDSv5; the dispatch wrapper forwards calls to HDF5-backed routines when the locator version indicates HDSv5)
- **EMS** (Starlink Error Message Service receiving backend errors: every forwarded HDF5 call is wrapped with an EMS status check that copies the HDF5 error stack into EMS on failure)
- **uthash** (Hash-table macros used within the re-implemented library to track HDS group membership (relevant to wrapper-level bookkeeping such as hdsLink/hdsFlush))

## Evaluation signals

- Compile the wrapper and confirm each of the ~150 public HDS API routines dispatches to the backend matching the locator's version integer
- Create files under each tuning parameter setting and confirm each re-opens with the intended backend (HDSv4 attempted first, HDSv5 fallback on open)
- Trigger a backend failure and verify the HDF5 error stack content appears in the EMS error stack
- Confirm a small number of low-level implementation-detail routines are excluded (per the article, 'all but a handful' were implemented)
- Verify locator creation always initializes the version field as the first member of HDSLoc so no call dispatches on uninitialized data

## Limitations

- A handful of the approximately 150 HDS API functions were not implemented — those that query low-level implementation details
- HDF5 does not support primary vs secondary locator semantics, so datPrmry behavior may require asserting all locators are primary or tracking this in the wrapper
- Migration of HDSv4 files to HDF5 was not addressed; the wrapper forwards calls but does not convert existing files
- datRef and datMsg do not report the full path to the file, and datWhere is probably not going to work

## Evidence

- [discussion] scope of the API surface: [evidence span withheld — anchor status: not_found]
