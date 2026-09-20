---
name: process-log-output-parsing
description: Use when driving a command-line scientific tool (e.g. casapy) from Python
  via pexpect and you need to detect and report errors automatically from the tool's
  logged output rather than by human inspection.
when_to_use_negative:
- When the external tool's output format is unstable or about to change — the fixed
  parser rules break if output is re-laid-out in a still human-friendly but different
  form
- When interacting with tools outside the terminal-emulation pattern, e.g. raw AMI-REDUCE
  calibration handled by drive-ami rather than casapy log parsing
- When a native programmatic interface exists that avoids terminal emulation and output
  parsing altogether
tools:
- name: drive-casa
  role: Python interface package whose CasapyProcess-style class spawns casapy, routes
    command/output logging, and parses casapy log output for errors
  repo: https://github.com/timstaley/drive-casa
- name: pexpect
  role: Library used to emulate terminal interaction — spawning the child casapy process
    and responding to expected patterns in its output
  repo: https://github.com/pexpect/pexpect
- name: CASA
  role: Target NRAO package whose subroutines (importuvfits, Clean) are driven externally
    and whose log output is parsed for errors
  repo: http://casa.nrao.edu
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:0e40cb17ed9419e80a80b230883f64e40de3137c35cfab63f6119a6e55cc3f81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/drive-casa
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
yes_rate: 0.6363636363636364
n_claims: 11
---

# process-log-output-parsing

## Summary

Implements automated error detection and reporting by parsing the log output of a spawned casapy process, as used by the drive-casa Python interface so that CASA subroutines can be driven from a fully automated reduction pipeline.

## When to use

Use when driving a command-line scientific tool (e.g. casapy) from Python via pexpect and you need to detect and report errors automatically from the tool's logged output rather than by human inspection.

## When NOT to use

- When the external tool's output format is unstable or about to change — the fixed parser rules break if output is re-laid-out in a still human-friendly but different form
- When interacting with tools outside the terminal-emulation pattern, e.g. raw AMI-REDUCE calibration handled by drive-ami rather than casapy log parsing
- When a native programmatic interface exists that avoids terminal emulation and output parsing altogether

## Inputs

- Log output stream from a spawned casapy sub-process
- Python command strings sent to the casapy process (e.g. importuvfits, Clean invocations)
- casapy scripts assembled programmatically from UVFITS input files

## Outputs

- Parsed error reports / detected-error flags derived from the casapy log
- Separately channelled logs of commands sent vs. output received
- CASAmasurement-set conversion and dirty-image products produced under the monitored process

## How to apply

Spawn the casapy executable in a sub-process with pexpect and interact with it by emulating interactive terminal input. Route command input and output logging to separate channels (e.g. distinct logfile attributes for sent commands vs. returned output) so that commands sent to casapy are distinguishable from the process's logged output. Implement fixed parser rules against the output format produced by casapy to detect errors and report them to the calling pipeline; the parser necessarily depends on casapy's specific log layout. Exercise the path end-to-end by sending a generated casapy script (e.g. an importuvfits call converting UVFITS to a measurement set, followed by a Clean with zero iterations) through the spawned process and confirming that errors are detected and reported correctly.

## Related tools

- **drive-casa** (Python interface package whose CasapyProcess-style class spawns casapy, routes command/output logging, and parses casapy log output for errors) — https://github.com/timstaley/drive-casa
- **pexpect** (Library used to emulate terminal interaction — spawning the child casapy process and responding to expected patterns in its output) — https://github.com/pexpect/pexpect
- **CASA** (Target NRAO package whose subroutines (importuvfits, Clean) are driven externally and whose log output is parsed for errors) — http://casa.nrao.edu

## Examples

```
import pexpect; casapy_proc = pexpect.spawn('casapy')
```

## Evaluation signals

- Commands sent to casapy appear in a separate logging channel from the process's own output, so the two streams are distinguishable
- Injected or naturally occurring casapy errors in the log are detected and reported by the fixed parser rules
- End-to-end run of a generated script (importuvfits UVFITS-to-measurement-set, then Clean with zero iterations) yields the dirty image and a clean error-report path
- Parser output matches known-good casapy log transcripts for both success and failure cases

## Limitations

- Fixed parser rules depend on the output format produced by casapy; a changed layout that is still human-friendly but different breaks the parser
- Output-parsing in terminal emulation is inherently brittle compared with a native programmatic interface
- The approach requires only a working Python installation (v2.6 or greater) for the pexpect dependency, implying an older-Python compatibility envelope

## Evidence

- [discussion] Parser brittleness: [evidence span withheld — anchor status: not_found]
- [readme] pexpect behaviour: [evidence span withheld — anchor status: external_doc]
- [readme] drive-casa purpose: [evidence span withheld — anchor status: external_doc]
