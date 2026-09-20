---
name: task-parsing
description: Use when building or extending a websocket-based scientific server that
  must accept, validate, authenticate, and queue task-execution requests (e.g. send_task
  messages with module/task/parameters) from browser clients.
when_to_use_negative:
- When the client request is not a task-execution job (e.g. image loading, data-source
  creation, or admin settings changes) — those go through other Toyz web handlers,
  not the send_task queue
- When modules to be executed do not follow the enforced naming convention ending
  in tasks.py — such module names must be rejected rather than queued
- When unauthenticated or cookie-less requests arrive — the user must be authenticated
  from the secure cookie before a job is queued
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3077
tools:
- name: Toyz
  role: Web application framework whose toyz.web.app and toyz.web.tasks modules implement
    the server-side job API being reconstructed
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Implementation language for the Tornado websocket handler, parsing, and subprocess
    execution of tasks
- name: Tornado
  role: Web framework used to run the websocket handler that spawns a session process
    per client connection and routes send_task messages
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:d6a0351068a93912e3658c2592909e9200a9a673a8d99b9d938b17b7442f404c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Python
  - Tornado
  - Toyz
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# -task-parsing

## Summary

Parses client-submitted job requests on the Toyz Tornado server: each incoming send_task message is read as a JSON dictionary with module, task, and parameters keys, authenticated against the secure-cookie user, and queued for execution in a per-session subprocess.

## When to use

Use when building or extending a websocket-based scientific server that must accept, validate, authenticate, and queue task-execution requests (e.g. send_task messages with module/task/parameters) from browser clients.

## When NOT to use

- When the client request is not a task-execution job (e.g. image loading, data-source creation, or admin settings changes) — those go through other Toyz web handlers, not the send_task queue
- When modules to be executed do not follow the enforced naming convention ending in tasks.py — such module names must be rejected rather than queued
- When unauthenticated or cookie-less requests arrive — the user must be authenticated from the secure cookie before a job is queued

## Inputs

- send_task message from client websocket, dictionary with keys module, task, parameters
- secure cookie identifying the authenticated user
- per-session job queue
- toyz_settings configuration passed to each task subprocess

## Outputs

- JSON response over the websocket for every completed job, containing at minimum a status key
- tid dictionary (user_id, session_id, request_id) attached to each queued job
- destroyed subprocess after job completion

## How to apply

In a Tornado websocket handler, spawn a dedicated session process per client connection identified by a unique session_id. On each incoming send_task message, parse the payload as a JSON dictionary and require the module, task, and parameters keys, enforcing that all executable module names end in tasks.py before acceptance. Authenticate the user from the secure cookie, build a tid dictionary containing user_id, session_id, and request_id, and place the authenticated job in the per-session queue. Execute each queued job in a new subprocess, passing toyz_settings, tid, and params to the task function, and destroy the subprocess when the job completes. Send a JSON response back over the websocket for every completed job, including at minimum a status key. The reference implementation lives in the toyz.web.app and toyz.web.tasks modules of the fred3m/toyz repository.

## Related tools

- **Toyz** (Web application framework whose toyz.web.app and toyz.web.tasks modules implement the server-side job API being reconstructed) — https://github.com/fred3m/toyz
- **Python** (Implementation language for the Tornado websocket handler, parsing, and subprocess execution of tasks)
- **Tornado** (Web framework used to run the websocket handler that spawns a session process per client connection and routes send_task messages)

## Examples

```
git clone git://github.com/fred3m/toyz.git && cd toyz && toyz --port=<remote_port>
```

## Evaluation signals

- script_runs validation: the server starts, accepts one websocket connection, executes one task in a subprocess, and the client receives a JSON response containing a status key
- Every queued job carries a tid dictionary with user_id, session_id, and request_id populated before execution
- Any send_task payload whose module name does not end in tasks.py is rejected and never queued
- Each completed job's subprocess no longer exists after the JSON response with a status key is sent

## Limitations

- Executable module names must end in tasks.py, so tasks stored under other module names cannot be dispatched
- Jobs must be verified for authenticity before queueing; unverified or unauthenticated submissions are not run for the session
- Each job runs in a fresh subprocess that is destroyed on completion, so per-job state does not persist between submitted tasks

## Evidence

- [results] Jobs sent from a client to the Toyz server are verified for authenticity and put in a queue to be run for the session that submitted them.: [evidence span withheld — anchor status: not_found]
- [methods] Clone the Toyz source repository and inspect toyz.web.app and toyz.web.tasks modules as the reference for the server-side job API.: [evidence span withheld — anchor status: not_found]
scientists by allowing them to interact with their data via a web browser and run
scripts on a remote server"
