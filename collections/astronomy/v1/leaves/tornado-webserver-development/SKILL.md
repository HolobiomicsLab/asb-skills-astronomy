---
name: tornado-webserver-development
description: Use when a client must interact with data or run scripts on a remote
  server through a browser-based interface, and you need to implement or reconstruct
  the session/job-queue control loop of the Toyz Tornado server.
when_to_use_negative:
- The client is doing purely local analysis — Toyz exists because existing astronomy
  visualization tools must run on a local machine with data stored locally, which
  is what it avoids
- You need a batch/scientific pipeline rather than an interactive browser-driven web
  application framework
tools:
- name: Toyz
  role: Web application framework whose Tornado server-side session/job-queue control
    loop this skill reconstructs
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Implementation language of the Toyz server, websocket handlers, and task subprocesses
  repo: https://github.com/fred3m/toyz
- name: Tornado
  role: Web framework providing the websocket handler that spawns per-session processes
    and queues submitted jobs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:10da129b71fd6b752ddbf9f1ba205268355c15f95e661a5938211c7f83afb083
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
n_claims: 8
---

# tornado-webserver-development

## Summary

Reconstruct or build the server-side control loop of the Toyz web application, in which each Tornado websocket client connection spawns a dedicated session process that authenticates jobs and runs them in per-session queues.

## When to use

Use when a client must interact with data or run scripts on a remote server through a browser-based interface, and you need to implement or reconstruct the session/job-queue control loop of the Toyz Tornado server.

## When NOT to use

- The client is doing purely local analysis — Toyz exists because existing astronomy visualization tools must run on a local machine with data stored locally, which is what it avoids
- You need a batch/scientific pipeline rather than an interactive browser-driven web application framework

## Inputs

- Incoming websocket send_task messages (JSON dictionaries with module, task, and parameters keys)
- Secure authentication cookie used to identify the user
- Toyz source repository (toyz.web.app and toyz.web.tasks modules) as the reference implementation

## Outputs

- Per-session job queue keyed by session_id
- Subprocess per queued job executing the task function with toyz_settings, tid, and params
- JSON response over the websocket per completed job, containing at minimum a status key

## How to apply

Clone the Toyz source (git clone git://github.com/fred3m/toyz.git) and use the toyz.web.app and toyz.web.tasks modules as the reference for the server-side job API. Implement a Tornado websocket handler so that each new client connection spawns a dedicated session process identified by a unique session_id. Parse each incoming send_task message as a JSON dictionary with module, task, and parameters keys, enforcing that all executable module names end in tasks.py. Authenticate the user from the secure cookie, build the tid dictionary (user_id, session_id, request_id), and place the authenticated job in the per-session queue; execute each queued job in a new subprocess that receives toyz_settings, tid, and params, and destroy the subprocess when the job completes. Send a JSON response back over the websocket for every completed job, including at minimum a status key.

## Related tools

- **Toyz** (Web application framework whose Tornado server-side session/job-queue control loop this skill reconstructs) — https://github.com/fred3m/toyz
- **Python** (Implementation language of the Toyz server, websocket handlers, and task subprocesses) — https://github.com/fred3m/toyz
- **Tornado** (Web framework providing the websocket handler that spawns per-session processes and queues submitted jobs)

## Examples

```
git clone git://github.com/fred3m/toyz.git
```

## Evaluation signals

- The server starts, accepts one websocket connection, and spawns a session process with a unique session_id
- An authenticated send_task message is placed in the per-session queue and executed in a subprocess
- The executable module name ends in tasks.py (unauthorized module names are rejected)
- The client receives a JSON response containing a status key for every completed job
- The task subprocess is destroyed after the job has completed

## Limitations

- Toyz is a framework, not an application — server behavior must be customized for the end user's scientific needs, so the control loop is a template rather than a turnkey deployment
- FITS-image-driven usage (e.g. viewing > 2 Gb DECam images) implies the job loop must tolerate large data payloads on the remote server
- No changelog was available for the repository, so reconstructing exact behavioral changes over time is not possible from the README alone

## Evidence
