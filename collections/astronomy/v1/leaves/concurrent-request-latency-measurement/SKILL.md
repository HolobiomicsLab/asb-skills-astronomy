---
name: concurrent-request-latency-measurement
description: 'Use when estimating server capacity for tile-based image streaming:
  you need to measure requests-per-second and latency as concurrent 256×256 tile requests
  scale up on a multi-core server.'
when_to_use_negative:
- Input data is not full-precision (32-bit floating point) scientific imagery served
  as 256×256 tiles — capacity numbers will not transfer.
- The goal is client-side or network bandwidth measurement rather than server-side
  tile serving capacity.
- The server architecture is distributed or multi-server; this benchmark characterizes
  a single 12-core server only.
tools:
- name: visiomatic
  role: Tile server under test (visiomatic.server.tiled module via FastCGI) serving
    256×256 tiles from the tiled pyramid TIFF
  repo: github.com/astromatic/visiomatic
- name: iipsrv-astro
  role: Alternative FastCGI image streaming server reference supporting 32-bit floating
    point imagery
  repo: cmarmo/iipsrv-astro
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:9516c876739d3e62c5daa4642e13a7515cfc93a97b3ae318248e63c2dc0c3d1e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - iipsrv-astro
  - visiomatic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# concurrent-request-latency-measurement

## Summary

Benchmark how tile serving rate and per-request latency scale with concurrency on a single web server, using a modified Apache Bench client against a FastCGI tile server, to quantify how many concurrent users a server comfortably handles for full 32-bit astronomy imagery.

## When to use

Use when estimating server capacity for tile-based image streaming: you need to measure requests-per-second and latency as concurrent 256×256 tile requests scale up on a multi-core server.

## When NOT to use

- Input data is not full-precision (32-bit floating point) scientific imagery served as 256×256 tiles — capacity numbers will not transfer.
- The goal is client-side or network bandwidth measurement rather than server-side tile serving capacity.
- The server architecture is distributed or multi-server; this benchmark characterizes a single 12-core server only.

## Inputs

- tiled pyramid TIFF of a large single-channel floating-point astronomy image
- visiomatic tile server (FastCGI backend) deployed on a 12-core web server
- modified Apache Bench (ab) benchmark client configured for single-tile 256×256 HTTP requests

## Outputs

- tile_benchmark_results.csv with serving rate (requests/s) and mean latency (ms) versus number of concurrent requests
- tile_serving_benchmark.png reproducing Figure 8 curves (flat serving rate, linear latency growth)

## How to apply

Deploy the visiomatic tile server (visiomatic.server.tiled serving tiles from the tiled pyramid TIFF) via its FastCGI backend on the test server (e.g. 12 cores). Set up a modified Apache Bench (ab) client as described in Section 5 to issue repeated single-tile HTTP requests. Run the load test at progressively increasing concurrency levels up to ~700 concurrent requests (~100 users), recording requests-per-second and per-request latency at each level. Record the single-threaded FastCGI per-tile request time (5–10 ms) as the baseline reference. Tabulate serving rate and mean latency versus concurrency in tile_benchmark_results.csv and plot the Figure 8-style curves (flat serving rate, linear latency growth) as tile_serving_benchmark.png.

## Related tools

- **visiomatic** (Tile server under test (visiomatic.server.tiled module via FastCGI) serving 256×256 tiles from the tiled pyramid TIFF) — github.com/astromatic/visiomatic
- **iipsrv-astro** (Alternative FastCGI image streaming server reference supporting 32-bit floating point imagery) — cmarmo/iipsrv-astro

## Examples

```
ab -n 100000 -c 700 http://12-core-server/tile/256x256/tile.tif
```

## Evaluation signals

- The serving rate (requests/s) curve remains approximately flat across concurrency levels while latency grows linearly.
- The system sustains ~700 concurrent 256×256 tile requests, corresponding to about 100 users.
- The single-threaded FastCGI per-tile request baseline falls in the 5–10 ms range and is consistent with the observed scaling.
- tile_benchmark_results.csv contains both serving rate and per-request latency for each tested concurrency level.
- The plotted curves are consistent with the paper's claim that a single server comfortably handles more than a hundred simultaneous users accessing full precision 32-bit astronomy data.

## Limitations

- Results characterize a single 12-core web server loaded beyond the number of CPU cores; they do not extrapolate directly to other hardware or distributed deployments.
- The measurement targets large, uncompressed, single-channel floating-point images served as 256×256 tiles; other tile sizes or compressed formats will scale differently.
- The ~700 concurrent requests / ~100 users figure assumes users 'frantically browsing'; lighter usage patterns allow more users per server.

## Evidence

- [other] a single 12-core web server can handle ∼700 concurrent 256 × 256 tile requests, which corresponds to about 100 users frantically browsing large, uncompressed, single-channel floating-point images.: [evidence span withheld — anchor status: not_found]
- [other] Run the load test at progressively increasing concurrency levels up to ~700 concurrent requests (~100 users), recording requests-per-second and per-request latency at each level.: [evidence span withheld — anchor status: not_found]
- [other] Record the single-threaded FastCGI per-tile request time (5-10 ms) as the baseline reference for the latency scaling analysis.: [evidence span withheld — anchor status: not_found]
- [abstract] A single server can comfortably handle more than a hundred simultaneous users accessing full precision 32 bit astronomy data.: [evidence span withheld — anchor status: not_found]
- [readme] Fast lightweight embeddable FastCGI server module: [evidence span withheld — anchor status: external_doc]
