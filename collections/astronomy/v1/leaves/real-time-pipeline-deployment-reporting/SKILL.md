---
name: real-time-pipeline-deployment-reporting
description: Use when a paper states a production deployment of a pipeline (e.g. AMBER
  in the ARTS survey) in prose — nodes, GPUs, beams, DMs, integration steps — and
  the goal is to extract and tabulate those exact figures.
when_to_use_negative:
- The task is about tuning AMBER kernels or configuration files for a new scenario
  (use the AMBER_setup 'tune' workflow instead of re-reporting ARTS figures)
- The deployment numbers come from a different survey or hardware setup than the ARTS
  cluster — do not copy the ARTS table without source confirmation
tools:
- name: AMBER
  role: The real-time single-pulse FRB pipeline whose ARTS production deployment (nodes,
    GPUs, beams, DMs, integration steps) is documented by this skill
  repo: https://github.com/TRASAL/AMBER
- name: AMBER_setup
  role: Installer/tuning scripts that generate AMBER configuration files; relevant
    when deployment parameters must be reproduced rather than only reported
  repo: https://github.com/TRASAL/AMBER_setup
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.softx.2020.100549
    title: 'AMBER: A real-time pipeline for the detection of single pulse astronomical
      transients'
  source_content_hash: sha256:3c3522389fe26acb18f68fecbcebfb0e4ea626d0a76cd545d92c1b64518fc31f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TRASAL/AMBER
  tools:
  - AMBER
  - AMBER_setup
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# real-time-pipeline-deployment-reporting

## Summary

Compiles the hardware/algorithm resource-allocation figures used to run a real-time transient-detection pipeline in production (cluster nodes, GPUs, beams, DM search space, integration steps) into a verifiable configuration table. It exists because deployment numbers are scattered in prose and need cross-checking before reuse.

## When to use

Use when a paper states a production deployment of a pipeline (e.g. AMBER in the ARTS survey) in prose — nodes, GPUs, beams, DMs, integration steps — and the goal is to extract and tabulate those exact figures.

## When NOT to use

- The task is about tuning AMBER kernels or configuration files for a new scenario (use the AMBER_setup 'tune' workflow instead of re-reporting ARTS figures)
- The deployment numbers come from a different survey or hardware setup than the ARTS cluster — do not copy the ARTS table without source confirmation

## Inputs

- Published deployment statement from the AMBER SoftwareX paper (abstract section) describing the ARTS survey production setup
- Derived quantities implied by the reported figures (e.g. 5120 DMs / 3 GPUs ≈ one third of the DM space per GPU)

## Outputs

- deployment_config.csv — CSV table with header row (parameter, value, unit) covering 6 allocation dimensions: input beams (480), cluster nodes (40), GPUs per node (3), beams per node (12), DM search space (5120), integration steps (6)
- Derived annotation: per-GPU share of the DM search space (~1/3 of 5120 DMs)

## How to apply

Read the paper's deployment statement (for AMBER, the abstract reports ARTS survey numbers: 480 input beams, 40 cluster nodes, 3 onboard GPUs per node, 12 beams per node, a 5120-DM search space, and 6 integration steps). Compile each reported value into a table with one row per allocation dimension (beams, cluster nodes, GPUs per node, beams per node, DM search space, integration steps), recording the parameter name, value, and unit. Add derived notes only where arithmetic follows directly from reported figures, e.g. that each of the 3 GPUs handles roughly one third of the 5120-DM search space. Write the table to a machine-readable file (deployment_config.csv, CSV with a header row), then validate every cell against the source text and confirm the file parses with the expected columns.

## Related tools

- **AMBER** (The real-time single-pulse FRB pipeline whose ARTS production deployment (nodes, GPUs, beams, DMs, integration steps) is documented by this skill) — https://github.com/TRASAL/AMBER
- **AMBER_setup** (Installer/tuning scripts that generate AMBER configuration files; relevant when deployment parameters must be reproduced rather than only reported) — https://github.com/TRASAL/AMBER_setup

## Examples

```
printf 'parameter,value,unit\ninput_beams,480,beams\ncluster_nodes,40,nodes\ngpus_per_node,3,GPUs\nbeams_per_node,12,beams\ndm_search_space,5120,DMs\nintegration_steps,6,steps\n' > deployment_config.csv
```

## Evaluation signals

- Every table cell matches the paper's reported figures exactly: 480 beams, 40 nodes, 3 GPUs/node, 12 beams/node, 5120 DMs, 6 integration steps
- Consistency check: 40 nodes × 12 beams/node = 480 input beams
- CSV parses with the expected header row and one row per allocation dimension (6 rows plus header)
- Derived note (each GPU handles ~1/3 of the 5120-DM search space) follows arithmetically from 3 GPUs and 5120 DMs
- Each GPU is reported to handle all 6 integration steps, with the DM space (not integration) split across GPUs

## Limitations

- Figures are specific to the ARTS survey deployment; other AMBER deployments will differ in nodes, GPUs, beams, DMs, and integration steps
- Per-GPU DM allocation is reported as 'roughly one third' — exact DM partitioning per GPU is not stated in the source
- The paper's discussion section content was unavailable in the provided context, so only abstract-level deployment figures could be verified

## Evidence

- [abstract] ARTS deployment figures: "These observations are being processed in real-time on the 40 nodes of the ARTS cluster, using 3 of the onboard GPUs; each node processes 12 of the 480 input beams"
- [abstract] real-time purpose of AMBER: "AMBER, or the “Apertif Monitor for Bursts Encountered in Real-time”, is a software pipeline designed and developed to make it possible to detect single-pulse transients in real-time."
- [readme] AMBER definition from README: "AMBER is a many-core accelerated and fully auto-tuned pipeline for detecting Fast Radio Bursts and other single pulse radio transients."
