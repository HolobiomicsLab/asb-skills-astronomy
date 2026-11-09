---
name: frb-survey-configuration-extraction
description: Use when a paper states a production FRB survey deployment in prose (nodes,
  GPUs, beams per node, DM count, integration steps) and the goal is to reproduce
  that allocation as a structured table or CSV.
when_to_use_negative:
- The deployment details are not stated quantitatively in the source text (e.g. only
  qualitative statements like 'uses many-core accelerators') — do not fabricate numbers.
- The goal is to configure or tune AMBER for a new run rather than to document an
  existing published deployment — use the AMBER_setup tuning workflow (amber.sh tune)
  instead.
- The question concerns pipeline internals (dedispersion, RFI mitigation, candidate
  clustering) rather than hardware/resource allocation.
tools:
- name: AMBER
  role: The deployed real-time FRB pipeline whose ARTS survey configuration (40 nodes,
    3 GPUs/node, 12 beams/node, 5120 DMs, 6 integration steps) is being extracted
    and tabulated
  repo: https://github.com/TRASAL/AMBER
- name: AMBER_setup
  role: Installer/tuning scripts for AMBER; relevant as the contrast point for generating
    (rather than extracting) configuration files for a new deployment
  repo: https://github.com/TRASAL/AMBER_setup
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.softx.2020.100549
    title: 'AMBER: A real-time pipeline for the detection of single pulse astronomical
      transients'
  source_content_hash: sha256:6a0791791203c14f1855ae080fbd0a27a1a535f7fec3a23413e14e3d6d40caaf
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# frb-survey-configuration-extraction

## Summary

Extracts and tabulates the concrete hardware-allocation and search-space configuration used to deploy a real-time FRB pipeline in a production survey (e.g. AMBER within ARTS): cluster nodes, GPUs, beams, DMs, and integration steps. It turns prose deployment statements from a paper's abstract into a validated, machine-readable parameter table.

## When to use

Use when a paper states a production FRB survey deployment in prose (nodes, GPUs, beams per node, DM count, integration steps) and the goal is to reproduce that allocation as a structured table or CSV.

## When NOT to use

- The deployment details are not stated quantitatively in the source text (e.g. only qualitative statements like 'uses many-core accelerators') — do not fabricate numbers.
- The goal is to configure or tune AMBER for a new run rather than to document an existing published deployment — use the AMBER_setup tuning workflow (amber.sh tune) instead.
- The question concerns pipeline internals (dedispersion, RFI mitigation, candidate clustering) rather than hardware/resource allocation.

## Inputs

- Published article text (abstract) describing a real-time FRB survey deployment, e.g. the AMBER SoftwareX paper abstract
- Source research question or task card specifying which allocation dimensions to extract (nodes, GPUs, beams, DMs, integration steps)

## Outputs

- deployment_config.csv: CSV with header row (parameter, value, unit) and one row per allocation dimension
- Derived annotation note per GPU share of the DM search space (e.g. ~1/3 of 5120 DMs per GPU)

## How to apply

Locate the deployment sentence(s) in the source article — for AMBER this is in the SoftwareX abstract — and enumerate every quantitative allocation dimension mentioned. For ARTS the values are 40 cluster nodes, 3 onboard GPUs per node, 480 input beams total, 12 beams per node, a search space of 5120 dispersion measures (DMs), and 6 integration steps. Compile one row per allocation dimension (parameter name, reported value, unit) into a header-bearing CSV such as deployment_config.csv. Add derived notes only where the paper states them, e.g. each GPU handles roughly one third of the 5120-DM search space and all integration steps. Cross-check every tabulated value verbatim against the paper's figures before finalizing, and confirm the output file parses with the expected columns.

## Related tools

- **AMBER** (The deployed real-time FRB pipeline whose ARTS survey configuration (40 nodes, 3 GPUs/node, 12 beams/node, 5120 DMs, 6 integration steps) is being extracted and tabulated) — https://github.com/TRASAL/AMBER
- **AMBER_setup** (Installer/tuning scripts for AMBER; relevant as the contrast point for generating (rather than extracting) configuration files for a new deployment) — https://github.com/TRASAL/AMBER_setup

## Examples

```
python -c "import csv; rows=[('cluster_nodes',40,'nodes'),('gpus_per_node',3,'GPUs'),('input_beams',480,'beams'),('beams_per_node',12,'beams/node'),('dm_search_space',5120,'DMs'),('integration_steps',6,'steps')]; w=csv.writer(open('deployment_config.csv','w')); w.writerow(['parameter','value','unit']); w.writerows(rows)"
```

## Evaluation signals

- Every tabulated value matches the paper verbatim: 480 beams, 40 nodes, 3 GPUs per node, 12 beams per node, 5120 DMs, 6 integration steps.
- deployment_config.csv parses with the expected header row and one row per allocation dimension.
- The derived per-GPU DM share is consistent with the paper: 3 GPUs partitioning 5120 DMs means roughly one third (~1707) of the DM space per GPU, covering all integration steps.
- No parameter is reported in the table that lacks an explicit statement in the source article.
- Units are recorded for every parameter row (e.g. beams, nodes, GPUs, DMs, integration steps).

## Limitations

- Values reflect the specific ARTS survey deployment reported in the AMBER abstract; other surveys or hardware setups may use different allocations.
- Per-GPU DM share is approximate ('roughly one third'); the paper does not specify exact per-GPU DM partitioning or load-balancing details.
- Only dimensions explicitly stated in prose can be extracted; e.g. GPU model, per-node memory, or node interconnect are not reported and must not be inferred.

## Evidence

- [abstract] ARTS real-time deployment figures: "These observations are being processed in real-time on the 40 nodes of the ARTS cluster, using 3 of the onboard GPUs; each node processes 12 of the 480 input beams"
- [abstract] Production usage context: "While AMBER is being used in production within the ARTS survey, and it is already contributing to the discovery of new FRBs"
- [readme] Pipeline identity from README: "AMBER is a many-core accelerated and fully auto-tuned pipeline for detecting Fast Radio Bursts and other single pulse radio transients."
- [abstract] Output artifacts of a run: "AMBER produces two output files after each successful execution: (1) a file containing candidate detections, and (2) a file containing performance statistics on the run."
