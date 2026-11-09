---
name: python-interface-wrapping-of-legacy-cli
description: Use when you need to call a legacy or externally-maintained command-line
  pipeline (e.g. CASA's casapy, AMI-reduce) from Python scripts without modifying
  the original code, and no native Python API exists.
when_to_use_negative:
- When the external tool offers a native, scriptable Python API that can be imported
  directly — installing packages into CASA (e.g. via the astropy-into-CASA route)
  is preferred over deprecated executable-wrapping approaches.
- When the wrapped tool's output format is unstable across versions, since fixed parser
  rules break if the output layout changes to another human-friendly but incompatible
  format.
- When the pipeline must deconvolve rather than image — the convenience routine's
  zero-iteration Clean yields only the dirty image by design.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: pexpect
  role: Pure-Python Expect-like module used to emulate terminal interaction with the
    spawned casapy sub-process
  repo: https://github.com/pexpect/pexpect
- name: drive-casa
  role: The resulting Python package providing an interface layer to CASA subroutines
    via terminal emulation
  repo: https://github.com/timstaley/drive-casa
- name: drive-ami
  role: Companion package applying the same terminal-emulation wrapping approach to
    the AMI-reduce calibration pipeline
  repo: https://github.com/timstaley/drive-ami
- name: CASA
  role: The wrapped legacy CLI process whose image-synthesis subroutines (importuvfits,
    Clean) are driven from Python
- name: Python
  role: Implementation language for the interface class, script-generation routines,
    and log parsers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:9d78db0f2c0ecda0c1c2edb63672889bc7efd7e22acabafd8ac149c92004cf21
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pexpect/pexpect
  tools:
  - CASA
  - Python
  - drive-ami
  - drive-casa
  - pexpect
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# python-interface-wrapping-of-legacy-cli

## Summary

Wraps a legacy, externally-maintained command-line tool (e.g. the NRAO CASA casapy process) in a Python interface by spawning it in a sub-process and emulating interactive terminal input with pexpect, so mature radio-astronomy routines can be driven from a fully automated Python reduction pipeline.

## When to use

Use when you need to call a legacy or externally-maintained command-line pipeline (e.g. CASA's casapy, AMI-reduce) from Python scripts without modifying the original code, and no native Python API exists.

## When NOT to use

- When the external tool offers a native, scriptable Python API that can be imported directly — installing packages into CASA (e.g. via the astropy-into-CASA route) is preferred over deprecated executable-wrapping approaches.
- When the wrapped tool's output format is unstable across versions, since fixed parser rules break if the output layout changes to another human-friendly but incompatible format.
- When the pipeline must deconvolve rather than image — the convenience routine's zero-iteration Clean yields only the dirty image by design.

## Inputs

- Raw UVFITS file to be converted into a CASA measurement set via importuvfits
- A command-line executable to be wrapped (e.g. casapy from NRAO CASA, or AMI-reduce)
- CASA task parameters for programmatically assembled script calls (e.g. Clean with niter=0)

## Outputs

- Python interface class (CasapyProcess-style) controlling the spawned external process
- CASA measurement set produced from UVFITS input via importuvfits
- Dirty image produced by a zero-iteration Clean invocation
- Parsed log output with commands and process output on separate channels, plus error reports

## How to apply

Implement a CasapyProcess-style interface class that uses pexpect to spawn the external executable in a sub-process and interact with it by emulating interactive terminal input, as the pexpect-based approach allows rapid development of a Python interface without altering the original code. Route command input and output logging to separate channels so commands sent to the tool are distinguishable from the process's logged output, and parse the tool's log output with fixed parser rules to detect and report errors — recognising these parsers depend on the tool's particular output format. Add convenience routines that programmatically assemble the tool's script (for drive-casa: an importuvfits call to convert a UVFITS input into a CASA measurement set, followed by a Clean invocation with zero iterations so the routine yields the dirty image rather than a deconvolved map). Exercise the interface end-to-end by sending the generated script through the spawned process and confirming the log-parsing/reporting path behaves as designed.

## Related tools

- **pexpect** (Pure-Python Expect-like module used to emulate terminal interaction with the spawned casapy sub-process) — https://github.com/pexpect/pexpect
- **drive-casa** (The resulting Python package providing an interface layer to CASA subroutines via terminal emulation) — https://github.com/timstaley/drive-casa
- **drive-ami** (Companion package applying the same terminal-emulation wrapping approach to the AMI-reduce calibration pipeline) — https://github.com/timstaley/drive-ami
- **CASA** (The wrapped legacy CLI process whose image-synthesis subroutines (importuvfits, Clean) are driven from Python)
- **Python** (Implementation language for the interface class, script-generation routines, and log parsers)

## Examples

```
import drive_casa; casapy = drive_casa.CasapyProcess(); casapy.run_script(['importuvfits(fitsfile=\'raw.uvfits\', vis=\'raw.ms\')', 'clean(imagename=\'dirty.image\', niter=0)'])
```

## Evaluation signals

- End-to-end run: the generated script (importuvfits + zero-iteration Clean) executes successfully through the spawned casapy process and produces a measurement set and dirty image.
- Command input and the process's logged output appear on separate log channels, so every command sent to the tool is distinguishable from its output.
- The log parser correctly detects and reports injected errors in the casapy output rather than silently passing them through.
- The interface works without any alteration to the wrapped tool's original source code.
- Parser rules are validated against the exact output format produced by the wrapped tool version in use.

## Limitations

- Output-parsing in terminal emulation is brittle to changes in output format: if the tool changes to another human-friendly layout, the fixed parser rules break.
- The pexpect approach only works for tools that accept interactive command-line input; it does not expose native Python-level return objects from the wrapped routines.
- pexpect requires only a working Python installation (v2.6 or greater), so Python-version compatibility must be checked for modern environments.

## Evidence

- [discussion] brittle parsing limitation: [evidence span withheld — anchor status: not_found]
