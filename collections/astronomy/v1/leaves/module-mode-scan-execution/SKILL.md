---
name: module-mode-scan-execution
description: 'Use when a GAMBIT installation needs to be exercised end-to-end: an
  example YAML scan (e.g. yaml_files/MSSM7.yaml) must be run in module mode so that
  a module like DarkBit emits observable and likelihood results through the GAMBIT
  printer.'
when_to_use_negative:
- When the goal is to run DarkBit as a stand-alone code rather than inside a GAMBIT
  module-mode scan (DarkBit supports both modes).
- When backends or scanners are not yet configured — `gambit backends` / `gambit scanners`
  verification must pass first; running the scan without them risks fatal errors unrelated
  to the YAML input.
edam_operation: http://edamontology.org/operation_2945
edam_topics: []
tools:
- name: GAMBIT
  role: Driver invoked in module mode with an example YAML file to run the scan and
    print results
  repo: github.com/GambitBSM/gambit
- name: DarkBit
  role: GAMBIT module computing dark matter observables and likelihoods exercised
    by the example scan
  repo: github.com/GambitBSM/gambit_1.5
- name: DarkSUSY
  role: Backend that DarkBit relies on; must be confirmed configured via `gambit backends`
- name: micrOMEGAs
  role: Backend interfaced by DarkBit; verified as installed before the scan
- name: DDCalc
  role: Direct detection likelihood package used by DarkBit during module-mode runs
- name: gamLike
  role: Gamma-ray likelihood calculator invoked through DarkBit in the scan
- name: nulike
  role: Neutrino telescope likelihood package used via DarkBit backends
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:f84b5eb68348f0206717b2221f13299231c14cc16f85947dd2122a4981e9a2d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DDCalc
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - gamLike
  - micrOMEGAs
  - nulike
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# module-mode-scan-execution

## Summary

Launches a GAMBIT module (e.g. DarkBit) as part of a full module-mode scan driven by an example YAML input file, reproducing the packaged illustrative example results including dark matter observables and likelihoods.

## When to use

Use when a GAMBIT installation needs to be exercised end-to-end: an example YAML scan (e.g. yaml_files/MSSM7.yaml) must be run in module mode so that a module like DarkBit emits observable and likelihood results through the GAMBIT printer.

## When NOT to use

- When the goal is to run DarkBit as a stand-alone code rather than inside a GAMBIT module-mode scan (DarkBit supports both modes).
- When backends or scanners are not yet configured — `gambit backends` / `gambit scanners` verification must pass first; running the scan without them risks fatal errors unrelated to the YAML input.

## Inputs

- GAMBIT example YAML input file (e.g. yaml_files/MSSM7.yaml from the yaml_files folder)
- Configured GAMBIT installation with DarkBit module built
- Installed backends relied on by DarkBit: DarkSUSY, micrOMEGAs, DDCalc, gamLike, nulike

## Outputs

- Module-mode scan run log (e.g. darkbit_scan_output.txt) containing the DarkBit module header, dark matter observable results, and likelihood results
- Printed output from the GAMBIT printer (observables and likelihoods for the scanned parameter points)

## How to apply

First verify the installed compute components with `gambit backends` and `gambit scanners` to confirm the backends DarkBit relies on (DarkSUSY, micrOMEGAs, DDCalc, gamLike, nulike) are configured before spending compute on a scan. Then launch the scan in module mode from the repository root with `gambit -f yaml_files/MSSM7.yaml`, optionally substituting another example YAML from the yaml_files folder. Capture the complete module-mode output, including the dark matter observable and likelihood results emitted through the GAMBIT printer, into a run log such as darkbit_scan_output.txt. Rationale: the DarkBit paper demonstrates module-mode use via illustrative examples, so a successful example YAML run reproduces reported example results and validates that the module, its backends, and the printer pipeline are wired correctly.

## Related tools

- **GAMBIT** (Driver invoked in module mode with an example YAML file to run the scan and print results) — github.com/GambitBSM/gambit
- **DarkBit** (GAMBIT module computing dark matter observables and likelihoods exercised by the example scan) — github.com/GambitBSM/gambit_1.5
- **DarkSUSY** (Backend that DarkBit relies on; must be confirmed configured via `gambit backends`)
- **micrOMEGAs** (Backend interfaced by DarkBit; verified as installed before the scan)
- **DDCalc** (Direct detection likelihood package used by DarkBit during module-mode runs)
- **gamLike** (Gamma-ray likelihood calculator invoked through DarkBit in the scan)
- **nulike** (Neutrino telescope likelihood package used via DarkBit backends)

## Examples

```
gambit -f yaml_files/MSSM7.yaml > darkbit_scan_output.txt 2>&1
```

## Evaluation signals

- The `gambit -f yaml_files/MSSM7.yaml` invocation completes without fatal errors (script_runs check).
- The output log file (darkbit_scan_output.txt) exists on disk after the run.
- The log contains the DarkBit module header (contains_substring check on 'DarkBit').
- The log contains dark matter observable and likelihood results emitted through the GAMBIT printer.
- `gambit backends` and `gambit scanners` list the components DarkBit relies on as configured before the scan is launched.

## Limitations

- cmake may fail to find some dependencies on some systems without guidance; the Core paper provides more information, and cluster configuration examples are available via gambit.hepforge.org.
- Scanners and backends must be built separately (make -jn scanners / make -jn backends) before a module-mode scan will work.
- Only example YAML files in the yaml_files folder (e.g. MSSM7.yaml) are guaranteed to reproduce the packaged illustrative example results; other inputs are untested in this context.

## Evidence

- [results] DarkBit is demonstrated via illustrative examples in both stand-alone and GAMBIT module mode: [evidence span withheld — anchor status: not_found]
- [methods] Run GAMBIT scan via yaml input file: "gambit -f yaml_files/MSSM7.yaml"
- [methods] Verify installed backends and scanners: "gambit backends
  gambit scanners"
- [intro] GAMBIT backend system: "a flexible system for interfacing external codes (the "backend" system)"
- [readme] GAMBIT printer and scanning framework: "a fully featured statistical and parameter scanning framework"
