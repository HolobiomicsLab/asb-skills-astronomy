---
name: radio-interferometry-dirty-image-generation
description: Use when a Python reduction pipeline needs to drive NRAO CASA routines
  non-interactively — e.g. converting UVFITS to a measurement set and generating a
  dirty image without deconvolution — from a generic Python process.
when_to_use_negative:
- When you need a deconvolved (cleaned) image rather than a dirty image — the zero-iteration
  Clean invocation deliberately skips deconvolution
- When CASA can be driven natively (e.g. scripts run inside the CASA environment)
  and no external-process terminal emulation is needed
- When the external tool's output format changes, breaking fixed parser rules — the
  paper notes output-parsing in terminal emulation is brittle to format changes
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: drive-casa
  role: Python package providing the CasapyProcess-style pexpect interface for scripting
    CASA pipeline routines and assembling the importuvfits + zero-iteration Clean
    script
  repo: https://github.com/timstaley/drive-casa
- name: pexpect
  role: Pure-Python Expect-like library used to spawn the casapy child process and
    emulate interactive terminal input
  repo: https://github.com/pexpect/pexpect
- name: CASA
  role: NRAO image-synthesis routines (importuvfits, Clean) whose subroutines are
    automated from a generic Python process
- name: chimenea
  role: 'Downstream consumer: uses drive-casa to build upon CASA subroutines for automated
    imaging of multi-epoch radio-synthesis data'
  repo: https://github.com/timstaley/chimenea
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:158420a583af09ca71b2edbfa1d3f4ba777fba39e3a1963f17ec9b2928c2837f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/drive-casa
  tools:
  - CASA
  - chimenea
  - drive-casa
  - pexpect
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# radio-interferometry-dirty-image-generation

## Summary

Uses the drive-casa package (pexpect-based terminal emulation of the casapy process) to script CASA routines that convert a UVFITS input into a measurement set and produce a dirty image by running Clean with zero iterations, enabling CASA image-synthesis steps inside a fully automated Python pipeline.

## When to use

Use when a Python reduction pipeline needs to drive NRAO CASA routines non-interactively — e.g. converting UVFITS to a measurement set and generating a dirty image without deconvolution — from a generic Python process.

## When NOT to use

- When you need a deconvolved (cleaned) image rather than a dirty image — the zero-iteration Clean invocation deliberately skips deconvolution
- When CASA can be driven natively (e.g. scripts run inside the CASA environment) and no external-process terminal emulation is needed
- When the external tool's output format changes, breaking fixed parser rules — the paper notes output-parsing in terminal emulation is brittle to format changes

## Inputs

- UVFITS file (radio-interferometry visibility data)
- casapy executable available in the environment
- Python script assembling the CASA commands (importuvfits, Clean with niter=0)

## Outputs

- CASA measurement set produced by importuvfits
- Dirty image (Clean run with zero iterations, no deconvolution)
- Parsed log output with commands distinguished from casapy's logged output and errors detected/reported

## How to apply

Implement a CasapyProcess-style interface class that uses pexpect to spawn the casapy executable in a sub-process and interact with it by emulating interactive terminal input. Route command input and output logging to separate channels so commands sent to casapy are distinguishable from the process's logged output, and parse the casapy log output to detect and report errors, recognising that fixed parser rules depend on casapy's output format. Build the casapy script programmatically with convenience routines: first an importuvfits call to convert the UVFITS input into a CASA measurement set, then a Clean invocation performed with zero iterations so the routine yields the dirty image rather than a deconvolved map. Send the generated script through the spawned casapy process and confirm the log-parsing/reporting path behaves as designed; the pexpect-based approach works because it requires no alterations to the original CASA code.

## Related tools

- **drive-casa** (Python package providing the CasapyProcess-style pexpect interface for scripting CASA pipeline routines and assembling the importuvfits + zero-iteration Clean script) — https://github.com/timstaley/drive-casa
- **pexpect** (Pure-Python Expect-like library used to spawn the casapy child process and emulate interactive terminal input) — https://github.com/pexpect/pexpect
- **CASA** (NRAO image-synthesis routines (importuvfits, Clean) whose subroutines are automated from a generic Python process)
- **chimenea** (Downstream consumer: uses drive-casa to build upon CASA subroutines for automated imaging of multi-epoch radio-synthesis data) — https://github.com/timstaley/chimenea

## Examples

```
from drive_casa import casapy; cb = casapy.CasapyProcess(log_to_term=True, log_to_file=False, outfiledir='imgdir'); cb.send-command('importuvfits(fitsfile=\'raw.uvf\', vis=\'msdir/measurement_set.ms\')')
```

## Evaluation signals

- The spawned casapy process accepts the generated script end-to-end and the log output contains the expected importuvfits and Clean invocations
- The output image corresponds to a dirty image (Clean run with zero iterations) rather than a deconvolved map
- Command input and casapy's logged output remain on separate, distinguishable channels
- Errors injected into the casapy output are detected and reported by the log parser
- The resulting measurement set is a valid CASA measurement set readable by subsequent CASA routines

## Limitations

- Output-parsing in terminal emulation is brittle to changes in output format: a human-friendly re-layout of casapy output can break the fixed parser rules
- The pexpect/terminal-emulation route requires a working Python installation (v2.6 or greater per the paper) and only wraps command-line interaction, not a native API
- A zero-iteration Clean yields only the dirty image; the paper separately warns that repeated Clean processing (as in chimenea's iterative RMS re-estimation) can add significant processing time for large datasets

## Evidence

- [discussion] brittle output parsing: [evidence span withheld — anchor status: not_found]
