---
name: c-locator-struct-versioning
description: Use when a drop-in HDS replacement must interoperate with legacy HDSv4
  files and new HDF5-backed HDS files, so public API calls on HDSLoc locators must
  be routed to the matching backend implementation.
when_to_use_negative:
- You only need the standalone HDS/HDF5 library (libhdsh5) and will not open legacy
  HDSv4 files — the README defers the wrapper question rather than mandating it
- You plan to write a one-off conversion tool copying structures from HDS classic
  to HDF5 instead of maintaining runtime dual-backend dispatch
- The routine queries low-level implementation details of one specific backend (a
  handful of the ~150 API functions were not implemented for this reason)
tools:
- name: HDS
  role: Legacy HDSv4 backend and the public API surface (~150 routines) that the wrapper
    re-exposes with version-based dispatch
  repo: https://github.com/Starlink
- name: HDF5
  role: Underlying storage backend for the HDSv5 implementation; its error stack is
    forwarded into EMS by the wrapper
  repo: https://github.com/Starlink/hds-v5
- name: EMS
  role: Starlink Error Message Service used by the wrapper to record HDF5 backend
    errors via status checks around every backend call
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:5356e5e86359cd375b7eadc2dc837ec2a93e590c8e80e83c2a37ff050b6b04ba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EMS
  - HDF5
  - HDS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# c-locator-struct-versioning

## Summary

A wrapper-library dispatch pattern for the re-implemented HDS: a version integer placed as the first member of both the HDSv4 and HDSv5 locator structures lets ~150 public HDS API routines forward each call to the correct backend (native HDSv4 or the HDF5-based HDS) depending on which format the opened file uses.

## When to use

Use when a drop-in HDS replacement must interoperate with legacy HDSv4 files and new HDF5-backed HDS files, so public API calls on HDSLoc locators must be routed to the matching backend implementation.

## When NOT to use

- You only need the standalone HDS/HDF5 library (libhdsh5) and will not open legacy HDSv4 files — the README defers the wrapper question rather than mandating it
- You plan to write a one-off conversion tool copying structures from HDS classic to HDF5 instead of maintaining runtime dual-backend dispatch
- The routine queries low-level implementation details of one specific backend (a handful of the ~150 API functions were not implemented for this reason)

## Inputs

- HDSv4 library sources and headers from the Starlink repository
- HDSv5 (libhdsh5) library sources and headers from the Starlink/hds-v5 repository
- Existing HDSv4 and HDSv5 locator (HDSLoc) struct definitions
- Files being opened in either HDSv4 (classic) or HDF5-backed HDS format

## Outputs

- Wrapper library exposing the full public HDS API (~150 routines) with version-field-based dispatch
- Extended HDSLoc struct with a version integer as its first member
- EMS error stack entries populated from the HDF5 error stack on backend failures
- HDS files created in HDSv4 or HDSv5 format according to the tuning parameter

## How to apply

Extend the shared HDSLoc locator struct so a version integer is its first member and initialize that field whenever a locator is created, since dispatch relies on reading it before every call. Implement open so HDSv4 is attempted first on every file open, falling back to the HDF5-based HDS implementation when the file is not recognized as HDSv4; control whether new files are written as HDSv4 or HDSv5 via a tuning parameter. Write wrapper functions for the ~150 public HDS API routines that read the locator's version field and invoke the routine belonging to the corresponding library version. Wrap all backend calls with an EMS status check so the HDF5 error stack is read and stored in the EMS error stack on failure. Validate by compiling the wrapper and confirming each public routine dispatches correctly and that files created under each tuning setting re-open with the intended backend.

## Related tools

- **HDS** (Legacy HDSv4 backend and the public API surface (~150 routines) that the wrapper re-exposes with version-based dispatch) — https://github.com/Starlink
- **HDF5** (Underlying storage backend for the HDSv5 implementation; its error stack is forwarded into EMS by the wrapper) — https://github.com/Starlink/hds-v5
- **EMS** (Starlink Error Message Service used by the wrapper to record HDF5 backend errors via status checks around every backend call)

## Evaluation signals

- Wrapper compiles cleanly and every public HDS API routine dispatches to the backend matching the locator's version field
- A file opened as HDSv4 is served by the native HDSv4 backend; an unrecognized file falls back to the HDF5-based backend
- Files created under each tuning-parameter setting (HDSv4 vs HDSv5) re-open with the intended backend
- On a forced HDF5 failure, the HDF5 error stack content appears in the EMS error stack rather than being lost
- Behavior of the ~150 wrapped routines matches the original HDS API semantics for callers (drop-in replacement)

## Limitations

- A few of the approximately 150 HDS API functions — those querying low-level implementation details — were not implemented and cannot be dispatched
- Migration of HDS v4 files to HDF5 is not addressed by the HDSv5 implementation itself; the wrapper only forwards calls, it does not convert files
- The decision on whether the wrapper approach (vs a conversion tool) is the right migration path was still open in the README

## Evidence

- [readme] dispatch reads locator version field: "Should there be a wrapper HDS library that manages to forward calls to either the native HDS or HDS/HDF5 based on the file that is being opened?"
- [methods] EMS wrapping of HDF5 calls: "All calls to HDF5 are wrapped by an EMS status check and on error the HDF5 stack is read and stored in EMS error stack."
- [discussion] API coverage of ~150 routines: [evidence span withheld — anchor status: not_found]
- [readme] drop-in replacement intent: "The idea is that this library could be dropped in as a direct replacement to HDS and allow the entire Starlink software collection to run, using HDF5 files behind the scenes"
