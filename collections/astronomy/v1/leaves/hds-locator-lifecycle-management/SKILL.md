---
name: hds-locator-lifecycle-management
description: Use when reimplementing or porting the HDS API onto HDF5 and you must
  decide when a file can be closed given multiple outstanding locators, or implement
  locator-group behavior (hdsLink/hdsFlush) and datPrmry promotion.
when_to_use_negative:
- When HDF5's native locator model suffices — the README notes that 'In HDF5 it seems
  that all locators are primary', so no wrapper tracking is needed if you can assert
  all locators are primary
- When the caller requires hdsLock semantics — obtaining the underlying file descriptor
  is hard and no Starlink software calls the routine
- When migrating HDS v4 files to HDF5 — migration is explicitly out of scope of this
  implementation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: HDF5
  role: Underlying storage library whose file handles must remain open until the last
    primary HDS locator is annulled
  repo: https://github.com/Starlink/hds-v5
- name: uthash
  role: C hash-table macros used to key the global locator table by HDF5 file identifier,
    with a vector of locators per file
  repo: https://github.com/Starlink/hds-v5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:c7651db2e61c59d84641518dbc391518729961bf49a695bbc5897ec1d003a1c3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/hds-v5
  tools:
  - HDF5
  - uthash
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# HDS locator lifecycle management

## Summary

Tracks allocated HDS locators in a uthash-backed hash table keyed by the HDF5 file identifier, distinguishing primary from secondary locators so a file is closed only when the last primary locator is annulled. Implements HDS locator-group semantics (hdsLink/hdsFlush) above HDF5.

## When to use

Use when reimplementing or porting the HDS API onto HDF5 and you must decide when a file can be closed given multiple outstanding locators, or implement locator-group behavior (hdsLink/hdsFlush) and datPrmry promotion.

## When NOT to use

- When HDF5's native locator model suffices — the README notes that 'In HDF5 it seems that all locators are primary', so no wrapper tracking is needed if you can assert all locators are primary
- When the caller requires hdsLock semantics — obtaining the underlying file descriptor is hard and no Starlink software calls the routine
- When migrating HDS v4 files to HDF5 — migration is explicitly out of scope of this implementation

## Inputs

- HDS locators (HDSLoc structs) obtained via hdsOpen/datFind/datCell/datSlice, each carrying an HDF5 identifier and a primary/secondary flag
- HDF5 file identifiers (hid_t) for opened files
- Named locator-group identifiers for hdsLink/hdsFlush

## Outputs

- Global hash table entry per file identifier mapping to a vector of allocated locators with primary/secondary flags
- File close events issued only after the last primary locator is annulled
- Named group membership records enabling hdsFlush to annul all group members in one command

## How to apply

Declare a global hash table with the uthash macros keyed by the HDF5 file identifier (hid_t), each entry holding a dynamic array (e.g. utarray vector) of locator records each carrying a primary/secondary flag. Register every newly created locator in its file's vector, defaulting top-level locators to 'primary' and all others to 'secondary'; expose datPrmry to promote a secondary locator to primary. In datAnnul, scan the vector for the file identifier and close the underlying HDF5 file only when the last primary locator is annulled, automatically annulling any remaining secondary locators at that point. Implement hdsLink/hdsFlush above HDF5 as named locator groups tracked in the same hash-plus-vector structure, so hdsFlush processes (annuls) all members of a named group in a single command. Implement hdsClose as a thin forward to datAnnul, since it is documented as obsolete but still called throughout ADAM.

## Related tools

- **HDF5** (Underlying storage library whose file handles must remain open until the last primary HDS locator is annulled) — https://github.com/Starlink/hds-v5
- **uthash** (C hash-table macros used to key the global locator table by HDF5 file identifier, with a vector of locators per file) — https://github.com/Starlink/hds-v5

## Examples

```
datAnnul( loc ); /* hdsClose simply forwards here: annul locators, close file only if last primary */
```

## Evaluation signals

- A C test harness opens a file with multiple locators and verifies the underlying HDF5 file stays open until the last primary locator is annulled
- The test verifies that annulled remaining secondary locators do not close the file prematurely
- The test verifies hdsFlush annuls every locator in a named group in a single command
- datPrmry successfully promotes a secondary locator to primary and the close behavior changes accordingly
- The module compiles cleanly and all but a handful of the approximately 150 HDS API functions are implemented

## Limitations

- Only one routine in NDF (ndf1_htop.f) ever sets a primary locator to secondary; all others set to primary, so empirical coverage of secondary-locator behavior is narrow
- HDF5 has no native primary/secondary locator concept, so the distinction must be tracked entirely inside the wrapper layer
- datMap memory-mapped arrays complicate lifecycle: datAnnul must handle copy-back, and program exit before datUnmap must be reasoned about
- datParen does not currently work out when it hits the top of the group hierarchy

## Evidence

- [discussion] API coverage criterion used for validation: [evidence span withheld — anchor status: not_found]
