---
name: raw-data-normalization-caveat-awareness
description: Use when a workflow reads galkin's raw data products — output/rawdata.dat
  or galkin/data/data_*.dat — or original published tracer measurements, and must
  decide whether they can be compared or combined directly with normalized outputs.
when_to_use_negative:
- When the analysis can rely on normalized outputs such as output/vcdata.dat and output/posdata.dat
  produced by galkin_data.py under the selected input parameters
- When only counting or inventorying records (e.g. tracer-category totals) without
  interpreting physical values against a specific parameter set
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: galkin_data.py
  role: Produces both normalized outputs (output/vcdata.dat, output/posdata.dat) and
    the raw data file output/rawdata.dat whose use triggers this caveat
  repo: github.com/galkintool/galkin
- name: galkin
  role: Python package whose processdata routines normalize original published data
    to user-selected input parameters, the alternative to using raw files
  repo: github.com/galkintool/galkin
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:3261ca41ab75dd16e40560d95560d24e59f469c880bcfc085c554389a9aab589
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - galkin
  - galkin_data.py
  - numpy
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# raw-data-normalization-caveat-awareness

## Summary

A guard skill ensuring that raw, un-normalized kinematic data (e.g. galkin's output/rawdata.dat or galkin/data/data_*.dat) are only used with full awareness of each original reference's assumptions and nomenclature, since these files are not normalized to the user-selected Galactic parameters.

## When to use

Use when a workflow reads galkin's raw data products — output/rawdata.dat or galkin/data/data_*.dat — or original published tracer measurements, and must decide whether they can be compared or combined directly with normalized outputs.

## When NOT to use

- When the analysis can rely on normalized outputs such as output/vcdata.dat and output/posdata.dat produced by galkin_data.py under the selected input parameters
- When only counting or inventorying records (e.g. tracer-category totals) without interpreting physical values against a specific parameter set

## Inputs

- output/rawdata.dat (raw data in original references, produced by galkin_data.py)
- galkin/data/data_*.dat (original kinematic data from source references in Table 1)
- input parameter specifications (e.g. inputpars.txt) defining the selected Galactic parameters

## Outputs

- Decision/annotation flagging whether raw data are safe to use directly or must be replaced by normalized galkin output (vcdata.dat, posdata.dat)
- Downstream products (e.g. tracer_measurement_counts.csv) built only from appropriately normalized records

## How to apply

Before tabulating or analyzing raw measurement records (e.g. counting gas/stars/masers tracers in output/rawdata.dat), check that the analysis does not require normalization to the selected input parameters such as R0. If raw data must be used, the agent must first understand the assumptions and nomenclature of each original reference compiled in Table 1, because raw records preserve values as published under each reference's own Galactocentric parameters. For any comparison or derivation requiring consistent parameters, regenerate processed data with galkin_data.py instead, and validate against output/vcdata_baseline.dat and output/posdata_baseline.dat under the baseline Galactic parameter setup.

## Related tools

- **galkin_data.py** (Produces both normalized outputs (output/vcdata.dat, output/posdata.dat) and the raw data file output/rawdata.dat whose use triggers this caveat) — github.com/galkintool/galkin
- **galkin** (Python package whose processdata routines normalize original published data to user-selected input parameters, the alternative to using raw files) — github.com/galkintool/galkin

## Examples

```
python galkin_data.py inputpars.txt  # use output/vcdata.dat & output/posdata.dat; treat output/rawdata.dat as un-normalized
```

## Evaluation signals

- Raw-derived numbers (e.g. tracer counts summing to 2780 across gas, stars, masers) are only accepted as inventories; any physical velocity/radius value taken from rawdata.dat must be accompanied by documented per-reference assumptions
- Any analysis requiring consistent Galactic parameters uses normalized outputs and is checked against output/vcdata_baseline.dat and output/posdata_baseline.dat for the baseline setup
- Workflow log records explicit acknowledgment that rawdata.dat is 'given as in the original reference and thus not normalised to the selected input parameters'

## Limitations

- Raw data files under galkin/data/ and output/rawdata.dat come from references using different Galactocentric parameters and definitions, often overlapping in observed sources, so direct mixing can be inconsistent
- Users are warned not to use raw files directly 'unless you are fully aware of all assumptions and nomenclature used in each original reference'

## Evidence
