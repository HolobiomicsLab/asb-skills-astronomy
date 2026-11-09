---
name: client-server-task-dispatching
description: 'Reference for implementing or debugging websocket-based task dispatch
  in the Toyz web application: routing send_task messages to per-session subprocesses,
  persisting variables across sequential tasks, and isolating session state from the
  web layer.'
when_to_use_negative:
- The task only needs one-shot, stateless execution where variables need not persist
  between calls — the existing per-job subprocess path suffices
- The server is not running the Toyz/Tornado web application stack; this dispatch
  mechanism is specific to toyz.web.app and toyz.web.tasks
- The client cannot maintain a websocket connection, since per-session state depends
  on the session_id generated for each websocket connection
tools:
- name: Toyz
  role: Web application framework providing the websocket job API, session registry,
    and subprocess dispatch being modified
  repo: https://github.com/fred3m/toyz
- name: Python
  role: Language in which the server application and the per-session task subprocesses
    are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.10.001
    title: 'Toyz: A framework for scientific analysis of large datasets and astronomical
      images'
  source_content_hash: sha256:9f1a37f3dd211383e10a2387eb11f04404d76d53a5b7e7a8dcd787727378b2a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fred3m/toyz
  tools:
  - Python
  - Toyz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# client-server-task-dispatching

## Summary

Implements and exercises Toyz's mechanism for dispatching Python tasks from browser clients to a server process: a Tornado web application receives send_task JSON messages over websockets and routes them to isolated Python subprocesses, one per session, so per-session variable state persists for the duration of the connection.

## When to use

Reference for implementing or debugging websocket-based task dispatch in the Toyz web application: routing send_task messages to per-session subprocesses, persisting variables across sequential tasks, and isolating session state from the web layer.

## When NOT to use

- The task only needs one-shot, stateless execution where variables need not persist between calls — the existing per-job subprocess path suffices
- The server is not running the Toyz/Tornado web application stack; this dispatch mechanism is specific to toyz.web.app and toyz.web.tasks
- The client cannot maintain a websocket connection, since per-session state depends on the session_id generated for each websocket connection

## Inputs

- send_task JSON messages over a websocket connection, with module, task, and parameters keys
- user permission configuration granting module execution rights
- Python modules ending in tasks.py registered with the Toyz server
- websocket session_id, user_id, and request_id identifiers

## Outputs

- A per-session Python subprocess holding in-memory variable state for the duration of the connection
- A response returned by the server for every completed job
- Session registry mapping (user_id, session_id, request_id) to running subprocesses

## How to apply

Install Toyz from source in the github.com/fred3m/toyz repository (python setup.py install, then pip install -e .[all]) and inspect the Tornado-based server (toyz.web.app and toyz.web.tasks), where the job API currently runs each received task in a new subprocess destroyed on completion. Replace that per-job model with a session registry keyed by (user_id, session_id, request_id), where session_id is the unique id generated for each websocket connection. Modify the dispatch path so the first send_task message from a websocket — JSON with module, task, parameters keys — spawns a dedicated Python subprocess for that session, and route subsequent tasks from the same connection to that subprocess. Enforce permissions: only modules the user is granted permission to execute, and only modules ending in tasks.py. Keep the subprocess state isolated from the Tornado web application layer and destroy the subprocess when the websocket disconnects, ensuring the server returns a response for every completed job.

## Related tools

- **Toyz** (Web application framework providing the websocket job API, session registry, and subprocess dispatch being modified) — https://github.com/fred3m/toyz
- **Python** (Language in which the server application and the per-session task subprocesses are implemented)

## Examples

```
git clone git://github.com/fred3m/toyz.git; cd toyz; python setup.py install; toyz --port=8888
```

## Evaluation signals

- A test client sends sequential tasks over one websocket and a variable created in the first task is retrievable in the second
- A second websocket session cannot read the first session's variables (session isolation holds)
- The server returns a response for every completed job
- A subprocess is destroyed when its websocket disconnects (no orphan processes)
- Dispatch rejects modules the user lacks permission for, and modules not ending in tasks.py

## Limitations

- The subprocess is isolated from the Tornado web application layer, so the web layer cannot directly touch session variables
- Session state exists only for the duration of the websocket connection and is lost on disconnect
- Only permitted modules ending in tasks.py may be executed by a session

## Evidence

- [abstract] Users connect to the Toyz web application via a web browser, making it an convenient tool for students to visualize and interact with astronomical data without having to install any software on their: "Users connect to the Toyz web application via a web browser, making it an convenient tool for students to visualize and interact with astronomical data"
- [readme] README: Toyz web app runs scripts on a remote server: "Toyz is a web application framework designed to meet the needs of "Big Data""
- [methods] Methods: clone the Toyz source from github: "git clone git://github.com/fred3m/toyz.git"
- [methods] Methods: launch the toyz application on a specified port: "toyz --port=<remote_port>"
