---
name: model-parameter-extraction-from-paper
description: Use when a repository's configuration/parameter file (e.g. g2myparms.m)
  is missing or must be rebuilt, and the authoritative parameter values are listed
  in the accompanying paper such as ./paper/paper.pdf.
when_to_use_negative:
- The parameter file already exists and matches the paper — only rebuild when the
  configuration module is missing or inconsistent
- The parameters you need are not documented in the companion paper and would have
  to be guessed or tuned empirically
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Matlab 8.0 (R2012b)
  role: Runtime in which g2myparms.m is authored and validated by running g2test from
    the galax2d directory
- name: galax2d
  role: Repository under reconstruction; supplies g2test, the paper directory ./paper/paper.pdf,
    and the figures4paper/makefigs workflow
  repo: https://github.com/Someone789/galax2d
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.03.005
    title: 2-D stationary gas dynamics in a barred galaxy
  source_content_hash: sha256:aa8bdffe88eeef3c9f81a79e37b161697d4fdd95df2b16eaa1bb1b20f6b7c976
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Someone789/galax2d
  tools:
  - Matlab 8.0 (R2012b)
  - galax2d
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# model-parameter-extraction-from-paper

## Summary

Reconstructs a model's parameter configuration module (e.g. g2myparms.m in the galax2d Matlab repository) by extracting the physical and numerical parameters from the companion paper (./paper/paper.pdf) and exposing each one as a named, editable field populated with the paper's exact value.

## When to use

Use when a repository's configuration/parameter file (e.g. g2myparms.m) is missing or must be rebuilt, and the authoritative parameter values are listed in the accompanying paper such as ./paper/paper.pdf.

## When NOT to use

- The parameter file already exists and matches the paper — only rebuild when the configuration module is missing or inconsistent
- The parameters you need are not documented in the companion paper and would have to be guessed or tuned empirically

## Inputs

- Companion paper listing model parameters, e.g. ./paper/paper.pdf in the galax2d repository
- galax2d Matlab source tree (Someone789/galax2d)

## Outputs

- g2myparms.m — a Matlab function returning a parameter structure with one editable named field per paper-listed parameter, each set to the paper's exact value

## How to apply

Open the companion paper (for galax2d, ./paper/paper.pdf) and enumerate the physical and numerical parameters of the 2-D compressible isothermal Euler gas-dynamics model in a galactic potential with a rotating weak bar. Author the configuration module (g2myparms.m) as a Matlab function that returns a parameter structure with one editable named field per parameter listed in the paper. Populate each named field with the exact value reported in the paper rather than defaults or approximations. Validate by starting Matlab 8.0 (R2012b) or later from the galax2d directory, running g2test (cd run; g2test;), and confirming the rebuilt module loads without error; note g2test can take one or two hours depending on Matlab version and hardware.

## Related tools

- **Matlab 8.0 (R2012b)** (Runtime in which g2myparms.m is authored and validated by running g2test from the galax2d directory)
- **galax2d** (Repository under reconstruction; supplies g2test, the paper directory ./paper/paper.pdf, and the figures4paper/makefigs workflow) — https://github.com/Someone789/galax2d

## Examples

```
cd run; g2test;
```

## Evaluation signals

- Every named field in g2myparms.m corresponds to a parameter listed in the paper (no extra, no missing fields)
- Each field value matches the value reported in ./paper/paper.pdf exactly
- cd run; g2test; executes in Matlab 8.0 (R2012b) or later without errors when loading g2myparms.m
- Values written via g2myparms.m propagate when recreating paper figures via figures4paper/ and makefigs

## Limitations

- Validation via g2test can take one or two hours depending on Matlab version and hardware
- Requires Matlab 8.0 (R2012b) or later; older Matlab versions are not supported
- No changelog was found for the galax2d repository, so version drift between the paper and the code cannot be ruled out

## Evidence

- [other] Task finding on configuration module: [evidence span withheld — anchor status: not_found]
