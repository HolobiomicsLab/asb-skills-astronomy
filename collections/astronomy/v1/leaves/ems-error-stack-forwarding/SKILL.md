---
name: ems-error-stack-forwarding
description: Use when layering a re-implemented API (e.g. HDSv5 over HDF5) onto a
  backend that maintains its own error stack, so backend failures must be reported
  through the host error system (EMS).
when_to_use_negative:
- Code calls native HDSv4 directly — it already reports through EMS, so there is no
  separate backend error stack to drain.
- The consumer does not use Starlink error handling (e.g. a pure-HDF5 reader such
  as h5py); forwarding into EMS provides no benefit there.
- The task is migrating HDS v4 files to HDF5 — the README explicitly defers migration,
  and this skill covers error reporting only on the HDSv5 path.
tools:
- name: HDSv5 (libhdsh5)
  role: Wrapper layer whose dat*/hds* routines guard each HDF5 backend call with an
    EMS status check
  repo: https://github.com/Starlink/hds-v5
- name: HDF5
  role: Backend library whose native error stack is read and stored into EMS on failure
- name: EMS
  role: Starlink Error Message Service that receives the drained HDF5 stack and reports
    it to callers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:2b87eae32ac32571f12c9b13e149ea0d55e53a0c795be44dac113398d1990dff
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/hds-v5
  tools:
  - EMS
  - HDF5
  - HDS
  - HDSv5 (libhdsh5)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ems-error-stack-forwarding

## Summary

A porting technique from the Starlink HDSv5 re-implementation (libhdsh5): every call into the HDF5 backend is guarded by an EMS status check, and on failure the HDF5 error stack is read and stored in the EMS error stack so Starlink applications see backend errors through their native error-handling convention.

## When to use

Use when layering a re-implemented API (e.g. HDSv5 over HDF5) onto a backend that maintains its own error stack, so backend failures must be reported through the host error system (EMS).

## When NOT to use

- Code calls native HDSv4 directly — it already reports through EMS, so there is no separate backend error stack to drain.
- The consumer does not use Starlink error handling (e.g. a pure-HDF5 reader such as h5py); forwarding into EMS provides no benefit there.
- The task is migrating HDS v4 files to HDF5 — the README explicitly defers migration, and this skill covers error reporting only on the HDSv5 path.

## Inputs

- HDF5 backend calls (e.g. H5Dwrite in datPut) issued from the HDSv5 wrapper layer
- Starlink EMS STATUS argument (INTEGER) of each HDS routine
- HDF5 error stack contents after a failed backend call

## Outputs

- Populated EMS error stack carrying forwarded HDF5 error messages
- Non-zero Starlink STATUS returned to the HDS API caller on backend failure
- Wrapper source in which every HDF5 call site is guarded by an EMS status check

## How to apply

Treat every call into the HDF5 backend from the HDSv5 wrapper — the dat* and hds* routines implementing the ~150-function HDS API — as a guarded call site: invoke the H5 routine, then check its status. On failure, read the full HDF5 error stack (HDF5 maintains its own error stack, separate from Starlink's) and store its contents in the EMS error stack before returning a non-zero STATUS, so the caller's EMS-based handling receives the backend diagnostics. Apply the wrapping uniformly rather than selectively: the paper reports that all calls to HDF5 are wrapped, so no unguarded H5* call site should remain in the wrapper. The rationale is that HDS and the Starlink software use EMS (Rees et al., 2008) for error handling; without forwarding, HDF5 failures would be invisible to ADAM applications and NDF routines. A key decision point is draining the whole backend stack rather than only the top-level return code, preserving context such as an H5Dwrite failure after datPut performs its string type conversion.

## Related tools

- **HDSv5 (libhdsh5)** (Wrapper layer whose dat*/hds* routines guard each HDF5 backend call with an EMS status check) — https://github.com/Starlink/hds-v5
- **HDF5** (Backend library whose native error stack is read and stored into EMS on failure)
- **EMS** (Starlink Error Message Service that receives the drained HDF5 stack and reports it to callers)

## Evaluation signals

- Static audit of the wrapper confirms an EMS status check guards every HDF5 call site; no bare H5* call remains.
- Inducing a backend failure — e.g. a type conversion in datPut before H5Dwrite fails — yields a non-zero Starlink STATUS with the HDF5 error text present in the EMS error stack.
- The EMS stack receives the actual HDF5 stack contents read from the backend, not merely a generic wrapper message, preserving the diagnostic chain.
- On the success path STATUS stays at zero and no spurious EMS messages are emitted by the wrapping.

## Limitations

- Only errors raised at HDF5 call sites are captured; wrapper-level issues flagged in the README (datWhere is 'probably not going to work'; datParen cannot work out when it hits the top of the group hierarchy) never appear in the HDF5 stack.
- Inapplicable to native HDSv4, which already reports through EMS — there is no second error stack to drain.
- Diagnostic content can be incomplete: datRef and datMsg currently do not report the full path to a file, unlike the HDS library, so even correctly forwarded messages may lack path context.

## Evidence

- [methods] All HDF5 backend calls in the HDSv5 wrapper are wrapped by EMS status checks that forward the HDF5 error stack into EMS: "All calls to HDF5 are wrapped by an EMS status check and on error the HDF5 stack is read and stored in EMS error stack."
- [readme] The two libraries use separate error mechanisms: HDF5 has its own error stack while HDS/Starlink uses EMS: "HDF5 maintains its own error stack and HDS (and Starlink) uses EMS."
- [intro] HDS and Starlink software rely on EMS for error handling: "HDS uses the Starlink Error Message Service (EMS; Rees et al., 2008) for error handling"
