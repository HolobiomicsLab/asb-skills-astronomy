---
name: http-load-benchmarking-of-tile-servers
description: 'Use when sizing or validating a web tile server for scientific imagery:
  you need requests-per-second and per-request latency scaling with concurrency, or
  a user-capacity figure (e.g. ~100 users on a 12-core server).'
when_to_use_negative:
- The imagery is already stored as static 8-bit compressed images, so the benchmark
  would not exercise server-side dynamic processing of full bit-depth data
- The workload is interactive client-side rendering or FITS decoding in the browser
  rather than server tile-serving throughput
- The system under test is not an HTTP tile server (e.g. desktop viewers such as ds9,
  Gaia, or Aladin)
tools:
- name: visiomatic
  role: 'Tile server under test: visiomatic.server.tiled module serving tiles from
    the tiled pyramid TIFF via its FastCGI backend'
  repo: github.com/astromatic/visiomatic
- name: iipsrv (IIPImage / iipsrv-astro)
  role: Alternative embeddable FastCGI image server supporting 32-bit floating point
    tiles, deployable behind Apache/Lighttpd/NginX for the same benchmark
  repo: https://github.com/cmarmo/iipsrv-astro
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:6ae7d99128941203abbe62fc9772d86aadc239ea012f3e92c0df12bfbc56d40a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - iipsrv (IIPImage / iipsrv-astro)
  - visiomatic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# HTTP load benchmarking of tile servers

## Summary

Benchmark HTTP image-tile serving throughput and latency as a function of concurrent request count, using a repeated single-tile load client (modified Apache Bench) against a FastCGI tile server, to determine how many simultaneous users a single server can sustain while serving full-precision 32-bit astronomy imagery.

## When to use

Use when sizing or validating a web tile server for scientific imagery: you need requests-per-second and per-request latency scaling with concurrency, or a user-capacity figure (e.g. ~100 users on a 12-core server).

## When NOT to use

- The imagery is already stored as static 8-bit compressed images, so the benchmark would not exercise server-side dynamic processing of full bit-depth data
- The workload is interactive client-side rendering or FITS decoding in the browser rather than server tile-serving throughput
- The system under test is not an HTTP tile server (e.g. desktop viewers such as ds9, Gaia, or Aladin)

## Inputs

- Tiled pyramid TIFF of full-precision (32-bit floating-point) scientific imagery served by a FastCGI tile server
- Benchmark client (modified Apache Bench / ab) configured for repeated single-tile HTTP requests
- 12-core test web server with FastCGI backend deployment

## Outputs

- tile_benchmark_results.csv: serving rate (requests/s) and mean latency (ms) versus number of concurrent requests
- tile_serving_benchmark.png: plotted serving-rate and latency curves versus concurrency (Figure 8 reproduction)
- Capacity figure: maximum sustainable concurrent tile requests (~700) and equivalent user count (~100)

## How to apply

Deploy the tile server (e.g. the visiomatic tiled module or the iipsrv FastCGI server) behind an fcgi-enabled web server on the test machine, serving tiles extracted from a tiled pyramid TIFF. Set up a benchmark client — the paper used a modified Apache Bench (ab) that issues repeated single-tile HTTP requests for 256x256 tiles. Run the load test at progressively increasing concurrency levels up to ~700 concurrent requests (~100 users), recording requests-per-second and mean per-request latency at each level, and note the single-threaded FastCGI per-tile time (5–10 ms) as the latency baseline. Tabulate serving rate and latency versus concurrency, then plot both curves: the expected signature is a flat serving-rate curve with latency growing linearly once the server saturates.

## Related tools

- **visiomatic** (Tile server under test: visiomatic.server.tiled module serving tiles from the tiled pyramid TIFF via its FastCGI backend) — github.com/astromatic/visiomatic
- **iipsrv (IIPImage / iipsrv-astro)** (Alternative embeddable FastCGI image server supporting 32-bit floating point tiles, deployable behind Apache/Lighttpd/NginX for the same benchmark) — https://github.com/cmarmo/iipsrv-astro

## Examples

```
ab -n 100000 -c 700 "http://server/cgi-bin/visiomatic?FIF=image_pyr.tif&JTL=0,0,0"
```

## Evaluation signals

- The serving-rate (requests/s) curve remains approximately flat across increasing concurrency while mean latency grows linearly (Figure 8 signature)
- The server sustains ~700 concurrent 256x256 tile requests, corresponding to ~100 users browsing large single-channel floating-point images
- Per-tile request time falls in the 5–10 ms range for the single-threaded FastCGI baseline
- tile_benchmark_results.csv contains rows for each tested concurrency level with requests/s and mean latency columns populated
- The plotted latency-versus-concurrency relationship is consistent with the claim that a single server comfortably handles more than a hundred simultaneous users accessing full precision 32-bit astronomy data

## Limitations

- Results are specific to a single 12-core web server and uncompressed single-channel floating-point imagery; capacity will differ for other hardware, tile sizes, or compression settings
- The 'user count' (~100) assumes frantically browsing users, so it is an upper bound relative to typical idle-heavy viewing behavior
- Caching behavior (e.g. iipsrv's MAX_IMAGE_CACHE_SIZE compressed-tile RAM cache) can inflate throughput if the benchmark re-requests the same tiles; the paper used a modified ab client to control request patterns

## Evidence
