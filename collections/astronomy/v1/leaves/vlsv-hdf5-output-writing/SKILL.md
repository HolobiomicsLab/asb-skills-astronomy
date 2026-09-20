---
name: vlsv-hdf5-output-writing
description: Use when an end-to-end Vlasiator-like solver run (Strang-split semi-Lagrangian
  Vlasov + constrained-transport field solver) has completed and the resulting ion
  distribution function and field data must be persisted to a declared VLSV, HDF5,
  or NumPy file.
when_to_use_negative:
- The solver run has not completed or errored — no partial-file writes should be validated
- The consumer requires a format outside the declared VLSV/HDF5/NumPy set
- The task requires quantitative validation against thresholds; this paper reports
  no quantitative threshold for the minimal test
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Vlasiator
  role: Global hybrid-Vlasov simulation whose run output is serialized to the VLSV/HDF5/NumPy
    file in this step
  repo: github.com/fmihpc/vlasiator
- name: analysator
  role: Python tools used to read and analyze the VLSV output files produced by this
    skill
  repo: https://github.com/fmihpc/analysator
- name: vlsv
  role: VLSV file-format library associated with Vlasiator output reading/writing
  repo: fmihpc/vlsv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:51ba5b87fce03d0f96cc8fa792f6fd5c3cfde0ea8e0c6c338f8431980311c267
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Vlasiator
  - analysator
  - vlsv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# vlsv-hdf5-output-writing

## Summary

Write the distribution-function and field data produced by a Vlasiator-style hybrid-Vlasov solver run to a declared data file format — VLSV (Vlasiator's native format), HDF5, or NumPy — so the run produces a parseable, verifiable output artifact.

## When to use

Use when an end-to-end Vlasiator-like solver run (Strang-split semi-Lagrangian Vlasov + constrained-transport field solver) has completed and the resulting ion distribution function and field data must be persisted to a declared VLSV, HDF5, or NumPy file.

## When NOT to use

- The solver run has not completed or errored — no partial-file writes should be validated
- The consumer requires a format outside the declared VLSV/HDF5/NumPy set
- The task requires quantitative validation against thresholds; this paper reports no quantitative threshold for the minimal test

## Inputs

- Distribution-function data (ion velocity distribution, e.g. from a reduced-velocity-space 1D test run)
- Field-solver output (upwind constrained-transport fields under the Darwin approximation)
- Driver script run state (completed Strang-split propagation and field-solve steps)

## Outputs

- A single data file in a declared format: VLSV, HDF5, or NumPy
- Parseable output that can be read back by Analysator (for VLSV) or standard NumPy/HDF5 readers

## How to apply

After the driver script finishes the coupled solver run (e.g. a small 1D test problem with reduced velocity space), serialize the written distribution-function slices and field quantities into one of the three accepted formats: VLSV, HDF5, or NumPy. Choose VLSV if downstream analysis with Vlasiator's tooling is intended, since Analysator's Python tools read and analyze VLSV files directly; HDF5 or NumPy are acceptable when a generic reader suffices. Because the paper reports no quantitative threshold for this minimal test, acceptance is qualitative: the driver script runs without error, the output file exists on disk, and the file is verifiably in the declared format (VLSV/HDF5/NumPy), not merely renamed or empty. Note that Vlasiator repositories use git submodules for dependent libraries, so clone with --recurse-submodules before building the code that produces the output.

## Related tools

- **Vlasiator** (Global hybrid-Vlasov simulation whose run output is serialized to the VLSV/HDF5/NumPy file in this step) — github.com/fmihpc/vlasiator
- **analysator** (Python tools used to read and analyze the VLSV output files produced by this skill) — https://github.com/fmihpc/analysator
- **vlsv** (VLSV file-format library associated with Vlasiator output reading/writing) — fmihpc/vlsv

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator
```

## Evaluation signals

- script_runs: the driver script executes end-to-end without error
- file_exists: the declared output file exists on disk after the run
- file_format_is: the written file is actually in the declared VLSV/HDF5/NumPy format (parseable, not a renamed/empty file)
- If VLSV was chosen: the file can be opened by Analysator's VlsvReader (pt.vlsvfile.VlsvReader)
- The output contains both distribution-function data and field data from the coupled run

## Limitations

- The paper reports no quantitative threshold for this minimal test — acceptance is only the qualitative check that the run completes and produces a parseable data file
- Vlasiator's dependent libraries are migrating to git submodules; cloning without --recurse-submodules and running git submodule update --init --recursive can leave the build incomplete
- VLSV files are specific to the Vlasiator ecosystem; analysator is needed for reading/analysis, with dependencies listed in its requirements.txt

## Evidence

- [abstract] A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere.: "A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [readme] Analysator is a set of Python tools and scripts used to read and analyze [Vlasiator](https://github.com/fmihpc/vlasiator) output files (VLSV).: [evidence span withheld — anchor status: not_found]
- [readme] by modelling ions as velocity distribution functions the outcome will be numerically noiseless.: [evidence span withheld — anchor status: external_doc]
