---
name: hdf5-file-identifier-tracking
description: Use when reimplementing an API layer over HDF5 whose locators must preserve
  HDS-style primary/secondary semantics, deciding when a file can actually be closed.
when_to_use_negative:
- When the target API has no primary/secondary locator distinction (plain HDF5 treats
  all locators as primary and needs no wrapper-side tracking)
- When migrating HDS v4 files to HDF5 or converting file formats — this skill addresses
  locator lifetime, not data migration
- When a caller genuinely needs hdsLock file-descriptor access, which this wrapper
  cannot provide
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: uthash
  role: Macro-based C hash table keyed by hid_t used to track per-file locator vectors
    and named locator groups
- name: HDF5
  role: Underlying storage layer whose file identifiers are the hash keys and whose
    file-close behaviour the locator bookkeeping controls
- name: HDS (Starlink Hierarchical Data System)
  role: 'Legacy API being emulated: datAnnul, datPrmry, hdsLink, hdsFlush, hdsClose
    semantics are reproduced above HDF5'
  repo: https://github.com/Starlink
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:648171b32419e21d51f5415552c871cbbf9e75cd02dfd48aad56eb1c3e681ebd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HDS (Starlink Hierarchical Data System)
  - uthash
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hdf5-file-identifier-tracking

## Summary

Track per-file allocated HDS locators in a uthash hash table keyed by HDF5 file identifier, tagging each locator primary or secondary so the underlying HDF5 file is closed only when the last primary locator is annulled.

## When to use

Use when reimplementing an API layer over HDF5 whose locators must preserve HDS-style primary/secondary semantics, deciding when a file can actually be closed.

## When NOT to use

- When the target API has no primary/secondary locator distinction (plain HDF5 treats all locators as primary and needs no wrapper-side tracking)
- When migrating HDS v4 files to HDF5 or converting file formats — this skill addresses locator lifetime, not data migration
- When a caller genuinely needs hdsLock file-descriptor access, which this wrapper cannot provide

## Inputs

- HDF5 file identifiers (hid_t) returned from file opens
- HDSLoc locator records created by datFind/datCell/datSlice and top-level hdsOpen calls
- Named locator-group membership from hdsLink calls

## Outputs

- Hash-table entry per hid_t containing a vector of locators with primary/secondary flag fields
- File-closure decisions from datAnnul/hdsClose (close HDF5 file only after last primary locator annulled)
- Group-wise annulment of all locators in a named group via hdsFlush

## How to apply

Declare a global hash table using the uthash macros keyed by the HDF5 file identifier (hid_t); each entry holds a utarray-style vector of locator records, each carrying a primary/secondary flag. Register every newly created locator in its file's vector, defaulting top-level locators to 'primary' and all others to 'secondary' (HDF5 itself treats all locators as primary, so this state must be maintained internally in the wrapper). In datAnnul, scan the vector for the file identifier and close the underlying HDF5 file only when the last primary locator is annulled, automatically annulling any remaining secondary locators at that point; expose datPrmry to promote a secondary locator to primary. Implement hdsLink/hdsFlush above HDF5 as named locator groups tracked in the same hash-plus-vector structure, so hdsFlush annuls every member of a named group in a single command, and implement hdsClose as a thin forward to datAnnul because it is obsolete but still called throughout ADAM.

## Related tools

- **uthash** (Macro-based C hash table keyed by hid_t used to track per-file locator vectors and named locator groups)
- **HDF5** (Underlying storage layer whose file identifiers are the hash keys and whose file-close behaviour the locator bookkeeping controls)
- **HDS (Starlink Hierarchical Data System)** (Legacy API being emulated: datAnnul, datPrmry, hdsLink, hdsFlush, hdsClose semantics are reproduced above HDF5) — https://github.com/Starlink

## Evaluation signals

- Module compiles cleanly as part of the hds-v5 library build
- A C test harness opening a file with multiple locators keeps the file open until the last primary locator is annulled
- Annulling only secondary locators does not close the underlying HDF5 file; datPrmry promotion flips closure behaviour
- hdsFlush annuls every locator in a named group in a single command
- hdsClose behaves identically to datAnnul, keeping ADAM parameter-file callers working

## Limitations

- HDF5 natively has no secondary-locator concept, so all primary/secondary state must be maintained internally in the wrapper
- hdsClose is obsolete and implemented only as a forward to datAnnul because ADAM still calls it
- hdsLock requires access to the underlying file descriptor, which is hard to obtain and no Starlink software calls the routine; hdsFree only acts if hdsLock was called
- datAnnul must also handle copy-back of mmapped arrays (datMap/datUnmap), which complicates closure-time bookkeeping

## Evidence
