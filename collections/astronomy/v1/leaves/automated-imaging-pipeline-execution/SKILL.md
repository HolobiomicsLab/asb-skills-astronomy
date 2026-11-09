---
name: automated-imaging-pipeline-execution
description: Use when you have pre-calibrated multi-epoch radio-synthesis visibilities
  (e.g. AMI-LA measurement sets) and need automated calibration-to-image reduction
  with source detection, avoiding 'Clean-bias'.
when_to_use_negative:
- Fields containing extended emission or complex blended sources — the paper states
  such fields 'still need to be manually reduced'
- 'Datasets where iterative re-Cleaning dominates runtime: for large datasets the
  re-Cleaning may become a significant contribution to overall processing time'
- When flux-density accuracy is the primary goal and sources are already identified
  — the paper sanctions uv-plane model fitting (UVMULTIFIT-style) as the extension
  for improved flux-density measurements instead of image-based measurements
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: AMIsurvey
  role: End-to-end telescope-specific pipeline that ingests raw AMI-LA datafiles and
    produces calibrated, cleaned multi-epoch images ready for source extraction
  repo: https://github.com/timstaley/amisurvey
- name: chimenea
  role: Implements the telescope-agnostic automated imaging algorithm (iterative RMS-estimation,
    automated source-detection, repeated Clean); key logic in chimenea/pipeline.py
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer for calling CASA subroutines (image-synthesis/Clean
    operations) from the pipeline
  repo: https://github.com/timstaley/drive-casa
- name: drive-ami
  role: Python package for scripting the AMI-Reduce calibration pipeline on raw AMI
    data
  repo: https://github.com/timstaley/drive-ami
- name: CASA
  role: Provides the underlying image-synthesis subroutines invoked by chimenea via
    drive-casa
- name: pexpect
  role: 'Terminal emulation: spawns a legacy command-line tool in a sub-process and
    emulates interactive terminal input, enabling Python control without altering
    the original code'
  repo: https://github.com/pexpect/pexpect
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:bc3ba45a7cf486e20f8c93b7751dcb3d3dc891d9ab3306b28aea463e8955af4e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/amisurvey
  tools:
  - AMIsurvey
  - CASA
  - chimenea
  - drive-ami
  - drive-casa
  - pexpect
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# automated-imaging-pipeline-execution

## Summary

Execution of the chimenea/AMIsurvey automated imaging pipeline: telescope-agnostic, scripted imaging of pre-calibrated multi-epoch radio-synthesis data using iterative RMS-estimation and repeated Clean operations driven from Python via drive-casa/CASA subroutines.

## When to use

Use when you have pre-calibrated multi-epoch radio-synthesis visibilities (e.g. AMI-LA measurement sets) and need automated calibration-to-image reduction with source detection, avoiding 'Clean-bias'.

## When NOT to use

- Fields containing extended emission or complex blended sources — the paper states such fields 'still need to be manually reduced'
- Datasets where iterative re-Cleaning dominates runtime: for large datasets the re-Cleaning may become a significant contribution to overall processing time
- When flux-density accuracy is the primary goal and sources are already identified — the paper sanctions uv-plane model fitting (UVMULTIFIT-style) as the extension for improved flux-density measurements instead of image-based measurements

## Inputs

- Raw AMI-LA datafiles (AMI-Reduce calibration input)
- Pre-calibrated multi-epoch radio-synthesis measurement sets (CASA format visibilities)

## Outputs

- Calibrated and cleaned multi-epoch radio images (CASA images)
- Background RMS estimates per imaging iteration
- Detected candidate source lists for flux-density measurement and transient identification

## How to apply

