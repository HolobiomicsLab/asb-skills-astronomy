---
name: pipeline-control-loop-reconstruction
description: Use when rebuilding or validating a modular scientific pipeline whose
  stages exchange state through a shared parameter block, or when reconstructing CosmoSIS's
  module/DataBlock execution loop from the article or repository.
when_to_use_negative:
- The task requires replacing an existing DataBlock value with a different type —
  once a value is added it can only be replaced with something of the same type.
- The workflow needs to pass data structures outside the supported set (int, double,
  str, and complex scalars or arrays) between modules.
- The target framework is not module/DataBlock based (e.g. a DAG-based workflow engine
  with no shared parameter block).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: CosmoSIS
  role: Core library providing DataBlock and the pipeline/module execution model;
    installed with 'pip install cosmosis'
  repo: https://github.com/cosmosis-developers/cosmosis
- name: Python
  role: Language in which the pipeline control loop, dummy modules, and DataBlock
    manipulations are written
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.005
    title: 'CosmoSIS: Modular cosmological parameter estimation'
  source_content_hash: sha256:10fbc5b5d6b2fab815d48ac4d6dc08342ed8e4e72c69dc10d4831dcdf9747413
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cosmosis-developers/cosmosis
  tools:
  - CosmoSIS
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pipeline-control-loop-reconstruction

## Summary

Reconstructs the CosmoSIS pipeline control structure, in which an ordered list of module objects each exposing an execute(datablock) entry point is driven by a DataBlock of chosen parameters that is passed from module to module, one by one. This is the core mechanism by which CosmoSIS composes scientific inference calculations from reusable components.

## When to use

Use when rebuilding or validating a modular scientific pipeline whose stages exchange state through a shared parameter block, or when reconstructing CosmoSIS's module/DataBlock execution loop from the article or repository.

## When NOT to use

- The task requires replacing an existing DataBlock value with a different type — once a value is added it can only be replaced with something of the same type.
- The workflow needs to pass data structures outside the supported set (int, double, str, and complex scalars or arrays) between modules.
- The target framework is not module/DataBlock based (e.g. a DAG-based workflow engine with no shared parameter block).

## Inputs

- Python DataBlock created with cosmosis.DataBlock() and populated via two-key section/name indexing or put methods
- Ordered list of module objects, each exposing an execute(datablock) entry point

## Outputs

- Final DataBlock containing keys and value types written by earlier modules and modified by later modules
- Completed pipeline run with no execution errors

## How to apply

First create an empty DataBlock with cosmosis.DataBlock() (core library installable via 'pip install cosmosis') and fill it with chosen parameters using dictionary-style two-key section/name indexing or the equivalent put methods; keys must be strings and are always treated as lower-case. Next define a Pipeline control loop that stores an ordered list of module objects, each exposing an execute(datablock) entry point. Write a producer module that writes new typed values (int, double, or arrays) into the incoming DataBlock under a named section, and a consumer module that reads those values with typed get methods (e.g. get_int, get_double_array_1d), transforms them, and writes results back into the same DataBlock. Run the pipeline by inserting the prepared DataBlock at the start and passing it along from module to module, one by one, in declared order. Respect the type system: values are limited to int, double, str, and complex scalars or arrays, and once a value is added it can only be replaced with something of the same type.

## Related tools

- **CosmoSIS** (Core library providing DataBlock and the pipeline/module execution model; installed with 'pip install cosmosis') — https://github.com/cosmosis-developers/cosmosis
- **Python** (Language in which the pipeline control loop, dummy modules, and DataBlock manipulations are written)

## Examples

```
import cosmosis; block = cosmosis.DataBlock(); block.put_int('section', 'a', 1); for m in modules: m.execute(block)
```

## Evaluation signals

- The pipeline run completes without error when the DataBlock is inserted at the start and passed module to module.
- The final DataBlock contains the keys written by module A and modified by module B.
- All DataBlock keys are stored as lower-case strings, matching the article's key-handling rule.
- Value types in the final DataBlock match the types written by module A (int, double, or arrays) and any replacements preserve type.
- Typed get calls (e.g. get_int, get_double_array_1d) in the consumer module succeed on the sections written by the producer module.

## Limitations

- DataBlock keys must be strings and are always treated as lower-case, so mixed-case lookups will not match.
- DataBlock values are limited to int, double, str, and complex scalars or arrays.
- Once a value is added to a DataBlock it can only be replaced with something of the same type.
- A default value used in a DataBlock get call is afterwards included in the datablock for future look-ups, which can surprise downstream modules.

## Evidence
