---
name: cosmological-software-api-usage
description: Use when a task requires chaining cosmological calculation stages (e.g.
  parameter modules that read/write a shared state object) in a fixed order using
  the CosmoSIS Python API and its DataBlock data flow.
when_to_use_negative:
- The task needs heterogeneous value types stored under one DataBlock key — values
  can only be replaced with something of the same type
- Uppercase or non-string keys are required — DataBlock keys must be strings and are
  always treated as lower-case
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: CosmoSIS
  role: Core Python library providing DataBlock and pipeline control structure for
    parameter estimation
  repo: https://github.com/cosmosis-developers/cosmosis
- name: Python
  role: Language in which the core CosmoSIS library is installed via pip and modules
    are written
- name: C
  role: Alternative language for accessing DataBlocks via C-callable functions
- name: Fortran
  role: Alternative language for accessing DataBlocks via C-callable functions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.005
    title: 'CosmoSIS: Modular cosmological parameter estimation'
  source_content_hash: sha256:3137e043ed559851a3bc45b9dae5c50af2c0f4c583d789282b45cfbf50ad83a4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cosmosis-developers/cosmosis
  tools:
  - C
  - CosmoSIS
  - Fortran
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# cosmological-software-api-usage

## Summary

Constructs and runs a CosmoSIS scientific inference calculation by wiring reusable module objects into an ordered pipeline and passing a parameter-carrying DataBlock from module to module via the core CosmoSIS Python library.

## When to use

Use when a task requires chaining cosmological calculation stages (e.g. parameter modules that read/write a shared state object) in a fixed order using the CosmoSIS Python API and its DataBlock data flow.

## When NOT to use

- The task needs heterogeneous value types stored under one DataBlock key — values can only be replaced with something of the same type
- Uppercase or non-string keys are required — DataBlock keys must be strings and are always treated as lower-case

## Inputs

- Parameter choices as two-key section/name entries in a cosmosis.DataBlock
- Ordered list of module objects each exposing execute(datablock)
- Typed values: int, double, str, complex scalars or 1D arrays

## Outputs

- Final DataBlock containing keys and value types written by upstream modules and transformed by downstream modules
- Completed pipeline run without error

## How to apply

Install the core library with 'pip install cosmosis' and create an empty block with cosmosis.DataBlock(), filling it with chosen parameters using dictionary-style two-key section/name indexing or equivalent put methods; keys must be lower-case strings and values are limited to int, double, str, and complex scalars or arrays. Define a pipeline control loop that stores an ordered list of module objects, each exposing an execute(datablock) entry point. Write producer modules that write new typed values into the incoming DataBlock under a named section, and consumer modules that read those values with typed get methods (e.g. get_int, get_double_array_1d), transform them, and write results back into the same DataBlock. Run the pipeline by inserting the prepared DataBlock at the start and passing it along from module to module, one by one, in declared order. Remember that once a value is added it can only be replaced with something of the same type, and defaults supplied in a get call are afterwards included in the datablock for future look-ups to maintain consistency.

## Related tools

- **CosmoSIS** (Core Python library providing DataBlock and pipeline control structure for parameter estimation) — https://github.com/cosmosis-developers/cosmosis
- **Python** (Language in which the core CosmoSIS library is installed via pip and modules are written)
- **C** (Alternative language for accessing DataBlocks via C-callable functions)
- **Fortran** (Alternative language for accessing DataBlocks via C-callable functions)

## Examples

```
import cosmosis; block = cosmosis.DataBlock(); block['section', 'b'] = 3; print(block.get_int('section', 'b'))
```

## Evaluation signals

- Pipeline run completes without error
- Final DataBlock contains the keys and value types written by module A and modified by module B
- All DataBlock keys are stored as lower-case strings
- Values read back via typed get methods (get_int, get_double_array_1d) match the types written via put methods

## Limitations

- DataBlock values are limited to int, double, str, and complex scalars or arrays
- Once a value is added to a DataBlock it can only be replaced with something of the same type
- DataBlock keys must be strings and are always treated as lower-case
- A default value used in a get call is afterwards included in the datablock for future look-ups, which can mask absent inputs

## Evidence

- [results] A pipeline is then a sequence of module objects. You put a `DataBlock` in at the start of the pipeline, with your choice of parameters, and it is passed along from module to module, one by one.: "A pipeline is then a sequence of module objects.  You put a `DataBlock` in at the start of the pipeline, with your choice of parameters, and it is passed along from module to module, one by one."
- [results] the keys must be strings, and are always treated as lower-case: "the keys must be strings, and are always treated as lower-case"
- [results] their values are limited to int, double, str, and complex scalars or arrays: "their values are limited to int, double, str, and complex scalars or arrays"
- [results] It only requires the core CosmoSIS python library, which can be installed with `pip install cosmosis`.: "It only requires the core CosmoSIS python library, which can be installed with `pip install cosmosis`."
- [results] once a value is added it can only be replaced with something of the same type: "once a value is added it can only be replaced with something of the same type"
