---
name: dispersion-delay-alignment
description: Use when input is a dispersed frequency-channel filterbank (e.g. SIGPROC
  format) in which a pulse arrives later at lower frequencies due to interstellar
  free electrons, and a DM-search dedispersed time series is needed.
when_to_use_negative:
- Input data is already dedispersed or comes from a coherent-dedispersion backend
  where frequency-dependent delays are already removed
- The goal is RFI excision rather than dispersion correction (use the separate RFIm
  stage)
- Input is a folded/pulsar-averaged time series rather than frequency-resolved filterbank
  samples
tools:
- name: Dedispersion
  role: OpenCL incoherent dedispersion kernels and C++ classes that apply the DM-dependent
    per-channel time shifts
  repo: https://github.com/TRASAL/Dedispersion
- name: AMBER
  role: Real-time pipeline hosting the Dedispersion stage as its first computing stage
  repo: https://github.com/TRASAL/AMBER_setup
- name: OpenCL
  role: Required dependency for executing the dedispersion kernels on many-core accelerators
    (GPUs)
  repo: https://github.com/isazi/OpenCL
- name: AstroData
  role: C++ classes to operate on the radio astronomical filterbank data fed to dedispersion
  repo: https://github.com/TRASAL/AstroData
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2020.100549
    title: 'AMBER: A real-time pipeline for the detection of single pulse astronomical
      transients'
  source_content_hash: sha256:d314e1b67cfefd5dc1fbfe863225282bdfe65ebd6f266ba994a0b96c06ba74d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TRASAL/Dedispersion
  tools:
  - AMBER
  - AstroData
  - Dedispersion
  - OpenCL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# dispersion-delay-alignment

## Summary

Aligns frequency channels of dispersed radio filterbank data by applying cold-plasma dispersion-delay corrections and summing channels, producing dedispersed time series for single-pulse transient (e.g. FRB) detection — this is the first processing stage of the AMBER pipeline.

## When to use

Use when input is a dispersed frequency-channel filterbank (e.g. SIGPROC format) in which a pulse arrives later at lower frequencies due to interstellar free electrons, and a DM-search dedispersed time series is needed.

## When NOT to use

- Input data is already dedispersed or comes from a coherent-dedispersion backend where frequency-dependent delays are already removed
- The goal is RFI excision rather than dispersion correction (use the separate RFIm stage)
- Input is a folded/pulsar-averaged time series rather than frequency-resolved filterbank samples

## Inputs

- SIGPROC-format filterbank with a dispersed pulse across frequency channels
- Grid of trial dispersion measures (DMs) spanning the expected DM

## Outputs

- One dedispersed time series per trial DM (e.g. dedispersed_timeseries.dat)
- Candidate detections downstream (beam, DM, time, pulse width, SNR)

## How to apply

Configure the Dedispersion stage (github.com/TRASAL/Dedispersion, an OpenCL many-core incoherent dedispersion algorithm) with a grid of trial dispersion measures spanning the expected/injected DM. Execute the kernel on the OpenCL device, applying the DM-dependent per-channel time shift for each trial DM. Sum the shifted frequency channels to produce one dedispersed time series per trial DM (written to e.g. dedispersed_timeseries.dat). Rationale: interstellar free electrons delay lower-frequency photons of the same emission, so only the correct trial DM collapses the pulse into a single aligned time bin. In AMBER this is the first of four computing stages, feeding downstream Integration, RFIm, and SNR stages, and kernels are auto-tuned to the execution hardware.

## Related tools

- **Dedispersion** (OpenCL incoherent dedispersion kernels and C++ classes that apply the DM-dependent per-channel time shifts) — https://github.com/TRASAL/Dedispersion
- **AMBER** (Real-time pipeline hosting the Dedispersion stage as its first computing stage) — https://github.com/TRASAL/AMBER_setup
- **OpenCL** (Required dependency for executing the dedispersion kernels on many-core accelerators (GPUs)) — https://github.com/isazi/OpenCL
- **AstroData** (C++ classes to operate on the radio astronomical filterbank data fed to dedispersion) — https://github.com/TRASAL/AstroData

## Examples

```
amber.sh tune examples/scenario.sh ${INSTALL_ROOT}/confs
```

## Evaluation signals

- The dedispersed time series at the injected/correct DM shows the pulse collapsed into a single aligned time bin
- Maximum peak SNR in the dedispersed time series occurs at the injected DM, not at neighbouring trial DMs
- A pulse injected at a known DM is visibly smeared across the time axis at wrong trial DMs (distinguishing real dispersion from noise)
- One dedispersed time series is produced per trial DM in the grid, written to the expected output file (dedispersed_timeseries.dat)

## Limitations

- Incoherent dedispersion only corrects dispersion to the resolution of the trial-DM grid; the injected DM must fall within the trial-DM range to recover full SNR
- OpenCL is a necessary dependency: without a working OpenCL environment (device, drivers) the kernels cannot run
- Kernels must be auto-tuned to the execution hardware; configuration files must be tuned for the specific scenario before testing

## Evidence

- [abstract] Dispersion delay physics: "Because the different frequencies emitted by a radio source are dispersed over time by the interaction of the emission photons with the free electrons present in the interstellar medium"
- [abstract] Dedispersion as pipeline stage: "Currently, the AMBER’s pipeline is composed of four computing stages"
- [readme] Dedispersion implementation: [evidence span withheld — anchor status: external_doc]
- [methods] Dedispersion dependency: "* [Dedispersion](https://github.com/TRASAL/Dedispersion)"
- [readme] OpenCL requirement: [evidence span withheld — anchor status: external_doc]
