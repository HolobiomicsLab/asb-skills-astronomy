---
name: file-format-fallback-open
description: Use when a legacy file-format library and its re-implementation (e.g.
  HDSv4 and HDS-on-HDF5) must coexist behind one API, and existing files of either
  flavor must open transparently without user-visible backend choice.
when_to_use_negative:
- When the goal is migrating or converting HDS v4 files to HDF5 rather than opening
  both formats — the README explicitly defers conversion, considering only a forwarding
  wrapper
- When the client code calls the few HDS routines that query low-level implementation
  details, which were not implemented in the HDSv5 backend
- When a single backend is guaranteed (e.g. all data already in HDF5 and read by external
  tools like h5py), making dual dispatch unnecessary overhead
tools:
- name: HDS
  role: Legacy (HDSv4) backend attempted first on every file open; provides the public
    API being wrapped
  repo: https://github.com/Starlink
- name: HDF5
  role: Storage backend of HDSv5; fallback target on open and creation backend selected
    by tuning parameter
- name: EMS
  role: Starlink Error Message Service that receives forwarded HDF5 error stacks from
    wrapped backend calls
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:75a4de4524a6e99ddab571e104fa6ffeb17fb8a329585e58182ae3d3ee5ca52c
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# file-format-fallback-open

## Summary

A wrapper-library pattern that lets one public API (here the ~150-routine HDS API) transparently dispatch calls to two backend implementations (HDSv4 and HDSv5/HDF5), selecting the backend per-file at open time with a fallback and controlling the output backend via a tuning parameter.

## When to use

Use when a legacy file-format library and its re-implementation (e.g. HDSv4 and HDS-on-HDF5) must coexist behind one API, and existing files of either flavor must open transparently without user-visible backend choice.

## When NOT to use

- When the goal is migrating or converting HDS v4 files to HDF5 rather than opening both formats — the README explicitly defers conversion, considering only a forwarding wrapper
- When the client code calls the few HDS routines that query low-level implementation details, which were not implemented in the HDSv5 backend
- When a single backend is guaranteed (e.g. all data already in HDF5 and read by external tools like h5py), making dual dispatch unnecessary overhead

## Inputs

- HDSv4 and HDSv5 library sources and headers from the Starlink GitHub repository
- Existing HDS v4 (.sdf) files opened by client applications
- New HDSv5/HDF5 files created under the tuning-parameter-controlled creation path

## Outputs

- A unified HDS API wrapper library (~150 public routines) that dispatches to the correct backend per locator
- Open calls resolved to HDSv4-first with fallback to HDSv5
- EMS error stack entries populated from forwarded HDF5 errors

## How to apply

Extend the shared locator struct (HDSLoc) so a version integer is its first member and initialize that field whenever a locator is created by either backend. Write wrapper functions for the ~150 public HDS API routines; each wrapper reads the locator's version field and forwards the call to the matching backend (HDSv4 or HDSv5). Implement the open path so HDSv4 is attempted first on every file open, falling back to HDSv5 when the file is not recognized as HDSv4. Route the file-creation path through a tuning parameter that selects HDSv4 vs HDSv5 output. Wrap backend HDF5 calls with an EMS status check so HDF5 error stacks are forwarded into the Starlink EMS error stack. The dispatch rationale follows the README's open question: forward calls to the native HDS or HDS/HDF5 based on the file being opened, so the library can be dropped in as a direct HDS replacement for the whole Starlink software collection.

## Related tools

- **HDS** (Legacy (HDSv4) backend attempted first on every file open; provides the public API being wrapped) — https://github.com/Starlink
- **HDF5** (Storage backend of HDSv5; fallback target on open and creation backend selected by tuning parameter)
- **EMS** (Starlink Error Message Service that receives forwarded HDF5 error stacks from wrapped backend calls)

## Evaluation signals

- Each public HDS API routine dispatches to the backend matching the locator's version field (verify by calling the same routine through HDSv4- and HDSv5-created locators)
- Files created under each tuning-parameter setting re-open with the intended backend (HDSv4-first open succeeds on v4 files; v5 files fall through to the HDF5 backend)
- The version integer is the first member of both backend locator structs and is initialized at every locator creation site
- HDF5 errors surface in the EMS error stack, not the native HDF5 stack, after wrapper interception
- Files written via the wrapper are readable by general-purpose HDF5 tools such as h5py

## Limitations

- A handful of the ~150 HDS API functions (those querying low-level implementation details) are not implemented in the HDSv5 backend
- Migration of HDS v4 files to HDF5 is explicitly out of scope; the wrapper only forwards calls, it does not convert files
- HDS primary/secondary locator semantics (datPrmry) may not map cleanly onto HDF5, potentially requiring all locators to be treated as primary or tracked internally in the wrapper
- datRef/datMsg may not report the full path to the file, and datWhere (Fortran API) is likely not workable over HDF5

## Evidence

- [readme] README migration rationale: "Should there be a wrapper HDS library that manages to forward calls to either the native HDS or HDS/HDF5 based on the file that is being opened?"
- [readme] drop-in replacement goal: "The idea is that this library could be dropped in as a direct replacement to HDS and allow the entire Starlink software collection to run, using HDF5 files behind the scenes"
- [methods] EMS error forwarding: "All calls to HDF5 are wrapped by an EMS status check and on error the HDF5 stack is read and stored in EMS error stack."
- [discussion] API coverage: [evidence span withheld — anchor status: not_found]
