---
name: python-subprocess-orchestration
description: Use when a websocket-based web application must execute user-submitted
  tasks such that variables defined in an earlier task remain in memory for later
  tasks on the same connection, but each session's state must be isolated from other
  sessions and from the web server process.
when_to_use_negative:
- Tasks are independent one-shot jobs whose state need not persist across requests
  — the original per-job subprocess (destroyed on completion) is sufficient
- The analysis runs entirely client-side; Toyz exists so users can interact with data
  stored on a remote server without installing local software, so local-only analysis
  gains nothing from this session layer
- The task module is not granted permission for the user or does not end in tasks.py
  — the dispatch path must refuse it rather than spawn/route it
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: Toyz
  role: Web application framework whose websocket job API and session registry are
    modified to run one Python subprocess per session
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Language of the subprocesses that hold per-session variable state and execute
    task modules
  repo: https://github.com/fred3m/toyz
- name: Tornado
  role: Web application layer from which the per-session subprocess state is kept
    isolated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:f71dbc71414070e337f12d9f46617fcadaff2d2a8eb0a29452863788da54a961
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
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# python-subprocess-orchestration

## Summary

Reconstructs a workspace-as-process layer in which each client session (browser window/tab) runs as a dedicated Python subprocess on the server, persisting the user's in-memory variables across sequential tasks while remaining isolated from the Tornado web application layer, as implemented in the Toyz framework.

## When to use

Use when a websocket-based web application must execute user-submitted tasks such that variables defined in an earlier task remain in memory for later tasks on the same connection, but each session's state must be isolated from other sessions and from the web server process.

## When NOT to use

- Tasks are independent one-shot jobs whose state need not persist across requests — the original per-job subprocess (destroyed on completion) is sufficient
- The analysis runs entirely client-side; Toyz exists so users can interact with data stored on a remote server without installing local software, so local-only analysis gains nothing from this session layer
- The task module is not granted permission for the user or does not end in tasks.py — the dispatch path must refuse it rather than spawn/route it

## Inputs

- send_task JSON messages from a websocket connection (keys: module, task, parameters)
- Session registry keys: user_id, session_id (unique per websocket connection), request_id
- Allowed server-side task modules (only modules ending in tasks.py)
- Per-user module execution permissions

## Outputs

- A dedicated Python subprocess per websocket session holding persistent variable state
- A response message returned to the client for every completed job
- Subprocess termination on websocket disconnect

## How to apply

Inspect the Tornado-based web application (toyz.web.app and toyz.web.tasks in github.com/fred3m/toyz) to identify how the job API currently runs each received task in a new subprocess destroyed on completion. Build a session registry keyed by (user_id, session_id, request_id), where session_id is the unique id generated for each websocket connection. Modify the job dispatch path so the first send_task message (JSON with module, task, parameters keys) from a websocket spawns a dedicated Python subprocess for that session instead of a per-job subprocess; route subsequent tasks from the same connection to that subprocess so earlier variables persist. Permit only modules the user is granted permission to execute and only modules ending in tasks.py, keep the subprocess state isolated from the Tornado layer, and destroy the subprocess when the websocket disconnects.

## Related tools

- **Toyz** (Web application framework whose websocket job API and session registry are modified to run one Python subprocess per session) — https://github.com/fred3m/toyz
- **Python** (Language of the subprocesses that hold per-session variable state and execute task modules) — https://github.com/fred3m/toyz
- **Tornado** (Web application layer from which the per-session subprocess state is kept isolated)

## Examples

```
python setup.py install; toyz --port=<remote_port>
```

## Evaluation signals

- A test client sends sequential tasks over one websocket and confirms a variable created in the first task is retrievable in the second task
- A second websocket session cannot read the first session's variables (cross-session isolation holds)
- The server returns a response for every completed job
- The subprocess is cleaned up (destroyed) when the websocket disconnects
- Dispatch refuses modules the user lacks permission to execute and any module not ending in tasks.py

## Limitations

- Subprocess state persists only for the duration of the websocket connection; variables are lost once the session disconnects
- Only permitted modules ending in tasks.py may be executed, so arbitrary user code cannot be dispatched
- Session subprocesses must be kept isolated from the Tornado web application layer; a leaked subprocess on disconnect would waste server resources

## Evidence

- [other] Each workspace runs as a separate Python process, isolated from the web application layer, in which the state of a user's variables is stored for the duration of the connection.: [evidence span withheld — anchor status: not_found]
