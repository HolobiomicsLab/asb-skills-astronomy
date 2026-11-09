---
name: casa-command-script-generation
description: Use when a Python pipeline must drive NRAO CASA subroutines non-interactively
  — e.g. converting a UVFITS input to a measurement set and producing dirty images
  as part of automated radio-synthesis imaging.
when_to_use_negative:
- The target pipeline has a native, non-interactive scripting interface (e.g. Python
  callable inside CASA) that avoids terminal-emulation parsing fragility.
- The workflow needs deconvolved (restored) maps rather than dirty images — the zero-iteration
  Clean convenience routine deliberately yields only the dirty image.
- The external tool's human-friendly log format changes, breaking the fixed parser
  rules; the article warns output-parsing is brittle to format changes.
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: drive-casa
  role: Python interface layer that spawns casapy via pexpect, generates casapy scripts
    (importuvfits, zero-iteration Clean), and parses logs for errors
  repo: https://github.com/timstaley/drive-casa
- name: pexpect
  role: Library used to emulate terminal interaction with the spawned casapy sub-process
  repo: https://github.com/pexpect/pexpect
- name: CASA
  role: External tool providing the image-synthesis subroutines (importuvfits, Clean)
    driven from the generated scripts
- name: chimenea
  role: Downstream consumer that builds its automated imaging algorithm on CASA subroutines
    through the drive-casa interface layer
  repo: https://github.com/timstaley/chimenea
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:438803348fc835d1ddcb40ff4fd4b2e9e0360ef1d13e4881040901882de4f45f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/drive-casa
  tools:
  - CASA
  - chimenea
  - drive-casa
  - pexpect
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# casa-command-script-generation

## Summary

Programmatically assemble and dispatch CASA command scripts (e.g. importuvfits followed by a zero-iteration Clean for dirty imaging) to a spawned casapy process via the drive-casa pexpect-based terminal-emulation interface, enabling CASA routines inside fully automated Python reduction pipelines.

## When to use

Use when a Python pipeline must drive NRAO CASA subroutines non-interactively — e.g. converting a UVFITS input to a measurement set and producing dirty images as part of automated radio-synthesis imaging.

## When NOT to use

- The target pipeline has a native, non-interactive scripting interface (e.g. Python callable inside CASA) that avoids terminal-emulation parsing fragility.
- The workflow needs deconvolved (restored) maps rather than dirty images — the zero-iteration Clean convenience routine deliberately yields only the dirty image.
- The external tool's human-friendly log format changes, breaking the fixed parser rules; the article warns output-parsing is brittle to format changes.

## Inputs

- UVFITS-format input datafiles (converted via importuvfits into a CASA measurement set)
- CASA subroutine specifications (e.g. importuvfits, Clean with niter=0)
- casapy executable available in a sub-process

## Outputs

- CASA measurement set produced by importuvfits
- Dirty image (Clean run with zero iterations)
- Command and output log streams with casapy errors detected via fixed parser rules

## How to apply

Implement a CasapyProcess-style interface class that uses pexpect to spawn the casapy executable as a sub-process and emulate interactive terminal input, routing command input and logged output to separate channels. Parse the casapy log output with fixed parser rules tuned to casapy's output format to detect and report errors. Build convenience routines that programmatically assemble the casapy script — start with an importuvfits call to convert UVFITS into a CASA measurement set, then add a Clean invocation with zero iterations so the routine yields the dirty image rather than a deconvolved map. Finally, send the generated script through the spawned casapy process end-to-end and check that log parsing and error reporting behave as designed. This approach requires no alterations to the original CASA code, which is why pexpect terminal emulation was chosen over altering the tool itself.

## Related tools

- **drive-casa** (Python interface layer that spawns casapy via pexpect, generates casapy scripts (importuvfits, zero-iteration Clean), and parses logs for errors) — https://github.com/timstaley/drive-casa
- **pexpect** (Library used to emulate terminal interaction with the spawned casapy sub-process) — https://github.com/pexpect/pexpect
- **CASA** (External tool providing the image-synthesis subroutines (importuvfits, Clean) driven from the generated scripts)
- **chimenea** (Downstream consumer that builds its automated imaging algorithm on CASA subroutines through the drive-casa interface layer) — https://github.com/timstaley/chimenea

## Evaluation signals

- The generated casapy script contains a valid importuvfits call that successfully converts the UVFITS input into a CASA measurement set.
- The Clean invocation in the script is run with zero iterations, so the output image is the dirty image rather than a deconvolved map.
- Command input and casapy log output appear on separate channels, with commands clearly distinguishable from the process's logged output.
- Deliberately induced errors in the casapy log are detected and reported by the fixed parser rules.
- The full generated script executes end-to-end through the spawned casapy process without unhandled failures.

## Limitations

- Output-parsing is brittle: if casapy's output format changes to another human-friendly layout, the fixed parser rules may break.
- The convenience routine only produces the dirty image (zero-iteration Clean); deconvolved imaging requires additional logic (as in chimenea's iterative RMS-estimation pipeline).
- drive-casa requires only a working Python installation (v2.6 or greater) as a prerequisite, but no version numbers are given for the packages described.

## Evidence

- [discussion] pexpect-based terminal emulation of casapy: "terminal emulation: spawn the external tool in a sub-process, but then interact with it by emulating interactive terminal input."
- [discussion] parser brittleness caveat: [evidence span withheld — anchor status: not_found]
- [readme] drive-casa purpose per README: [evidence span withheld — anchor status: external_doc]
