---
name: hdf5-backend-integration
description: Use when an existing HDS API client must read both legacy HDSv4 files
  and new HDF5-backed HDSv5 files transparently, without modifying application code.
when_to_use_negative:
- When migrating HDSv4 files to HDF5 — the author explicitly deferred this and considered
  a conversion tool instead of the wrapper
- When the application needs to query low-level implementation details, since a few
  of the ~150 HDS API routines were intentionally not implemented
- When the target format is already known to be HDF5 only and no HDSv4 compatibility
  is needed — call the HDS/HDF5 library directly
tools:
- name: HDS
  role: Native HDSv4 backend to which wrapper calls are forwarded when the locator
    version field indicates v4
  repo: https://github.com/Starlink
- name: HDF5
  role: Underlying storage backend of HDSv5 to which wrapper calls are forwarded for
    HDF5-backed files
  repo: https://github.com/Starlink
- name: EMS
  role: Starlink Error Message Service that receives backend HDF5 errors via status-check
    wrappers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:c9934b239417056229ecb00200f3ae76b7d2bd8ba3bbf51de060b3e4d3e09fe7
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hdf5-backend-integration

## Summary

Wraps the ~150 public HDS API routines in a dispatch layer that forwards each call to the native HDSv4 backend or the HDF5-based HDSv5 backend, selected by a version integer embedded in the HDSLoc locator structure.

## When to use

Use when an existing HDS API client must read both legacy HDSv4 files and new HDF5-backed HDSv5 files transparently, without modifying application code.

## When NOT to use

- When migrating HDSv4 files to HDF5 — the author explicitly deferred this and considered a conversion tool instead of the wrapper
- When the application needs to query low-level implementation details, since a few of the ~150 HDS API routines were intentionally not implemented
- When the target format is already known to be HDF5 only and no HDSv4 compatibility is needed — call the HDS/HDF5 library directly

## Inputs

- HDSv4 and HDSv5 library sources and headers from the Starlink repository
- HDS files in either HDSv4 (classic) or HDSv5 (HDF5) format opened through HDSLoc locators
- Tuning parameter controlling HDSv4 vs HDSv5 output format on file creation

## Outputs

- Wrapper library exposing the full public HDS API (~150 routines) that dispatches to the correct backend
- HDSv4 or HDSv5 files re-openable by the intended backend
- EMS error stack populated from the HDF5 error stack on backend failures

## How to apply

Extend the HDSLoc locator struct so a version integer is its first member, initializing that field whenever HDSv4 or HDSv5 creates a locator. Write wrapper functions for the approximately 150 public HDS API routines; each wrapper reads the locator's version field and invokes the corresponding HDSv4 or HDSv5 routine. On file open, attempt HDSv4 first and fall back to HDSv5 when the file is not recognized as HDSv4; on file creation, choose the output format via a tuning parameter. Forward HDF5 backend errors into the EMS error stack by wrapping every HDF5 call with an EMS status check so Starlink-style error handling continues to work.

## Related tools

- **HDS** (Native HDSv4 backend to which wrapper calls are forwarded when the locator version field indicates v4) — https://github.com/Starlink
- **HDF5** (Underlying storage backend of HDSv5 to which wrapper calls are forwarded for HDF5-backed files) — https://github.com/Starlink
- **EMS** (Starlink Error Message Service that receives backend HDF5 errors via status-check wrappers)

## Evaluation signals

- Compile the wrapper library and confirm each public HDS API routine dispatches to the correct backend
- Create files under each tuning-parameter setting and verify each re-opens with the intended backend (HDSv4 or HDSv5)
- Verify that an HDSv4 file open falls back to HDSv5 when the file is not recognized as HDSv4
- Check that HDF5 backend errors are correctly transferred onto the EMS error stack
- Confirm the locator version integer is initialized as the first member whenever any backend creates a locator

## Limitations

- A handful of the ~150 HDS API routines, those querying low-level implementation details, were not implemented
- Migration of existing HDSv4 files to HDF5 was not addressed in the implementation
- Whether the wrapper approach or a one-time conversion tool is preferable for archives such as the ~4 million JCMT Science Archive files remained an open question

## Evidence

- [other] wrapper dispatch on version integer: [evidence span withheld — anchor status: not_found]
- [discussion] API coverage: [evidence span withheld — anchor status: not_found]
