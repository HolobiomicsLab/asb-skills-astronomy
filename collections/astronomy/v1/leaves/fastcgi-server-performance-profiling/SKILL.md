---
name: fastcgi-server-performance-profiling
description: Use when benchmarking how a web tile/image server's throughput and latency
  scale with concurrent requests beyond the number of CPU cores, to substantiate per-server
  user-capacity claims.
when_to_use_negative:
- When the goal is client-side rendering or browser interaction performance rather
  than server-side tile-serving throughput
- When evaluating distributed or multi-server deployments — this method profiles a
  single 12-core server under concurrency beyond its core count
- When the data are pre-rendered 8-bit compressed images rather than full-precision
  floating-point tiles, since per-tile compute cost and conclusions differ
edam_operation: http://edamontology.org/operation_2238
tools:
- name: visiomatic
  role: 'Tile server under test: visiomatic.server.tiled serves 256x256 tiles from
    the tiled pyramid TIFF via its FastCGI backend'
  repo: github.com/astromatic/visiomatic
- name: iipsrv (IIPImage)
  role: Alternative FastCGI high-resolution image server module that can be profiled
    with the same load-testing approach; embeddable FastCGI server with inbuilt configurable
    cache
  repo: cmarmo/iipsrv-astro
- name: Apache Bench (ab, modified)
  role: Benchmark client issuing repeated single-tile HTTP requests at progressively
    increasing concurrency levels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:87cb6c1a456c49668789ed2801b42ca3e5b47914a015b73fb53634f33fc42e2d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Apache Bench (ab, modified)
  - iipsrv (IIPImage)
  - visiomatic
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fastcgi-server-performance-profiling

## Summary

A load-testing method that characterizes how FastCGI-based tile serving rate and per-request latency scale with concurrency on a single multi-core server, using a modified Apache Bench (ab) client against the visiomatic tile server. It grounds server-capacity claims (e.g. ~700 concurrent 256x256 tile requests, ~100 users) in measured requests-per-second and latency curves.

## When to use

Use when benchmarking how a web tile/image server's throughput and latency scale with concurrent requests beyond the number of CPU cores, to substantiate per-server user-capacity claims.

## When NOT to use

- When the goal is client-side rendering or browser interaction performance rather than server-side tile-serving throughput
- When evaluating distributed or multi-server deployments — this method profiles a single 12-core server under concurrency beyond its core count
- When the data are pre-rendered 8-bit compressed images rather than full-precision floating-point tiles, since per-tile compute cost and conclusions differ

## Inputs

- Tiled pyramid TIFF (single-channel, 32-bit floating-point astronomy image) served as 256x256 tiles
- FastCGI tile server endpoint (visiomatic.server.tiled or iipsrv.fcgi) behind Apache
- Modified Apache Bench (ab) load-generation client configured for repeated single-tile HTTP requests

## Outputs

- tile_benchmark_results.csv (serving rate in requests/s and mean latency in ms versus number of concurrent requests)
- tile_serving_benchmark.png (Figure-8-style curves: flat serving rate, linear latency growth versus concurrency)

## How to apply

Deploy the tile server (e.g. visiomatic.server.tiled serving tiles from the tiled pyramid TIFF, or iipsrv as a FastCGI module) behind a web server on the test host. Drive it with the modified Apache Bench (ab) client issuing repeated single-tile HTTP requests (256x256 tiles), sweeping concurrency levels progressively up to the saturation region (here ~700 concurrent requests, corresponding to ~100 users). Record the single-threaded FastCGI per-tile request time (5-10 ms in this system) as the latency baseline. Tabulate serving rate (requests/s) and mean latency (ms) per concurrency level, then plot both versus concurrency: the expected signature is an approximately flat serving-rate curve with linearly growing latency, since requests queue once concurrency exceeds available cores.

## Related tools

- **visiomatic** (Tile server under test: visiomatic.server.tiled serves 256x256 tiles from the tiled pyramid TIFF via its FastCGI backend) — github.com/astromatic/visiomatic
- **iipsrv (IIPImage)** (Alternative FastCGI high-resolution image server module that can be profiled with the same load-testing approach; embeddable FastCGI server with inbuilt configurable cache) — cmarmo/iipsrv-astro
- **Apache Bench (ab, modified)** (Benchmark client issuing repeated single-tile HTTP requests at progressively increasing concurrency levels)

## Examples

```
ab -c 700 -n 70000 "http://server/cgi-bin/visiomatic?cmd=tile&z=8&x=128&y=128"  # sweep -c up to ~700, log requests/s and mean ms per level
```

## Evaluation signals

- The serving-rate (requests/s) curve remains approximately flat as concurrency increases, matching the Figure 8 signature
- Mean per-request latency grows approximately linearly with concurrency once load exceeds the 12 CPU cores
- The server sustains ~700 concurrent 256x256 tile requests (~100 users) without throughput collapse
- Single-threaded FastCGI per-tile request time falls in the 5-10 ms baseline range
- tile_benchmark_results.csv contains rows for each concurrency level with both requests/s and latency columns populated

## Limitations

- Results are specific to a single 12-core server and a single-threaded FastCGI per-tile baseline of 5-10 ms; other hardware or server configurations will shift the saturation point
- The measured capacity (~700 concurrent requests, ~100 users) applies to uncompressed, single-channel 32-bit floating-point imagery; other data types change per-tile compute cost
- The benchmark uses a modified ab client issuing repeated single-tile requests, which does not model realistic mixed-client browsing patterns

## Evidence

- [abstract] Server capacity finding: "a single server can comfortably handle more than a hundred simultaneous users accessing full precision 32 bit astronomy data."
- [intro] Server-side encoding rationale: "server-side compression and encoding of the original data to a browser-friendly format remains necessary in order to achieve a satisfactory user experience"
- [readme] FastCGI server module: [evidence span withheld — anchor status: external_doc]
- [readme] High-performance caching server: [evidence span withheld — anchor status: external_doc]
