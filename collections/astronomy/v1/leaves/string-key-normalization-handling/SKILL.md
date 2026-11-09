---
name: string-key-normalization-handling
description: Use when implementing or extending an in-memory DataBlock-style store
  that carries calculation results between pipeline stages and needs deterministic,
  case-insensitive string keys and typed (int, double, str, complex) scalar or 1-D
  array access.
when_to_use_negative:
- When the analysis needs hierarchical or nested key structures — DataBlock supports
  only flat two-key (section, name) string indexing.
- When values of mixed types must be stored under the same key, since a value can
  only be replaced with something of the same type.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: CosmoSIS
  role: Provides the DataBlock class with lower-case string keys and typed get/put
    API that this skill implements and tests
  repo: https://github.com/cosmosis-developers/cosmosis
- name: Python
  role: Implementation and unittest/pytest test language for the DataBlock store and
    its key normalization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.005
    title: 'CosmoSIS: Modular cosmological parameter estimation'
  source_content_hash: sha256:b6daec961e5e2d3b55c673a49e2397dc77a0a03ce910acd36e24209c236d2dcb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cosmosis-developers/cosmosis
  tools:
  - CosmoSIS
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# string-key-normalization-handling

## Summary

Implements CosmoSIS-style key normalization for DataBlock containers: all keys must be strings and are always treated as lower-case, with two-key (section, name) indexing and typed get/put access. This guarantees consistent look-ups across modules in a pipeline regardless of the casing used by the writing module.

## When to use

Use when implementing or extending an in-memory DataBlock-style store that carries calculation results between pipeline stages and needs deterministic, case-insensitive string keys and typed (int, double, str, complex) scalar or 1-D array access.

## When NOT to use

- When the analysis needs hierarchical or nested key structures — DataBlock supports only flat two-key (section, name) string indexing.
- When values of mixed types must be stored under the same key, since a value can only be replaced with something of the same type.

## Inputs

- Python DataBlock object constructed with cosmosis.DataBlock()
- String (section, name) key pairs
- Values of type int, double, str, or complex as scalars or 1-D arrays

## Outputs

- Lower-case normalized string keys stored in the DataBlock
- Values retrieved via typed getters (e.g. get_int, get_double_array_1d)
- Defaults supplied to get calls persisted into the DataBlock for future look-ups
- Test script output printing round-trip retrieved values

## How to apply

Implement a DataBlock class whose keys must be strings and are always coerced to lower-case on both write and read paths, using two-key (section, name) dictionary-style indexing plus equivalent put methods. Restrict stored values to int, double, str, and complex scalars or 1-D arrays, and enforce that once a value is added it can only be replaced with something of the same type — raise an error on type mismatch rather than silently overwriting. Provide typed getters such as get_int and get_double_array_1d; when a default value is supplied to a get call, insert that default into the DataBlock so subsequent look-ups are consistent. Validate the implementation with a unittest/pytest script that builds an empty block via cosmosis.DataBlock(), writes values of each allowed type into a 'section', reads them back with typed getters, and simulates a first module writing and a second, independent module reading the same DataBlock.

## Related tools

- **CosmoSIS** (Provides the DataBlock class with lower-case string keys and typed get/put API that this skill implements and tests) — https://github.com/cosmosis-developers/cosmosis
- **Python** (Implementation and unittest/pytest test language for the DataBlock store and its key normalization)

## Examples

```
import cosmosis; block = cosmosis.DataBlock(); block['section','b'] = 7; print(block.get_int('section','b')); print(block.get_double_array_1d('section','c'))
```

## Evaluation signals

- Keys written with mixed case (e.g. 'Section', 'Name') are retrievable using any casing after lower-case normalization.
- Attempting to replace an existing value with a different type (e.g. int with double array) raises an error instead of overwriting.
- A get call with a default value (e.g. block.get_int('section','b', default=...)) causes the default to appear in subsequent look-ups of the same key.
- Round-trip test prints expected values from block.get_int('section','b') and block.get_double_array_1d('section','c') after writing each allowed type into 'section'.
- A second, independent module reads values written by a first module from the same DataBlock, mimicking pass-along from module to module.

## Limitations

- DataBlock values are limited to int, double, str, and complex scalars or arrays — other types cannot be stored.
- Once a value is added it can only be replaced with something of the same type, so type changes require a new key or block.
- Blocks are accessible from Python and from C, C++, Fortran, and Julia via C functions, but cross-language access must respect the same key normalization and typing rules.

## Evidence

- [results] default persistence in get: [evidence span withheld — anchor status: not_found]
