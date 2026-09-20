---
name: terminal-emulation-subprocess-control
description: Use when a mature, externally-maintained pipeline (e.g. casapy/CASA routines,
  AMI-reduce) accepts only command-line input and must be incorporated into a fully
  automated Python reduction pipeline without altering the original code.
when_to_use_negative:
- When the target package offers a native scripting interface or can host third-party
  packages directly (e.g. installing astropy into CASA), since terminal emulation
  and output-format-dependent parsing add avoidable fragility.
- When the external tool's output format changes frequently or cannot be guaranteed,
  because fixed parser rules that detect errors may silently break.
- When non-cli, API-level control or return-value access is required — terminal emulation
  only emulates a human typing commands and must parse logged text.
tools:
- name: pexpect
  role: Pure-Python Expect-like module used to spawn the casapy sub-process and emulate
    interactive terminal input/response to expected output patterns
  repo: https://github.com/pexpect/pexpect
- name: drive-casa
  role: Python package that wraps the pexpect-based CasapyProcess interface and provides
    script-generation convenience routines (importuvfits, zero-iteration Clean) for
    CASA routines
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: 'Target external process: image-synthesis subroutines (importuvfits, Clean)
    driven through the emulated terminal'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:530758883ac72dcc47a63b992dc51286a8a14a6c5a99f66c9def6d228d83b47c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pexpect/pexpect
  tools:
  - CASA
  - drive-casa
  - pexpect
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# terminal-emulation-subprocess-control

## Summary

Drive legacy command-line scientific tools (here, the casapy process running NRAO CASA routines) from Python by spawning them in a sub-process and emulating interactive terminal input with pexpect, enabling CASA to be embedded in a fully automated Python reduction pipeline.

## When to use

Use when a mature, externally-maintained pipeline (e.g. casapy/CASA routines, AMI-reduce) accepts only command-line input and must be incorporated into a fully automated Python reduction pipeline without altering the original code.

## When NOT to use

- When the target package offers a native scripting interface or can host third-party packages directly (e.g. installing astropy into CASA), since terminal emulation and output-format-dependent parsing add avoidable fragility.
- When the external tool's output format changes frequently or cannot be guaranteed, because fixed parser rules that detect errors may silently break.
- When non-cli, API-level control or return-value access is required — terminal emulation only emulates a human typing commands and must parse logged text.

## Inputs

- casapy executable available on the system (NRAO CASA)
- UVFITS input files to be converted via importuvfits into a CASA measurement set
- Python script(s) of casapy commands generated programmatically by convenience routines
- casapy log output stream used for error detection

## Outputs

- CASA measurement set produced from UVFITS input via importuvfits
- dirty image produced by a Clean invocation with niter=0
- Python object interface (CasapyProcess-style) returning command results with parsed error reporting
- separated log channels for sent commands vs. process output

## How to apply

Implement a CasapyProcess-style interface class that uses pexpect to spawn the casapy executable as a sub-process and interact with it by emulating interactive terminal input. Route command input and output logging to separate channels so commands sent to casapy remain distinguishable from the process's logged output, and parse the log output with fixed parser rules to detect and report errors. Provide convenience routines that programmatically assemble the casapy script — e.g. an importuvfits call to convert UVFITS input into a CASA measurement set, followed by a Clean invocation with zero iterations so the routine yields the dirty image rather than a deconvolved map. Finally, send the generated script through the spawned casapy process end-to-end and verify the log-parsing/reporting path behaves as designed.

## Related tools

- **pexpect** (Pure-Python Expect-like module used to spawn the casapy sub-process and emulate interactive terminal input/response to expected output patterns) — https://github.com/pexpect/pexpect
- **drive-casa** (Python package that wraps the pexpect-based CasapyProcess interface and provides script-generation convenience routines (importuvfits, zero-iteration Clean) for CASA routines) — https://github.com/timstaley/drive-casa
- **CASA** (Target external process: image-synthesis subroutines (importuvfits, Clean) driven through the emulated terminal)

## Examples

```
import pexpect; casapy = pexpect.spawn('casapy', timeout=None)
```

## Evaluation signals

- The spawned casapy process completes the generated script (importuvfits + zero-iteration Clean) end-to-end without an unreported exception.
- Commands sent to casapy and the process's logged output appear on separate channels and can be distinguished in the logs.
- The log parser correctly flags injected/known errors from casapy output and reports them rather than swallowing them.
- The output measurement set is a valid CASA measurement set and the resulting image is a dirty image (no deconvolution from niter=0).
- The interface works without any modification to the underlying CASA/casapy code.

## Limitations

- Output-parsing in terminal emulation is brittle: if the tool changes to another output layout which is still human-friendly but breaks the fixed parser rules, error detection fails.
- The approach emulates interactive terminal input rather than calling an API, so return values must be recovered by parsing logged text.
- Only requires a working Python installation (v2.6 or greater) as a prerequisite for pexpect, but is tied to the specific output formats of the wrapped tool.

## Evidence

- [discussion] parser brittleness caveat: [evidence span withheld — anchor status: not_found]
- [readme] drive-casa purpose: [evidence span withheld — anchor status: external_doc]
- [readme] pexpect mechanism: [evidence span withheld — anchor status: external_doc]
