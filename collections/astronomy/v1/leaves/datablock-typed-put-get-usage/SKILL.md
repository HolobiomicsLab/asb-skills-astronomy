---
name: datablock-typed-put-get-usage
description: 'Use when building or testing a CosmoSIS pipeline: you need to seed a
  DataBlock with chosen parameters, or a module must read/write typed values (e.g.
  get_int, get_double_array_1d) in section/name keys before handing the block to the
  next module.'
when_to_use_negative:
- You need to replace an existing DataBlock value with a different type — DataBlocks
  only allow same-type replacement
- You need key lookups with mixed-case or non-string keys — keys must be strings and
  are always treated as lower-case
- You are passing complex structured objects rather than int, double, str, or complex
  scalars/arrays, which are the only supported value types
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CosmoSIS
  role: Provides the DataBlock class and typed put/get API used to pass parameters
    between pipeline modules
  repo: https://github.com/cosmosis-developers/cosmosis
- name: Python
  role: Host language for the core CosmoSIS python library (pip install cosmosis)
    in which DataBlock put/get calls are made
  repo: https://github.com/cosmosis-developers/cosmosis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.005
    title: 'CosmoSIS: Modular cosmological parameter estimation'
  source_content_hash: sha256:633acf995bbc2839adc212a782e717ba0b2759de212040b1425f27d419828b87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cosmosis-developers/cosmosis
  tools:
  - CosmoSIS
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# datablock-typed-put-get-usage

## Summary

A method for constructing and passing CosmoSIS DataBlocks: creating an empty DataBlock, writing typed values (int, double, str, complex scalars or arrays) via dictionary-style two-key section/name indexing or put methods, and reading them back with typed get methods so that a DataBlock can flow module-by-module through a pipeline.

## When to use

Use when building or testing a CosmoSIS pipeline: you need to seed a DataBlock with chosen parameters, or a module must read/write typed values (e.g. get_int, get_double_array_1d) in section/name keys before handing the block to the next module.

## When NOT to use

- You need to replace an existing DataBlock value with a different type — DataBlocks only allow same-type replacement
- You need key lookups with mixed-case or non-string keys — keys must be strings and are always treated as lower-case
- You are passing complex structured objects rather than int, double, str, or complex scalars/arrays, which are the only supported value types

## Inputs

- Chosen parameter values as Python scalars or arrays (int, double, str, complex) with two-key section/name string keys
- A DataBlock produced by an upstream module (for chained module-to-module execution)

## Outputs

- A DataBlock containing typed section/name keys (lower-case strings) that can be passed along the pipeline
- Typed values retrieved via get_int, get_double_array_1d, or equivalent typed get methods

## How to apply

Create an empty block with cosmosis.DataBlock() (core library, installable with 'pip install cosmosis') and fill it using dictionary-style two-key indexing like block['section','name'] = value, or the equivalent put methods. Keys must be strings and are always treated as lower-case; values are limited to int, double, str, and complex scalars or arrays. Read values back with typed get methods such as get_int or get_double_array_1d, which enforce type consistency — once a value is added it can only be replaced with something of the same type. If a get call supplies a default value, that default is afterwards written into the DataBlock for future look-ups to maintain consistency. To validate, run the DataBlock through a sequence of module objects (each exposing execute(datablock)) and confirm the final block contains the keys and value types written by earlier modules, with keys stored as lower-case strings.

## Related tools

- **CosmoSIS** (Provides the DataBlock class and typed put/get API used to pass parameters between pipeline modules) — https://github.com/cosmosis-developers/cosmosis
- **Python** (Host language for the core CosmoSIS python library (pip install cosmosis) in which DataBlock put/get calls are made) — https://github.com/cosmosis-developers/cosmosis

## Examples

```
import cosmosis; block = cosmosis.DataBlock(); block['section','b'] = 3; print(block.get_int('section','b')); print(block.get_double_array_1d('section','c'))
```

## Evaluation signals

- The run completes without error when the DataBlock is passed from module to module in declared order
- Final DataBlock contains exactly the keys and value types written by module A and modified by module B
- All keys in the DataBlock are stored as lower-case strings
- Attempting to replace an existing value with a different type fails, confirming type-consistency enforcement
- Typed get calls (e.g. get_int, get_double_array_1d) return values of the declared type

## Limitations

- DataBlock keys must be strings and are always treated as lower-case
- Values are limited to int, double, str, and complex scalars or arrays
- Once a value is added it can only be replaced with something of the same type
- A default value used in a get call is silently added to the DataBlock for future look-ups

## Evidence

- [results] the keys must be strings, and are always treated as lower-case: "the keys must be strings, and are always treated as lower-case"
- [results] their values are limited to int, double, str, and complex scalars or arrays: "their values are limited to int, double, str, and complex scalars or arrays"
- [results] once a value is added it can only be replaced with something of the same type: "once a value is added it can only be replaced with something of the same type"
- [results] You put a `DataBlock` in at the start of the pipeline, with your choice of parameters: "A pipeline is then a sequence of module objects.  You put a `DataBlock` in at the start of the pipeline, with your choice of parameters, and it is passed along from module to module, one by one."
- [readme] full documentation, including installation instructions: "You can find full documentation, including installation instructions, at https://cosmosis.readthedocs.io"
