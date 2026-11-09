---
name: throughput-scaling-visualization
description: Use when you need to measure how an HTTP tile/image server's throughput
  and latency degrade as concurrent client requests exceed available CPU cores, e.g.
  to size server capacity for full-precision astronomy imagery users.
when_to_use_negative:
- When the client-side rendering or network transfer is the bottleneck and you need
  end-user experience metrics rather than server-side tile serving throughput
- When the input imagery is small 8-bit static images served from cache, where the
  server is not CPU-saturated and concurrency scaling conclusions for full-precision
  floating-point tiles do not apply
- When benchmarking multi-server or load-balanced deployments — this skill characterizes
  a single 12-core server only
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: visiomatic
  role: 'Tile server under test: visiomatic.server.tiled serves 256x256 tiles from
    the tiled pyramid TIFF via a FastCGI backend during the load test'
  repo: github.com/astromatic/visiomatic
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:3e8a1fd82e6cb5077294f2ae857af84f75582478e84488d6c4a291578bbbf21e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - visiomatic
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# throughput-scaling-visualization

## Summary

Benchmark and visualize how tile serving rate and per-request latency scale with concurrent tile requests on a single multi-core web server, to establish the practical user capacity of a web visualization system.

## When to use

Use when you need to measure how an HTTP tile/image server's throughput and latency degrade as concurrent client requests exceed available CPU cores, e.g. to size server capacity for full-precision astronomy imagery users.

## When NOT to use

- When the client-side rendering or network transfer is the bottleneck and you need end-user experience metrics rather than server-side tile serving throughput
- When the input imagery is small 8-bit static images served from cache, where the server is not CPU-saturated and concurrency scaling conclusions for full-precision floating-point tiles do not apply
- When benchmarking multi-server or load-balanced deployments — this skill characterizes a single 12-core server only

## Inputs

- Tiled pyramid TIFF image hosted by the visiomatic tile server (full precision 32-bit floating point astronomy imagery)
- Modified Apache Bench (ab) benchmark client configured for repeated single-tile HTTP GET requests
- 12-core (or similar) web server with FastCGI backend deployment of visiomatic.server.tiled

## Outputs

- tile_benchmark_results.csv: serving rate (requests/s) and mean latency (ms) versus number of concurrent requests
- tile_serving_benchmark.png: plot of serving rate and latency versus concurrency reproducing the flat-rate / linear-latency curves (Figure 8 analogue)

## How to apply

Deploy the visiomatic tile server (visiomatic.server.tiled serving tiles from a tiled pyramid TIFF) behind its FastCGI backend on a known-core server (reference setup: 12 cores). Drive load with a modified Apache Bench (ab) client issuing repeated single-tile HTTP requests for 256x256 tiles, sweeping concurrency upward while recording requests-per-second and mean per-request latency at each level. Record the single-threaded FastCGI per-tile request time (5-10 ms) as the latency baseline. Tabulate the results (tile_benchmark_results.csv) and plot serving rate and latency versus concurrency to reproduce the expected regime: an approximately flat serving-rate curve with linearly growing latency, sustaining about 700 concurrent tile requests (~100 frantic users browsing large uncompressed single-channel floating-point images).

## Related tools

- **visiomatic** (Tile server under test: visiomatic.server.tiled serves 256x256 tiles from the tiled pyramid TIFF via a FastCGI backend during the load test) — github.com/astromatic/visiomatic

## Examples

```
ab -n 10000 -c 700 http://server/fcgi-bin/iipsrv.fcgi?FIF=image.tif&OBJ=deep-zoom&tile=256,256
```

## Evaluation signals

- The serving rate (requests/s) curve remains approximately flat across the concurrency sweep instead of rising or collapsing
- Per-request latency grows approximately linearly with the number of concurrent requests
- The server sustains roughly 700 concurrent 256x256 tile requests (about 100 simultaneous users) on a 12-core server
- Recorded single-threaded FastCGI per-tile request time falls in the 5-10 ms baseline range
- CSV rows cover a monotonic sweep of concurrency levels up to ~700 requests with both rate and latency columns populated

## Limitations

- Characterization is for a single 12-core web server loaded beyond its CPU core count; extrapolation to other hardware or clustered deployments is untested
- The ~700 concurrent request / ~100 user figure applies to frantically browsing large, uncompressed, single-channel floating-point images; lighter usage patterns yield different capacity
- System relies on server-side compression/encoding because browsers display gamma-encoded 8-bit images; benchmarks on already-browser-friendly static data bypass the interesting code path

## Evidence

- [results] a single 12-core web server can handle ∼700 concurrent 256 × 256 tile requests: [evidence span withheld — anchor status: not_found]
- [intro] browser engines cannot display floating point data: [evidence span withheld — anchor status: not_found]
- [readme] Fast lightweight embeddable FastCGI server module: [evidence span withheld — anchor status: external_doc]