Ingest raw AMI-LA datafiles into AMIsurvey, which ties together drive-ami (scripting the AMI-Reduce calibration pipeline) and drive-casa (scripting NRAO CASA image-synthesis routines) to produce calibrated, cleaned multi-epoch images. For each field, run the chimenea algorithm implemented in chimenea/pipeline.py: perform a Clean operation, re-estimate the background RMS, and re-Clean iteratively with automated source-detection until convergence, which prevents 'Clean-bias' from over-negative clean components. Drive everything from a generic Python process; for legacy command-line-only tools, use the pexpect terminal-emulation approach (spawn the external tool in a sub-process and emulate interactive terminal input) rather than altering original code. The resulting cleaned images feed source extraction and transient identification as in the ALARRM programme. Validate by confirming the pipeline completes each epoch, background RMS estimates stabilise across re-Clean iterations, and detected sources are consistent across epochs.

## Related tools

- **AMIsurvey** (End-to-end telescope-specific pipeline that ingests raw AMI-LA datafiles and produces calibrated, cleaned multi-epoch images ready for source extraction) — https://github.com/timstaley/amisurvey
- **chimenea** (Implements the telescope-agnostic automated imaging algorithm (iterative RMS-estimation, automated source-detection, repeated Clean); key logic in chimenea/pipeline.py) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer for calling CASA subroutines (image-synthesis/Clean operations) from the pipeline) — https://github.com/timstaley/drive-casa
- **drive-ami** (Python package for scripting the AMI-Reduce calibration pipeline on raw AMI data) — https://github.com/timstaley/drive-ami
- **CASA** (Provides the underlying image-synthesis subroutines invoked by chimenea via drive-casa)
- **pexpect** (Terminal emulation: spawns a legacy command-line tool in a sub-process and emulates interactive terminal input, enabling Python control without altering the original code) — https://github.com/pexpect/pexpect

## Examples

```
from chimenea import pipeline; pipeline.reduce_all_obsid_list(obsid_list, work_dir, casa_cmd='casa')
```

## Evaluation signals

- Each epoch produces a calibrated, cleaned image without pipeline abort; AMIsurvey output images exist for all input epochs
- Iterative background RMS re-estimation converges — successive re-Clean iterations no longer change the RMS or detected source list materially
- No evidence of 'Clean-bias': detected source fluxes are not systematically depressed by over-aggressive clean components in low-RMS regions
- Automated source-detection lists are consistent across epochs for steady sources, enabling transient identification as in ALARRM
- Output-parsing from terminal-emulated tools succeeds across runs (pexpect parsers match the tool's current output format)

## Limitations

- No automated handling for extended emission or complex blended sources; such fields still need manual reduction
- Computational overhead of iterative re-Cleaning is unquantified for large datasets and may become a significant fraction of processing time
- Reduction times are benchmarked only on small ALARRM datasets (file sizes on the order of a gigabyte or less, reduction times on the order of minutes)
- Output-parsing in terminal emulation is brittle: a human-friendly change in output layout can break fixed parser rules
- No version numbers are given for the software packages described

## Evidence

- [abstract] chimenea implements telescope-agnostic automated imaging: "The most scientifically significant of these is chimenea, which implements a telescope-agnostic algorithm for automated imaging of pre-calibrated multi-epoch radio-synthesis data"
- [abstract] iterative RMS-estimation avoids Clean-bias: "The algorithm aims to improve upon standard imaging pipelines by utilizing iterative RMS-estimation and automated source-detection to avoid so called ‘Clean-bias’"
- [discussion] repeated Clean with RMS re-estimation: [evidence span withheld — anchor status: not_found]
- [intro] AMIsurvey end-to-end pipeline role: [evidence span withheld — anchor status: not_found]
- [abstract] pexpect terminal emulation approach: "This approach allows for rapid development of a Python interface to any legacy or externally-maintained pipeline which accepts command-line input, without requiring alterations to the original code."
- [readme] chimenea built on CASA via drive-casa: "The chimenea pipeline is built upon NRAO [CASA](http://casa.nrao.edu) 
subroutines, interacting with the CASA environment via the 
[drive-casa](https://github.com/timstaley/drive-casa)
interface"
