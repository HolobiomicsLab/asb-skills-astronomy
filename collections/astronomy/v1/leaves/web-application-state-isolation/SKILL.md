---
name: web-application-state-isolation
description: Use when a browser-based scientific web application must keep per-session
  variables alive across sequential tasks on a remote server, requiring session-scoped
  Python subprocess isolation from the web layer.
when_to_use_negative:
- Stateless, one-shot job execution where destroying the subprocess after each task
  is acceptable (the default Toyz per-job subprocess behavior suffices)
- 'Persistence that must outlive the browser connection: variable state lasts only
  for the duration of the websocket connection'
- Situations requiring shared state across different browser windows or tabs of unrelated
  users; each session's variables are isolated from other sessions
tools:
- name: Toyz
  role: Web application framework whose job API and websocket layer are modified to
    spawn session-scoped Python subprocesses
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Language of the isolated workspace subprocess in which the user's variables
    are stored
- name: Tornado
  role: Web application layer from which the session subprocess state must remain
    isolated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:edf8d0e8268662f0160c5122e767d11c7da7a86c4f4b361283313b4d8e971c5e
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
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# web-application-state-isolation

## Summary

Reconstructs the Toyz workspace-as-process layer: each browser websocket session is bound to a dedicated Python subprocess, kept separate from the Tornado web application layer, in which the user's variables persist for the duration of the connection.

## When to use

Use when a browser-based scientific web application must keep per-session variables alive across sequential tasks on a remote server, requiring session-scoped Python subprocess isolation from the web layer.

## When NOT to use

- Stateless, one-shot job execution where destroying the subprocess after each task is acceptable (the default Toyz per-job subprocess behavior suffices)
- Persistence that must outlive the browser connection: variable state lasts only for the duration of the websocket connection
- Situations requiring shared state across different browser windows or tabs of unrelated users; each session's variables are isolated from other sessions

## Inputs

- Toyz source tree (toyz.web.app, toyz.web.tasks) from git clone git://github.com/fred3m/toyz.git
- WebSocket send_task messages (JSON with module, task, parameters keys)
- Task id fields: user_id, session_id, request_id
- User permission configuration for executable modules

## Outputs

- Session registry mapping websocket session_id to a dedicated Python subprocess
- Per-session persistent variable state living in the session subprocess
- Server response returned for every completed job

## How to apply

Inspect the Toyz web application in github.com/fred3m/toyz (Tornado-based server, toyz.web.app and toyz.web.tasks modules), noting that the default job API runs each received task in a new subprocess that is destroyed on completion. Build a session registry keyed by the task id fields (user_id, session_id, request_id), where session_id is the unique id generated for each websocket connection. Modify the job dispatch path so the first send_task message (JSON with module, task, parameters keys) from a websocket spawns a dedicated Python subprocess for that session instead of a per-job subprocess, then route subsequent tasks from the same connection to that subprocess. Permit only modules the user has permission to execute and only modules ending in tasks.py, so variables defined in earlier tasks remain in memory for later tasks. Keep the subprocess state isolated from the Tornado web application layer and destroy the subprocess when the websocket disconnects.

## Related tools

- **Toyz** (Web application framework whose job API and websocket layer are modified to spawn session-scoped Python subprocesses) — https://github.com/fred3m/toyz
- **Python** (Language of the isolated workspace subprocess in which the user's variables are stored)
- **Tornado** (Web application layer from which the session subprocess state must remain isolated)

## Examples

```
git clone git://github.com/fred3m/toyz.git; cd toyz; python setup.py install
```

## Evaluation signals

- A test client sends sequential tasks over one websocket and confirms a variable created in the first task is retrievable in the second
- A second websocket session cannot read the first session's variables
- The session subprocess is cleaned up (destroyed) when the websocket disconnects
- The server returns a response for every completed job
- Only modules the user is granted permission to execute, and only modules ending in tasks.py, are run in the session subprocess

## Limitations

- The default Toyz job API runs each received task in a new subprocess destroyed on completion, so state persistence requires modifying the dispatch path
- Subprocess state is only persisted for the duration of the websocket connection and is lost on disconnect
- No changelog found for the repository, so version drift in the job API may need to be verified against the source

## Evidence

- [other] Each workspace runs as a separate Python process, isolated from the web application layer, in which the state of a user's variables is stored for the duration of the connection.: [evidence span withheld — anchor status: not_found]
