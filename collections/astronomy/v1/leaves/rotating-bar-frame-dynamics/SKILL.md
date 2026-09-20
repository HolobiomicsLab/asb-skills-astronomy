---
name: rotating-bar-frame-dynamics
description: 'Use when the goal is a stationary gas-dynamic flow in a barred galaxy:
  isothermal Euler equations in 2-D with a galactic potential plus a rotating weak
  bar source term, to be solved in Matlab.'
when_to_use_negative:
- The problem is 3-D, non-isothermal (energy equation required), or excludes a rotating
  bar — the package is specific to 2-D isothermal Euler equations with a galactic
  potential and rotating weak bar
- A quick illustrative run is needed with strict time limits — g2test takes one or
  two hours depending on Matlab version and hardware
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: galax2d
  role: Matlab package that finds the stationary solution of the 2-D compressible
    isothermal Euler equations in a galactic potential with a rotating weak bar; run
    via cd run; g2test with parameters set in g2myparms.m
  repo: https://github.com/Someone789/galax2d
- name: Matlab 8.0 (R2012b)
  role: Runtime environment required (R2012b or later) to execute g2test and makefigs
    from the galax2d run and figures4paper directories
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.03.005
    title: 2-D stationary gas dynamics in a barred galaxy
  source_content_hash: sha256:d8c41e09eb69d9b8cd5e445538d76b672c027d304232fa3a027a2253e2ea32de
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Someone789/galax2d
  tools:
  - Matlab 8.0 (R2012b)
  - galax2d
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# rotating-bar-frame-dynamics

## Summary

Method for computing a stationary solution of the 2-D compressible isothermal Euler equations of gas dynamics in an axisymmetric galactic potential perturbed by a rotating weak bar, using the galax2d Matlab package (Mulder 2015, Astronomy and Computing).

## When to use

Use when the goal is a stationary gas-dynamic flow in a barred galaxy: isothermal Euler equations in 2-D with a galactic potential plus a rotating weak bar source term, to be solved in Matlab.

## When NOT to use

- The problem is 3-D, non-isothermal (energy equation required), or excludes a rotating bar — the package is specific to 2-D isothermal Euler equations with a galactic potential and rotating weak bar
- A quick illustrative run is needed with strict time limits — g2test takes one or two hours depending on Matlab version and hardware

## Inputs

- g2myparms.m parameter file (grid extents, sound speed, bar strength, bar pattern speed) from the galax2d run/ directory
- Matlab 8.0 (R2012b) or later environment

## Outputs

- Stationary 2-D solution of the compressible isothermal Euler equations in the galactic potential with a rotating weak bar
- Paper figures regenerated via makefigs in figures4paper/

## How to apply

Start Matlab 8.0 (R2012b) or later, change directory to the packaged run directory, and start g2test, which iterates toward a stationary solution of the 2-D compressible isothermal Euler equations in the galactic potential with the rotating weak bar; the run takes one to two hours depending on Matlab version and hardware. Before running, edit g2myparms.m to set the parameters (grid extents, sound speed, bar strength and pattern speed) documented in the paper (./paper/paper.pdf). To reproduce the paper's figures, switch the Matlab directory to figures4paper/ and run makefigs. Correctness is judged by the flow converging to a stationary state consistent with the parameters in g2myparms.m and by regenerated figures matching those in paper.pdf.

## Related tools

- **galax2d** (Matlab package that finds the stationary solution of the 2-D compressible isothermal Euler equations in a galactic potential with a rotating weak bar; run via cd run; g2test with parameters set in g2myparms.m) — https://github.com/Someone789/galax2d
- **Matlab 8.0 (R2012b)** (Runtime environment required (R2012b or later) to execute g2test and makefigs from the galax2d run and figures4paper directories)

## Examples

```
cd run; g2test;
```

## Evaluation signals

- The g2test run completes without error and produces a stationary flow (runtime of roughly one to two hours is expected depending on Matlab version and hardware)
- Parameters used match those listed in the paper (./paper/paper.pdf) as set in g2myparms.m
- makefigs in figures4paper/ reproduces the figures of the paper
- Solver behavior is consistent with the isothermal closure and the bar rotating at the pattern speed specified in g2myparms.m

## Limitations

- Runtime is significant: one or two hours for g2test depending on Matlab version and hardware
- Requires Matlab 8.0 (R2012b) or later; results may vary across Matlab versions
- Parameters are limited to those listed in the accompanying paper; no changelog is available for the repository

## Evidence
