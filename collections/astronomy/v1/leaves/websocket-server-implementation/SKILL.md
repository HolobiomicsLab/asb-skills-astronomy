---
name: websocket-server-implementation
description: Use when building a browser-based scientific tool that must run analysis
  tasks on the server storing the data, and clients need authenticated, per-session
  asynchronous job execution over websockets.
when_to_use_negative:
- The analysis must run locally on the client machine — Toyz's design premise is that
  existing visualization tools 'must be run on a local machine with data stored locally,
  which is no longer feasible', so this skill applies only to server-side execution
  on the host storing the data
- The task module is not a tasks.py module — the server only executes module names
  that end in tasks.py
- The client cannot use browser-based websocket access — Toyz requires that 'Users
  connect to the Toyz web application via a web browser'
edam_operation: null
tools:
- name: Toyz
  role: Open source Python web application framework providing the reference websocket/session/job-queue
    server (toyz.web.app, toyz.web.tasks) to reconstruct or extend
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Implementation language for the Tornado websocket handler, task subprocesses,
    and JSON message parsing
- name: Tornado
  role: Web framework hosting the websocket handler that spawns dedicated session
    processes and routes send_task messages to the per-session queue
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:60d3506f871e3fc964e8e1fde77f14ed12e1511b81656d2b3ab39258127a939c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Python
  - Tornado
  - Toyz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# websocket-server-implementation

## Summary

Implements a Tornado websocket server that spawns a per-session process for each client connection, authenticates and queues submitted jobs, executes them in subprocesses, and streams back JSON status responses — the control-loop pattern behind the Toyz web framework.

## When to use

Use when building a browser-based scientific tool that must run analysis tasks on the server storing the data, and clients need authenticated, per-session asynchronous job execution over websockets.

## When NOT to use

- The analysis must run locally on the client machine — Toyz's design premise is that existing visualization tools 'must be run on a local machine with data stored locally, which is no longer feasible', so this skill applies only to server-side execution on the host storing the data
- The task module is not a tasks.py module — the server only executes module names that end in tasks.py
- The client cannot use browser-based websocket access — Toyz requires that 'Users connect to the Toyz web application via a web browser'

## Inputs

- Client websocket connection initiating a session
- send_task message: JSON dictionary with module, task, and parameters keys
- Secure cookie containing user identity
- toyz_settings configuration dictionary

## Outputs

- JSON response over the websocket per completed job, containing at minimum a status key
- Per-session job queue keyed by session_id
- tid dictionary (user_id, session_id, request_id) per authenticated job
- Task subprocess destroyed after job completion

## How to apply

Clone the Toyz source (git clone git://github.com/fred3m/toyz.git) and use toyz.web.app and toyz.web.tasks as the reference server-side job API. Implement a Tornado websocket handler so that each new client connection spawns a dedicated session process identified by a unique session_id. Parse each incoming send_task message as a JSON dictionary with module, task, and parameters keys, enforcing that all executable module names end in tasks.py so only task modules are callable. Authenticate the user from the secure cookie, build the tid dictionary (user_id, session_id, request_id), and place the authenticated job in the per-session queue. Execute each queued job in a new subprocess, passing toyz_settings, tid, and params to the task function, destroy the subprocess on completion, and return a JSON response over the websocket containing at minimum a status key.

## Related tools

- **Toyz** (Open source Python web application framework providing the reference websocket/session/job-queue server (toyz.web.app, toyz.web.tasks) to reconstruct or extend) — https://github.com/fred3m/toyz
- **Python** (Implementation language for the Tornado websocket handler, task subprocesses, and JSON message parsing)
- **Tornado** (Web framework hosting the websocket handler that spawns dedicated session processes and routes send_task messages to the per-session queue)

## Examples

```
git clone git://github.com/fred3m/toyz.git && cd toyz && python setup.py install && toyz --port=<remote_port>
```

## Evaluation signals

- script_runs validation: the server starts, accepts one websocket connection, executes one task in a subprocess, and the client receives a JSON response containing a status key
- Every incoming send_task message parses as a JSON dictionary with module, task, and parameters keys before being queued
- Unauthenticated or non-tasks.py module names are rejected from the per-session queue
- Each executed job runs in a new subprocess that is destroyed when the job has completed
- Each response carries the tid fields (user_id, session_id, request_id) tying the result to the submitting session

## Limitations

- Executable task modules are restricted to names ending in tasks.py; arbitrary module names cannot be dispatched
- The Toyz admin account ships with a default password that all users, 'including and *ESPECIALLY* the **admin**', are advised to change — leaving it is a security failure mode
- Remote access requires ssh port forwarding setup (ssh -L<local_port>:localhost:<remote_port>) before launching the app on the matching remote port

## Evidence

- [methods] Implement a Tornado websocket handler such that each new client connection spawns a dedicated session process identified by a unique session_id.: [evidence span withheld — anchor status: not_found]
- [methods] Jobs verified and queued per session.: [evidence span withheld — anchor status: not_found]
