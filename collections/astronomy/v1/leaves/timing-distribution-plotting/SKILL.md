---
name: timing-distribution-plotting
description: Use when benchmarking a tile-server pipeline (e.g. iipsrv serving 256x256
  tiles from 32-bit floating-point tiled TIFFs) and you need to attribute latency
  to pipeline stages under different cache and storage configurations.
when_to_use_negative:
- When only end-to-end request latency matters and per-stage attribution is unnecessary
  — a simple load test suffices
- When the image is already an 8-bit compressed format, so normalization/gamma/quantization
  stages do not apply
- When measuring client-side or network latency rather than the server-side processing
  pipeline
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: iipsrv-astro
  role: tile server whose pipeline stages (tile access, normalization, gamma correction,
    quantization, JPEG encoding) are instrumented with timers; its in-RAM cache (MAX_IMAGE_CACHE_SIZE)
    is toggled to define benchmark contexts
  repo: https://github.com/cmarmo/iipsrv-astro
- name: matplotlib
  role: plots the per-stage timing distributions into stage_timing_distributions.png
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:6ec768c19e7dd19af32671d66035fed6fec52ce81bf1ab25191fbbb7f495444f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - iipsrv-astro
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# timing-distribution-plotting

## Summary

Instrument each stage of a server-side image-tile processing pipeline (initialization, tile access, normalization, gamma correction, 8-bit quantization, JPEG encoding), collect repeated per-stage timings across cache/storage contexts, and plot the timing distributions to expose I/O- versus compute-dominated stages.

## When to use

Use when benchmarking a tile-server pipeline (e.g. iipsrv serving 256x256 tiles from 32-bit floating-point tiled TIFFs) and you need to attribute latency to pipeline stages under different cache and storage configurations.

## When NOT to use

- When only end-to-end request latency matters and per-stage attribution is unnecessary — a simple load test suffices
- When the image is already an 8-bit compressed format, so normalization/gamma/quantization stages do not apply
- When measuring client-side or network latency rather than the server-side processing pipeline

## Inputs

- 32-bit floating-point tiled TIFF test image with 256x256 tile size
- iipsrv-astro server instance with configurable MAX_IMAGE_CACHE_SIZE
- Repeated IIP JTL/Tile requests for random 256x256 tiles
- Per-context configuration: RAID/disk type, page cache flushed/unflushed, iipsrv cache on/off

## Outputs

- stage_timings.csv with per-stage timing samples aggregated per context
- stage_timing_distributions.png showing timing distributions per pipeline stage

## How to apply

Deploy the iipsrv-astro FastCGI server against a 32-bit floating-point tiled TIFF (256x256 tiles) and define benchmark contexts by varying RAID/disk type (spinning disk vs SSD), OS page cache state (flushed via /proc/sys/vm/drop_caches vs unflushed), and the iipsrv in-RAM tile cache (MAX_IMAGE_CACHE_SIZE on vs off). In each context, issue repeated IIP JTL/Tile requests for random 256x256 tiles while wrapping each pipeline stage — initialization, tile access, normalization, gamma correction, 8-bit quantization and JPEG encoding — with timers. Aggregate samples per context into a stage_timings.csv, then use matplotlib to plot per-stage timing distributions so relative ordering is visible. The rationale: when the page cache is flushed, tile access is dominated by disk I/O (≈25 ms on HDDs, <1 ms on SSDs), so distribution plots reveal which stages are I/O-bound versus CPU-bound (JPEG encoding, quantization).

## Related tools

- **iipsrv-astro** (tile server whose pipeline stages (tile access, normalization, gamma correction, quantization, JPEG encoding) are instrumented with timers; its in-RAM cache (MAX_IMAGE_CACHE_SIZE) is toggled to define benchmark contexts) — https://github.com/cmarmo/iipsrv-astro
- **matplotlib** (plots the per-stage timing distributions into stage_timing_distributions.png)

## Examples

```
echo 3 > /proc/sys/vm/drop_caches; curl 'http://localhost/fcgi-bin/iipsrv.fcgi?FIF=test.tif&JTL=0,0'; python plot_stage_timings.py --csv stage_timings.csv --out stage_timing_distributions.png
```

## Evaluation signals

- All six stages (initialization, tile access, normalization, gamma correction, 8-bit quantization, JPEG encoding) are measured in all four cache/storage contexts
- With page cache flushed, the tile-access distribution dominates and shows ≈25 ms on spinning hard disks
- Switching to SSD reduces the uncached tile-access distribution to less than 1 ms
- Relative stage ordering across contexts matches the benchmark behavior reported in the paper's performance figure
- stage_timings.csv contains repeated samples per stage per context, sufficient to form distributions rather than single points

## Limitations

- Timing results are highly sensitive to storage backend and OS page cache state; forgetting to flush via /proc/sys/vm/drop_caches makes uncached measurements indistinguishable from cached ones
- The iipsrv internal cache (default MAX_IMAGE_CACHE_SIZE = 10MB) must be disabled or sized carefully, or cached tiles will mask I/O costs
- Comparisons are qualitative — validation is against relative stage ordering in the paper's figure, not absolute timings, which depend on hardware

## Evidence

- [other] uncached tile access on spinning hard disks can reach ≈25 ms, while switching to SSDs reduces uncached file access latencies to less than 1 ms: [evidence span withheld — anchor status: not_found]
- [readme] High performance with inbuilt configurable cache: [evidence span withheld — anchor status: external_doc]
- [readme] MAX_IMAGE_CACHE_SIZE: Max image cache size to be held in RAM in MB. This is a cache of the compressed JPEG image tiles requested by the client.: [evidence span withheld — anchor status: external_doc]
