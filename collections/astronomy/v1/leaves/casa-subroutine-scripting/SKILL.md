---
name: casa-subroutine-scripting
description: Use when a Python pipeline must execute CASA subroutines (e.g. Clean
  for image synthesis) on pre-calibrated multi-epoch radio-synthesis measurement sets
  without manual interaction with the CASA environment.
when_to_use_negative:
- Fields containing extended emission or complex blended sources — the article notes
  such fields still need to be manually reduced
- Interactive/exploratory CASA analysis where a human-driven CASA session is simpler
  than building a scripted interface
- Situations where output-parsing depends on a fixed output format that may change,
  breaking brittle terminal-emulation parsers
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3299
tools:
- name: drive-casa
  role: Python interface layer used to script and automate interaction with CASA subroutines
    from a generic Python process
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the underlying image-synthesis subroutines (e.g. Clean) invoked by
    the scripted pipeline
  repo: http://casa.nrao.edu
- name: chimenea
  role: Automated imaging algorithm that builds upon CASA subroutines via drive-casa
    with iterative RMS re-estimation and re-Cleaning
  repo: https://github.com/timstaley/chimenea
- name: pexpect
  role: Python library used to emulate terminal interaction with an external command-line
    process without altering the original code
  repo: https://github.com/pexpect/pexpect
- name: drive-ami
  role: Companion library for scripting the AMI-Reduce calibration pipeline from Python
  repo: https://github.com/timstaley/drive-ami
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:c2686bfce5c386975767b412cfafdf980eccedd9b6310c95c7e39462d5b839aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/drive-casa
  tools:
  - CASA
  - chimenea
  - drive-ami
  - drive-casa
  - pexpect
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# casa-subroutine-scripting

## Summary

Script NRAO CASA image-synthesis subroutines from a generic Python process using the drive-casa interface layer, enabling automated imaging pipelines (e.g. chimenea) to invoke CASA routines programmatically rather than through manual CASA sessions.

## When to use

Use when a Python pipeline must execute CASA subroutines (e.g. Clean for image synthesis) on pre-calibrated multi-epoch radio-synthesis measurement sets without manual interaction with the CASA environment.

## When NOT to use

- Fields containing extended emission or complex blended sources — the article notes such fields still need to be manually reduced
- Interactive/exploratory CASA analysis where a human-driven CASA session is simpler than building a scripted interface
- Situations where output-parsing depends on a fixed output format that may change, breaking brittle terminal-emulation parsers

## Inputs

- Pre-calibrated multi-epoch radio-synthesis data (CASA measurement sets)
- Python driver script invoking CASA subroutines via drive-casa
- Raw AMI-LA datafiles when embedded in the end-to-end AMIsurvey pipeline

## Outputs

- Calibrated and cleaned multi-epoch images ready for source extraction and transient identification
- Iteratively re-Cleaned images with re-estimated background RMS
- Reduction logs from scripted CASA subprocess execution

## How to apply

Use drive-casa to spawn and drive CASA from a Python script, so that mature CASA routines such as Clean can be invoked as subroutines within an automated data-reduction flow. The chimenea pipeline builds on this by calling CASA imaging routines via drive-casa inside an iterative loop in which the background RMS is re-estimated and the field is re-Cleaned multiple times, avoiding 'Clean-bias' through automated source-detection. Where a legacy pipeline only accepts command-line input (e.g. AMI-REDUCE), pair drive-casa-style scripting with pexpect terminal emulation to interact with the external process without altering its original code. Measure the computational cost of repeated re-Cleaning, since for large datasets re-Cleaning may become a significant contribution to overall processing time.

## Related tools

- **drive-casa** (Python interface layer used to script and automate interaction with CASA subroutines from a generic Python process) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the underlying image-synthesis subroutines (e.g. Clean) invoked by the scripted pipeline) — http://casa.nrao.edu
- **chimenea** (Automated imaging algorithm that builds upon CASA subroutines via drive-casa with iterative RMS re-estimation and re-Cleaning) — https://github.com/timstaley/chimenea
- **pexpect** (Python library used to emulate terminal interaction with an external command-line process without altering the original code) — https://github.com/pexpect/pexpect
- **drive-ami** (Companion library for scripting the AMI-Reduce calibration pipeline from Python) — https://github.com/timstaley/drive-ami

## Examples

```
from drive_casa import casarun; images = casarun('clean', vis='precal.ms', imagename='epoch1', niter=1000)
```

## Evaluation signals

- The scripted subprocess drives CASA to completion and returns cleaned images for each epoch without manual intervention
- Iterative loop terminates after background RMS re-estimation converges, and resulting images avoid 'Clean-bias' per the algorithm's design
- Reduction completes within plausible runtime for the dataset size (ALARRM datasets of order a gigabyte or less reduce in minutes)
- Processing time of repeated re-Cleaning steps is measured and does not dominate the pipeline for large datasets
- Terminal-emulation output parsing succeeds against the external tool's actual output format

## Limitations

- For a large dataset the re-Cleaning may become a significant contribution to the overall processing time
- ALARRM reduction benchmarks cover only small datasets (on the order of a gigabyte or less, minutes of reduction time)
- Output-parsing in the terminal-emulation approach is brittle: a changed (but still human-friendly) output layout can break fixed parser rules
- Fields with extended emission or complex blended sources still need to be manually reduced
- Building AMI-REDUCE from source on modern Ubuntu Linux proved problematic; only the pre-built binary was usable

## Evidence

- [discussion] iterative re-Cleaning: [evidence span withheld — anchor status: not_found]
