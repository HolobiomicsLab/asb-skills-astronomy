---
name: cross-module-data-passing-testing
description: Use when building or testing an in-memory store that passes calculation
  results between sequential pipeline modules, especially when values must be keyed
  by (section, name) pairs and retrieved via typed getters.
when_to_use_negative:
- When data must be shared across processes or persisted to disk — the DataBlock is
  an in-memory container passed module to module, not a storage format
- When values of mixed types must be stored under the same (section, name) key — replacement
  must always be with the same type
- When keys need to preserve case or be non-string types — keys must be strings and
  are always treated as lower-case
tools:
- name: CosmoSIS
  role: Provides the DataBlock container and its dictionary-style two-key get/put
    API that the skill implements and tests
  repo: https://github.com/cosmosis-developers/cosmosis
- name: Python
  role: Language used to implement the DataBlock class, write the unittest/pytest
    test script, and run the pipeline simulation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.005
    title: 'CosmoSIS: Modular cosmological parameter estimation'
  source_content_hash: sha256:9efb6135f810e376d980e084d36a70c164b8bb4afd6821feecbfaabe268589ca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cosmosis-developers/cosmosis
  tools:
  - CosmoSIS
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-module-data-passing-testing

## Summary

Implements and tests a CosmoSIS-style DataBlock, an in-memory dictionary keyed by (section, name) string pairs that carries module calculation results between pipeline stages, with typed get/put access and type-preservation rules.

## When to use

Use when building or testing an in-memory store that passes calculation results between sequential pipeline modules, especially when values must be keyed by (section, name) pairs and retrieved via typed getters.

## When NOT to use

- When data must be shared across processes or persisted to disk — the DataBlock is an in-memory container passed module to module, not a storage format
- When values of mixed types must be stored under the same (section, name) key — replacement must always be with the same type
- When keys need to preserve case or be non-string types — keys must be strings and are always treated as lower-case

## Inputs

- Empty DataBlock created via cosmosis.DataBlock()
- Scalar values of type int, double, str, and complex
- 1-D arrays of int, double, str, or complex values
- Two-key (section, name) string identifiers

## Outputs

- DataBlock containing typed values retrievable via get_int, get_double_array_1d, and equivalent typed getters
- Round-trip test output printing retrieved values (e.g. block.get_int('section','b'), block.get_double_array_1d('section','c'))

## How to apply

Implement a DataBlock class whose keys must be strings and are always treated as lower-case, using two-key (section, name) dictionary-style indexing plus equivalent put methods. Restrict stored values to int, double, str, and complex scalars or 1-D arrays, and enforce that once a value is added it can only be replaced with something of the same type. Provide typed getters such as get_int and get_double_array_1d; when a default is supplied in a get call, afterwards include that default in the datablock for future look-ups to maintain consistency. Verify correctness by constructing an empty block with cosmosis.DataBlock(), writing values of each allowed type into a 'section', reading them back with the typed getters, and then simulating a pipeline where a first module writes results and a second, independent module reads them, mimicking a DataBlock being passed along from module to module.

## Related tools

- **CosmoSIS** (Provides the DataBlock container and its dictionary-style two-key get/put API that the skill implements and tests) — https://github.com/cosmosis-developers/cosmosis
- **Python** (Language used to implement the DataBlock class, write the unittest/pytest test script, and run the pipeline simulation)

## Examples

```
block = cosmosis.DataBlock(); block['section', 'b'] = 42; print(block.get_int('section', 'b')); print(block.get_double_array_1d('section', 'c'))
```

## Evaluation signals

- Typed getters return exactly the values written: block.get_int('section','b') and block.get_double_array_1d('section','c') reproduce the stored values
- Attempting to replace a stored value with a different type fails or is rejected, enforcing type preservation
- A default supplied in a get call appears in subsequent look-ups of the same (section, name) key
- A second, independent module can read values written by a first module through the same DataBlock instance (pipeline round-trip)
- Keys are normalized to lower-case regardless of input case

## Limitations

- DataBlock values are limited to int, double, str, and complex scalars or arrays — no nested or arbitrary types
- Once a value is added it can only be replaced with something of the same type
- Keys must be strings and are always treated as lower-case

## Evidence

- [results] keys must be strings and are always treated as lower-case: "the keys must be strings, and are always treated as lower-case"
- [results] value type restriction and replacement rule: "their values are limited to int, double, str, and complex scalars or arrays"
- [results] default value persistence on get: [evidence span withheld — anchor status: not_found]
- [results] typed getter invocation example: "print(block.get_int('section', 'b'))"
- [results] DataBlock passes between modules: "You put a `DataBlock` in at the start of the pipeline, with your choice of parameters, and it is passed along from module to module, one by one."
- [readme] documentation location from README: "You can find full documentation, including installation instructions, at https://cosmosis.readthedocs.io"
