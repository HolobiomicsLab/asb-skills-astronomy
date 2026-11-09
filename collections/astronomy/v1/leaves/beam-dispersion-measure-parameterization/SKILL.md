---
name: beam-dispersion-measure-parameterization
description: Use when you have SNR detections over an integrated dedispersed time-series
  for many beams and dispersion measures, and need to convert threshold-exceeding
  samples into per-candidate records with beam, DM, time, pulse width, and SNR.
when_to_use_negative:
- Input is raw dispersed voltage or filterbank data that has not yet been dedispersed
  and integrated — the dedispersion and integration stages must run first
- The goal is pipeline performance characterization rather than candidate identification
  — AMBER's second output file (performance statistics) covers that, not the candidate
  records
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3399
tools:
- name: AMBER
  role: Pipeline whose final stage produces the candidate detections file; defines
    the candidates.dat output format and two-output convention
  repo: https://github.com/TRASAL/AMBER
- name: SNR
  role: OpenCL SNR kernels executed over the integrated time series for every beam
    and dispersion measure to flag threshold-exceeding candidates
  repo: https://github.com/TRASAL/SNR
- name: AstroData
  role: C++ classes used to load and operate on the integrated dedispersed radio astronomical
    time-series
  repo: https://github.com/TRASAL/AstroData
- name: Integration
  role: Provides the integration algorithm whose output time-series are the input
    to the SNR detection stage
  repo: https://github.com/TRASAL/Integration
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2020.100549
    title: 'AMBER: A real-time pipeline for the detection of single pulse astronomical
      transients'
  source_content_hash: sha256:a3bc3dcabb0d5dd40d6c929d5530d90eb862a0c39929a2690357e3aa5eecb759
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TRASAL/AMBER
  tools:
  - AMBER
  - AstroData
  - Integration
  - SNR
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# beam-dispersion-measure-parameterization

## Summary

Labels every single-pulse candidate detection from an SNR search over dedispersed time-series with the coordinates that identify it — beam, DM, time of detection, and pulse width — together with the SNR of the detected signal, producing AMBER-style candidate records.

## When to use

Use when you have SNR detections over an integrated dedispersed time-series for many beams and dispersion measures, and need to convert threshold-exceeding samples into per-candidate records with beam, DM, time, pulse width, and SNR.

## When NOT to use

- Input is raw dispersed voltage or filterbank data that has not yet been dedispersed and integrated — the dedispersion and integration stages must run first
- The goal is pipeline performance characterization rather than candidate identification — AMBER's second output file (performance statistics) covers that, not the candidate records

## Inputs

- Integrated dedispersed time-series per beam and dispersion measure (TRASAL/AstroData and TRASAL/Integration data structures)
- SNR detection threshold
- Tuned AMBER configuration files generated for the observation scenario

## Outputs

- candidates.dat in the AMBER output format, with one row per candidate containing beam, DM, time, pulse width, and SNR
- Optionally compacted candidate list: only the highest-SNR candidate per DM/pulse-width cluster
- Separate performance statistics file for the run

## How to apply

Load the integrated dedispersed time-series produced by the integration stage using the TRASAL/AstroData and TRASAL/Integration data structures. Run the SNR stage from the TRASAL/SNR module, executing its OpenCL kernel over the integrated time series for every beam and dispersion measure in the observation, flagging as candidates all time samples whose SNR exceeds the detection threshold. For each flagged sample, record the five identifying fields: beam, DM, time of detection, pulse width, and SNR. Optionally, if the results compacting mode is enabled, cluster candidates in the two dimensions DM and pulse width and output only the candidate with the highest SNR of each cluster. Write the candidate detections to candidates.dat in the AMBER output format, alongside the separate performance statistics file.

## Related tools

- **AMBER** (Pipeline whose final stage produces the candidate detections file; defines the candidates.dat output format and two-output convention) — https://github.com/TRASAL/AMBER
- **SNR** (OpenCL SNR kernels executed over the integrated time series for every beam and dispersion measure to flag threshold-exceeding candidates) — https://github.com/TRASAL/SNR
- **AstroData** (C++ classes used to load and operate on the integrated dedispersed radio astronomical time-series) — https://github.com/TRASAL/AstroData
- **Integration** (Provides the integration algorithm whose output time-series are the input to the SNR detection stage) — https://github.com/TRASAL/Integration

## Examples

```
amber.sh tune examples/scenario.sh ${INSTALL_ROOT}/confs  # generate tuned configs so the SNR stage can emit candidates.dat for every beam and DM
```

## Evaluation signals

- candidates.dat exists after every successful execution, consistent with AMBER's documented two-output convention (candidate detections plus performance statistics)
- Every row of candidates.dat contains the five required fields: beam, DM, time, pulse width, and SNR
- Every recorded candidate's SNR exceeds the configured detection threshold
- Candidates are enumerated across all beams and dispersion measures of the observation (e.g. 480 beams and 5120 DMs in the ARTS scenario)
- If results compacting is enabled, no two output candidates share the same DM/pulse-width cluster

## Limitations

- Without results compacting enabled, clustered candidates in DM and pulse width are all written out, inflating candidate counts for single astrophysical events
- Candidate coordinate fidelity depends on the upstream dedispersion and integration stages and on auto-tuned OpenCL kernel parameters (work-group size, local memory, serial processing per thread)
- Requires a working OpenCL environment; kernels require many low-level parameters that must be tuned to the execution hardware

## Evidence

- [abstract] Candidate records carry the five identifying coordinates plus SNR: "AMBER produces two output files after each successful execution: (1) a file containing candidate detections, and (2) a file containing performance statistics on the run."
- [abstract] Results compacting clusters candidates and keeps only the highest-SNR one: "If the results compacting mode is enabled, AMBER clusters the results in two dimensions (i.e. DM and pulse width), and outputs only the candidate with the highest SNR of each cluster."
- [methods] SNR module provides the many-core kernels used for detection: [evidence span withheld — anchor status: external_doc]
- [abstract] OpenCL kernels depend on low-level tuning parameters: "The OpenCL kernels require many low-level parameters, such as the OpenCL work-group size, the usage of local memory, or the amount of serial processing per thread"
- [readme] OpenCL is a mandatory dependency of the pipeline: [evidence span withheld — anchor status: external_doc]
