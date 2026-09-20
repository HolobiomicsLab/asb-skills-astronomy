---
name: runtime-benchmark-measurement
description: Use when a new fitting or analysis pipeline's computational cost must
  be compared against a reference method on a fixed sample, e.g. showing that SMART's
  radiative-transfer-only fitting of 42 HERUS ULIRG SEDs takes comparable time to
  popular energy balance methods.
when_to_use_negative:
- The reference method has been re-implemented locally with different settings — re-measure
  both rather than mixing your own timings with literature-reported values.
- The analysis goal is fitting accuracy or parameter recovery rather than computational
  cost — runtime benchmarking alone will not answer that.
edam_topics:
- http://edamontology.org/topic_0007
tools:
- name: SMART
  role: Target code whose per-galaxy and total wall-clock runtime is measured while
    it performs Bayesian MCMC SED fitting with radiative transfer libraries
  repo: https://github.com/ch-var/SMART
- name: Python
  role: Used to assemble the per-object and total runtime comparison table and write
    runtime_comparison.csv
  repo: https://github.com/ch-var/SMART.git
- name: Spitzer
  role: Source of the spectroscopy included in the HERUS ULIRG SEDs being fit during
    the timed runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:e43150d26ddc5d7fddb070cfc495ec3c4bdafa4e5d066995060e9b334a0eb5ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART
  tools:
  - Python
  - SMART
  - Spitzer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0007
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# runtime-benchmark-measurement

## Summary

Method for measuring and comparing the wall-clock execution time of a scientific SED-fitting code (e.g. SMART's Bayesian MCMC fitting of radiative transfer libraries) against an established alternative method (e.g. energy balance fitting) on the same sample, to establish computational comparability.

## When to use

Use when a new fitting or analysis pipeline's computational cost must be compared against a reference method on a fixed sample, e.g. showing that SMART's radiative-transfer-only fitting of 42 HERUS ULIRG SEDs takes comparable time to popular energy balance methods.

## When NOT to use

- The reference method has been re-implemented locally with different settings — re-measure both rather than mixing your own timings with literature-reported values.
- The analysis goal is fitting accuracy or parameter recovery rather than computational cost — runtime benchmarking alone will not answer that.

## Inputs

- Multiwavelength SED photometry plus Spitzer spectroscopy for the 42 HERUS ULIRGs
- SMART pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust)
- Published runtime figures for a reference energy balance SED-fitting method (from literature/paper text)

## Outputs

- Per-object and total wall-clock runtime measurements for the target code (SMART)
- runtime_comparison.csv: comparison table with 42 galaxy entries plus summary rows of SMART vs. energy balance method runtimes

## How to apply

Load the full multiwavelength SED inputs for every object in the sample (here, including Spitzer spectroscopy for the 42 HERUS ULIRGs) into the target code. Run the code's fitting procedure unchanged — for SMART this is Bayesian MCMC over four pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) — and record elapsed wall-clock time per object and for the full sample. Retrieve the reported runtime of the reference method on comparable work from its publication text rather than re-running it, since only an order-of-magnitude comparison is claimed. Assemble a per-object and total comparison table in Python and write it to runtime_comparison.csv. Validate that the table contains 42 galaxy entries plus summary rows and that the target runtime is of the same order as the reference method's, matching the paper's claim that SMART 'takes comparable time to popular energy balance methods to run'.

## Related tools

- **SMART** (Target code whose per-galaxy and total wall-clock runtime is measured while it performs Bayesian MCMC SED fitting with radiative transfer libraries) — https://github.com/ch-var/SMART
- **Python** (Used to assemble the per-object and total runtime comparison table and write runtime_comparison.csv) — https://github.com/ch-var/SMART.git
- **Spitzer** (Source of the spectroscopy included in the HERUS ULIRG SEDs being fit during the timed runs)

## Examples

```
df.to_csv('runtime_comparison.csv')
```

## Evaluation signals

- runtime_comparison.csv contains 42 galaxy entries plus summary rows
- SMART per-sample total runtime is the same order of magnitude as the energy balance method's reported runtime
- Timings were recorded per galaxy as well as for the full sample, not just as one aggregate number
- Reference-method runtime value matches the number reported in the published paper text
- SMART run used its standard setup (four pre-computed libraries, Spitzer spectroscopy included) with no artificial speed-up or slow-down

## Limitations

- The comparison claim is order-of-magnitude comparability, not a controlled head-to-head benchmark: the energy balance method runtime is taken from literature-reported values on comparable work, not measured on identical hardware and inputs.
- SMART v1.0 as benchmarked is developed in Python 3.10 (Anaconda), so absolute runtimes depend on the interpreter/hardware environment and are not directly transferable across setups.
- The benchmark is specific to ULIRG SED fitting including Spitzer spectroscopy; runtimes for other galaxy types or data resolutions are not established by this result.

## Evidence

- [abstract] SMART runs in comparable time to popular energy balance methods despite fitting exclusively with radiative transfer models: "although it fits SEDs exclusively with radiative transfer models, it takes comparable time to popular energy balance methods to run"
- [abstract] Benchmark sample: 42 HERUS ULIRGs with Spitzer spectroscopy: "test it by fitting the multiwavelength SEDs of the 42 local ultraluminous infrared galaxies (ULIRGs) that constitute the HERschel Ultraluminous Infrared Galaxy Survey (HERUS) sample"
- [readme] Language/environment of the timed code: [evidence span withheld — anchor status: not_found]
- [abstract] Fitting procedure being timed uses four radiative transfer model libraries: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,"
