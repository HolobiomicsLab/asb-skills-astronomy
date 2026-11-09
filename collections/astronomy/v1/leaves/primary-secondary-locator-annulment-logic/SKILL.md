---
name: primary-secondary-locator-annulment-logic
description: Use when implementing or porting an HDS-style API onto HDF5 (as in Starlink/hds-v5)
  and file-lifetime semantics must depend on distinguishing primary from secondary
  locators, or when named locator groups (hdsLink/hdsFlush) must be tracked.
when_to_use_negative:
- When the target API is pure HDF5 with no HDS compatibility requirement — HDF5 has
  no secondary-locator concept, so this wrapper bookkeeping is unnecessary overhead
- When migrating HDS v4 files to HDF5 — the hds-v5 library explicitly does not yet
  address HDS v4 file migration
- When a routine needs the underlying file descriptor (hdsLock) — that access is hard
  to obtain through this locator-tracking layer and no Starlink software calls it
tools:
- name: uthash
  role: uthash macros implement the hash table keyed by HDF5 file identifier that
    tracks locator group membership
- name: HDF5
  role: underlying storage library; locators wrap HDF5 file/dataset handles and the
    wrapper must decide when the HDF5 file can actually be closed
- name: HDS (Starlink Hierarchical Data System)
  role: source of the primary/secondary locator semantics being reproduced (datAnnul,
    datPrmry, hdsClose, hdsFlush API)
  repo: https://github.com/Starlink/hds-v5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:b9974c2ff112a752029484e6988b4443fef8bb3fb881b9eb82fa71aef2b50182
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/hds-v5
  tools:
  - HDF5
  - HDS (Starlink Hierarchical Data System)
  - uthash
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# primary-secondary-locator-annulment-logic

## Summary

Tracks HDS locators (primary vs secondary) in a uthash hash table keyed by HDF5 file identifier so the underlying HDF5 file is closed only when the last primary locator is annulled, mirroring HDS file-lifetime semantics on top of HDF5.

## When to use

Use when implementing or porting an HDS-style API onto HDF5 (as in Starlink/hds-v5) and file-lifetime semantics must depend on distinguishing primary from secondary locators, or when named locator groups (hdsLink/hdsFlush) must be tracked.

## When NOT to use

- When the target API is pure HDF5 with no HDS compatibility requirement — HDF5 has no secondary-locator concept, so this wrapper bookkeeping is unnecessary overhead
- When migrating HDS v4 files to HDF5 — the hds-v5 library explicitly does not yet address HDS v4 file migration
- When a routine needs the underlying file descriptor (hdsLock) — that access is hard to obtain through this locator-tracking layer and no Starlink software calls it

## Inputs

- HDS locators (HDSLoc handles) obtained from hdsOpen, datFind, datCell, datSlice, datVec
- HDF5 file identifiers (hid_t) for open files
- Primary/secondary flag on each locator record
- Named locator group identifiers for hdsLink/hdsFlush

## Outputs

- Global uthash table mapping hid_t -> utarray of locator records with primary/secondary flags
- datAnnul behavior: file closed only on annulment of the last primary locator, with remaining secondaries annulled automatically
- datPrmry promotion of a secondary locator to primary
- hdsFlush annulment of every locator in a named group

## How to apply

Declare a global hash table using the uthash macros keyed by the HDF5 file identifier (hid_t); each entry holds a vector (utarray) of locator records with a primary/secondary flag field. Register every newly created locator in the file's vector, defaulting top-level locators to 'primary' and all others (datCell/datSlice/datFind results) to 'secondary', and expose datPrmry to promote a secondary locator to primary. In datAnnul, scan the vector for the file identifier and close the underlying HDF5 file only when the last primary locator is annulled, automatically annulling any remaining secondary locators at that point — because HDF5 itself treats all locators as primary, this bookkeeping must be done entirely in the wrapper layer. Implement hdsLink/hdsFlush as named locator groups in the same hash-plus-vector structure so hdsFlush annuls all members of a group in one command, and implement hdsClose as a thin forward to datAnnul since it is obsolete but still called throughout ADAM.

## Related tools

- **uthash** (uthash macros implement the hash table keyed by HDF5 file identifier that tracks locator group membership)
- **HDF5** (underlying storage library; locators wrap HDF5 file/dataset handles and the wrapper must decide when the HDF5 file can actually be closed)
- **HDS (Starlink Hierarchical Data System)** (source of the primary/secondary locator semantics being reproduced (datAnnul, datPrmry, hdsClose, hdsFlush API)) — https://github.com/Starlink/hds-v5

## Examples

```
datPrmry( locator, &status ); /* promote secondary locator to primary before datAnnul decides file closure */
```

## Evaluation signals

- Open a file with multiple locators and verify the HDF5 file stays open until the last primary locator is annulled
- Verify that annulment of the last primary locator automatically annuls all remaining secondary locators
- Verify datPrmry promotes a secondary locator to primary and thereby changes the file-close decision
- Verify hdsFlush annuls every locator registered in a named group in a single command
- Module compiles cleanly, consistent with the paper's criterion that all but a handful of the ~150 HDS API functions are implemented

## Limitations

- HDF5 does not natively distinguish primary from secondary locators; the semantics are emulated entirely in the wrapper, which is why the README asks 'Does HDF5 have the concept of a secondary locator?'
- In Starlink software only one routine (ndf1_htop.f) ever sets a primary locator to secondary — the mechanism is rarely exercised in the wild
- datAnnul must also handle copy-back for memory-mapped arrays (datMap uses an anonymous mmap copied back at datUnmap), which complicates annulment on the file root
- If the program ends before datUnmap, behavior of mapped arrays tied to annulled locators is unresolved

## Evidence
