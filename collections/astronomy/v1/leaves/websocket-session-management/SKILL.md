---
name: websocket-session-management
description: Use when a Tornado/websocket-based web application (e.g. Toyz) must keep
  per-session server-side state so variables defined in one task remain in memory
  for later tasks over the same connection.
when_to_use_negative:
- Tasks are one-shot batch jobs that do not need variables from earlier requests,
  so the existing per-job subprocess destroyed on completion is sufficient
- The application has no websocket layer and no per-connection server-side state to
  isolate
- Session state can legitimately live client-side or in a shared store rather than
  in a per-session process
tools:
- name: Toyz
  role: Web application framework whose toyz.web.app and toyz.web.tasks modules are
    modified to add the session registry and per-session subprocess dispatch
  repo: github.com/fred3m/toyz
- name: Tornado
  role: Web server layer hosting the websocket connections; kept isolated from the
    session subprocesses
- name: Python
  role: Language in which each workspace/session runs as a separate process holding
    the user's variable state
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:3ae4d662701d1a493f620c6091860805ef9172cba88e6a66192222190ae856b2
schema_version: 0.2.0
metadata:
  license_tier: unknown
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

# websocket-session-management

## Summary

A method for giving each browser websocket connection to a Toyz (Tornado) server its own dedicated Python subprocess whose in-memory variable state persists across sequential tasks, replacing per-job subprocesses that are destroyed on completion.

## When to use

Use when a Tornado/websocket-based web application (e.g. Toyz) must keep per-session server-side state so variables defined in one task remain in memory for later tasks over the same connection.

## When NOT to use

- Tasks are one-shot batch jobs that do not need variables from earlier requests, so the existing per-job subprocess destroyed on completion is sufficient
- The application has no websocket layer and no per-connection server-side state to isolate
- Session state can legitimately live client-side or in a shared store rather than in a per-session process

## Inputs

- send_task JSON message with module, task, and parameters keys received over a websocket connection
- Websocket connection identifiers: user_id, session_id, request_id
- List of modules the user is permitted to execute (task modules ending in tasks.py)

## Outputs

- Session registry mapping session_id to a dedicated Python subprocess
- Server response returned for every completed job
- Persistent in-process variable state per browser window or tab
- Destroyed subprocess and registry entry on websocket disconnect

## How to apply

Inspect the Toyz web application (Tornado server, toyz.web.app and toyz.web.tasks modules) to find where the job API spawns a new subprocess per received task and destroys it on completion. Build a session registry keyed by the task id fields (user_id, session_id, request_id), where session_id is the unique id generated per websocket connection. Modify the dispatch path so the first send_task message (JSON with module, task, parameters keys) on a websocket spawns a dedicated session subprocess instead of a per-job subprocess, then route subsequent tasks from that connection to the same subprocess so variables persist. Enforce permissions at dispatch: only modules the user is granted permission to execute, and only modules ending in tasks.py. Keep the subprocess state isolated from the Tornado web application layer and destroy the subprocess when the websocket disconnects.

## Related tools

- **Toyz** (Web application framework whose toyz.web.app and toyz.web.tasks modules are modified to add the session registry and per-session subprocess dispatch) — github.com/fred3m/toyz
- **Tornado** (Web server layer hosting the websocket connections; kept isolated from the session subprocesses)
- **Python** (Language in which each workspace/session runs as a separate process holding the user's variable state)

## Examples

```
toyz --port=<remote_port>
```

## Evaluation signals

- A test client sends sequential tasks over one websocket and a variable created in the first task is retrievable in the second
- A second websocket session cannot read the first session's variables
- The server returns a response for every completed job
- The session subprocess is cleaned up when the websocket disconnects
- Dispatch rejects any module the user lacks permission to execute or that does not end in tasks.py

## Limitations

- Session state lives only in process memory for the duration of the connection, so it is lost when the websocket disconnects
- Each websocket session holds a dedicated subprocess, so many concurrent sessions increase server process count
- Only modules ending in tasks.py and explicitly permitted for the user may be executed, restricting what a session can run

## Evidence

- [other] Each workspace runs as a separate Python process, isolated from the web application layer, in which the state of a user's variables is stored for the duration of the connection.: [evidence span withheld — anchor status: not_found]
