---
name: session-identifier-keyed-resource-tracking
description: Use when a Tornado-based web application must keep a user's variables
  alive across sequential tasks sent over one websocket, while preventing other browser
  windows or tabs from reading that state.
when_to_use_negative:
- Each task is independent and no cross-task variable persistence is needed — the
  existing per-job subprocess that is destroyed on completion is sufficient
- State must be shared between multiple browser windows or tabs of the same user rather
  than isolated per websocket connection
- Modules to execute are arbitrary Python rather than permitted modules ending in
  tasks.py
tools:
- name: Toyz
  role: Web application framework whose websocket job API and workspace mechanism
    are modified to spawn one subprocess per session keyed by session_id
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Language in which each isolated workspace session subprocess runs and in which
    the session registry is implemented
- name: Tornado
  role: Web application layer from which the per-session subprocess state must remain
    isolated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:82dc6977967d0e18ae657b74cf3d6eac8d557a6fc3a9f491c7d053d96f160cdd
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
yes_rate: 0.6
n_claims: 5
---

# Session-identifier-keyed resource tracking

## Summary

Isolates and persists per-connection server state by keying a registry of dedicated Python subprocesses on the task id fields (user_id, session_id, request_id), where session_id is the unique id generated for each websocket connection, so variables created in earlier tasks remain in memory for later tasks in the same Toyz session.

## When to use

Use when a Tornado-based web application must keep a user's variables alive across sequential tasks sent over one websocket, while preventing other browser windows or tabs from reading that state.

## When NOT to use

- Each task is independent and no cross-task variable persistence is needed — the existing per-job subprocess that is destroyed on completion is sufficient
- State must be shared between multiple browser windows or tabs of the same user rather than isolated per websocket connection
- Modules to execute are arbitrary Python rather than permitted modules ending in tasks.py

## Inputs

- Websocket send_task messages (JSON with module, task, parameters keys)
- Task id fields: user_id, session_id, request_id
- List of user-permitted modules (only modules ending in tasks.py)
- Toyz source tree (toyz.web.app, toyz.web.tasks)

## Outputs

- Session registry mapping (user_id, session_id, request_id) to a dedicated Python subprocess
- Per-session workspace whose variables persist across sequential tasks for the duration of the connection
- A response returned for every completed job
- Subprocess teardown when the websocket disconnects

## How to apply

Inspect the web application layer (toyz.web.app and toyz.web.tasks in github.com/fred3m/toyz) to identify how the job API currently runs each received task in a new subprocess destroyed on completion. Build a session registry keyed by (user_id, session_id, request_id), treating session_id as the unique id generated for each websocket connection. Change the dispatch path so the first send_task message (JSON with module, task, parameters keys) from a websocket spawns a dedicated Python subprocess for that session instead of a per-job subprocess, and route subsequent tasks from the same connection to that subprocess. Restrict execution to modules the user is granted permission to run and only modules ending in tasks.py. Keep the subprocess state isolated from the Tornado web application layer and destroy the subprocess when the websocket disconnects; the rationale is that the workspace runs as a separate Python process in which the state of a user's variables is stored for the duration of the connection.

## Related tools

- **Toyz** (Web application framework whose websocket job API and workspace mechanism are modified to spawn one subprocess per session keyed by session_id) — https://github.com/fred3m/toyz
- **Python** (Language in which each isolated workspace session subprocess runs and in which the session registry is implemented)
- **Tornado** (Web application layer from which the per-session subprocess state must remain isolated)

## Examples

```
toyz --port=<remote_port>
```

## Evaluation signals

- A test client sends sequential tasks over one websocket and a variable created in the first task is retrievable in the second
- A second websocket session cannot read the first session's variables (cross-session isolation holds)
- The server returns a response for every completed job
- Subprocess is cleaned up when the websocket disconnects (no orphaned processes)
- Only permitted modules ending in tasks.py can be executed through the session

## Limitations

- The subprocess lifecycle is tied to the websocket connection, so state does not survive a disconnect or a new browser window/tab opening a new session_id
- State is held in the memory of the session subprocess and is isolated from the Tornado web application layer, so the web layer cannot directly inspect workspace variables
- Notebook-style persistence across reconnects is not provided; each connection gets a fresh workspace

## Evidence

scientists by allowing them to interact with their data via a web browser and run
scripts on a remote server"
