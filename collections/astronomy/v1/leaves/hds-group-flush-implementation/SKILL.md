---
name: hds-group-flush-implementation
description: 'Use when reimplementing HDS file-close semantics over HDF5: you must
  decide when an HDF5 file can be closed given multiple locators, named groups, and
  datPrmry promotions.'
when_to_use_negative:
- The application never uses locator groups or secondary locators (e.g. only one top-level
  locator per file) — plain HDF5 open/close semantics suffice
- The target is native HDF5 code with no HDS API compatibility requirement — the wrapper
  table adds overhead with no benefit
- hdsLock/hdsFree-style file-descriptor access is needed; the README notes the underlying
  file descriptor is hard to obtain and no Starlink software calls hdsLock
tools:
- name: uthash
  role: uthash macros implement the hash table keyed by HDF5 file identifier that
    tracks group membership and locator vectors
- name: HDF5
  role: Underlying storage layer; all locator bookkeeping is implemented above HDF5
    because HDF5 treats all locators as primary
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:b6e23952ee7e8c448330ffd2c23c72feb87ca37042b0187e5637b38dc5eddd66
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - uthash
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# hds-group-flush-implementation

## Summary

Implements the HDS named-locator-group mechanism (hdsLink/hdsFlush) and the primary/secondary locator tracking on top of HDF5, which has no native concept of locator groups or secondary locators.

## When to use

Use when reimplementing HDS file-close semantics over HDF5: you must decide when an HDF5 file can be closed given multiple locators, named groups, and datPrmry promotions.

## When NOT to use

- The application never uses locator groups or secondary locators (e.g. only one top-level locator per file) — plain HDF5 open/close semantics suffice
- The target is native HDF5 code with no HDS API compatibility requirement — the wrapper table adds overhead with no benefit
- hdsLock/hdsFree-style file-descriptor access is needed; the README notes the underlying file descriptor is hard to obtain and no Starlink software calls hdsLock

## Inputs

- HDF5 file identifier (hid_t) returned by an HDS-level file open (datAnnul/hdsOpen path)
- HDSLoc locator records created by top-level opens, datFind, datCell, datSlice, datVec
- Named group identifiers passed to hdsLink/hdsFlush
- datPrmry promotion requests on secondary locators

## Outputs

- Global uthash table mapping each hid_t to a vector of locators with primary/secondary flags
- Named locator groups consumable by a single hdsFlush command
- Correct file-closure behavior: HDF5 file closed only on annulment of the last primary locator

## How to apply

Declare a global hash table using the uthash macros keyed by the HDF5 file identifier (hid_t), with each entry holding a dynamic array of locator records that each carry a primary/secondary flag. Register every newly created locator in its file's vector, defaulting top-level locators to 'primary' and all others to 'secondary'; datPrmry promotes a secondary locator to primary. In datAnnul, scan the vector for the file identifier and close the underlying HDF5 file only when the last primary locator is annulled, automatically annulling any remaining secondary locators at that point. Implement hdsLink/hdsFlush entirely above HDF5 as named locator groups tracked in the same hash-plus-vector structure, so hdsFlush processes all members of a named group in a single command. Implement hdsClose as a thin forward to datAnnul, since it is documented as obsolete but still called throughout ADAM. Because HDF5 treats all locators as primary, this internal tracking is the only way to reproduce HDS's rule that annulling a secondary locator never keeps-or-closes the file on its own.

## Related tools

- **uthash** (uthash macros implement the hash table keyed by HDF5 file identifier that tracks group membership and locator vectors)
- **HDF5** (Underlying storage layer; all locator bookkeeping is implemented above HDF5 because HDF5 treats all locators as primary)

## Examples

```
hdsFlush(file_locator, "group_name", &status);  /* annuls every locator linked into the named group */
```

## Evaluation signals

- A C test harness opens one file with multiple locators and verifies the underlying HDF5 file stays open until the last primary locator is annulled
- Annuling a secondary locator alone must NOT close the file; datPrmry promotion must change that outcome
- hdsFlush on a named group must annul every locator registered in that group in a single command
- hdsClose must behave identically to datAnnul (thin forward), matching legacy ADAM call sites
- Consistency with the paper's criterion that all but a handful of the ~150 HDS API functions are implemented

## Limitations

- HDF5 has no native secondary-locator concept, so the primary/secondary distinction exists only inside the wrapper's bookkeeping
- Only one routine in NDF (ndf1_htop.f) ever demotes a primary locator to secondary, so demotion paths are lightly exercised
- datRef/datMsg do not yet report the full path to the file, which can complicate diagnostics tied to locator bookkeeping

## Evidence

- [discussion] All but a handful of the approximately 150 HDS API functions have been implemented, with the remaining few querying low-level implementation details.: [evidence span withheld — anchor status: not_found]
