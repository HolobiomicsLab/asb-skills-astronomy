---
name: iipsrv-tile-server-deployment
description: Use when serving gigapixel-scale 32-bit floating point scientific TIFFs
  to browser clients via on-the-fly 256x256 JPEG tile generation, or benchmarking
  server-side tile-processing stages under different storage and cache configurations.
when_to_use_negative:
- Data are small, already 8-bit static images, or fit entirely in the browser — client-side
  FITS decoding (e.g., jsFITS) avoids server deployment entirely
- The target is a catalog or HiPS-only visualization (e.g., Mizar-supported WMS/HiPS/MOC
  workflows) rather than full-precision pixel tile streaming
- JPEG2000 licensing (Kakadu) or OpenJPEG performance trade-offs are unacceptable
  for the deployment environment
tools:
- name: iipsrv-astro
  role: FastCGI tile server that performs tile access, normalization, gamma correction,
    8-bit quantization and JPEG encoding of 32-bit floating point tiled TIFFs
  repo: https://github.com/cmarmo/iipsrv-astro
- name: visiomatic
  role: Client-side web application consuming iipsrv-served tiles for astronomical
    image visualization
  repo: https://github.com/astromatic/visiomatic
- name: GDAL / tiffcp
  role: Generation of the tiled TIFF test image with 256x256 tiles from 32-bit floating
    point science data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:d4be94f528cdd27a35278c73ab726c54ed7193d88a56d286d7a168478f644b8d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - GDAL / tiffcp
  - iipsrv-astro
  - visiomatic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# iipsrv-tile-server-deployment

## Summary

Build, configure and deploy the iipsrv-astro (IIPImage) FastCGI tile server to stream on-the-fly JPEG tiles from large 32-bit floating-point tiled TIFF astronomy imagery, with control over its internal cache, JPEG quality and filesystem access settings.

## When to use

Use when serving gigapixel-scale 32-bit floating point scientific TIFFs to browser clients via on-the-fly 256x256 JPEG tile generation, or benchmarking server-side tile-processing stages under different storage and cache configurations.

## When NOT to use

- Data are small, already 8-bit static images, or fit entirely in the browser — client-side FITS decoding (e.g., jsFITS) avoids server deployment entirely
- The target is a catalog or HiPS-only visualization (e.g., Mizar-supported WMS/HiPS/MOC workflows) rather than full-precision pixel tile streaming
- JPEG2000 licensing (Kakadu) or OpenJPEG performance trade-offs are unacceptable for the deployment environment

## Inputs

- 32-bit floating-point tiled TIFF astronomy image (e.g., 256x256 tile size, generated with GDAL or tiffcp)
- Web server with FastCGI support (Apache, Lighttpd, NginX, IIS)
- iipsrv-astro source tree (cmarmo/iipsrv-astro)

## Outputs

- Deployed iipsrv.fcgi FastCGI tile server streaming JPEG tiles via IIP/Zoomify/DeepZoom/IIIF protocols
- Per-request JPEG tiles (e.g., 256x256 JTL tile responses) suitable for HTML5 web clients
- Server-side per-stage timings (initialization, tile access, normalization, gamma correction, 8-bit quantization, JPEG encoding) when instrumented

## How to apply

Convert the science data into a tiled TIFF (e.g., 256x256 tile size, 32-bit floating point, written with GDAL or tiffcp). Build iipsrv-astro with the standard autoconf process (./autogen.sh for git versions, then ./configure && make), linking libtiff, zlib and IJG JPEG libraries, and install the resulting iipsrv.fcgi executable into the web server's fcgi directory (Apache, Lighttpd, NginX and IIS are supported). Configure startup variables relevant to the experiment: MAX_IMAGE_CACHE_SIZE to switch the in-RAM compressed-tile cache on or off (default 10 MB), FILESYSTEM_PREFIX to restrict and prefix image paths, and JPEG_QUALITY (default 75) for the encoding stage. Serve IIP tile requests (JTL/Tile commands) for random 256x256 tiles; when measuring uncached I/O behaviour, flush the OS page cache (e.g., via /proc/sys/vm/drop_caches) and toggle the iipsrv internal cache to isolate storage effects.

## Related tools

- **iipsrv-astro** (FastCGI tile server that performs tile access, normalization, gamma correction, 8-bit quantization and JPEG encoding of 32-bit floating point tiled TIFFs) — https://github.com/cmarmo/iipsrv-astro
- **visiomatic** (Client-side web application consuming iipsrv-served tiles for astronomical image visualization) — https://github.com/astromatic/visiomatic
- **GDAL / tiffcp** (Generation of the tiled TIFF test image with 256x256 tiles from 32-bit floating point science data)

## Examples

```
./autogen.sh && ./configure && make  # then copy src/iipsrv.fcgi into the web server's fcgi-bin and set VERBOSITY, MAX_IMAGE_CACHE_SIZE, FILESYSTEM_PREFIX, JPEG_QUALITY
```

## Evaluation signals

- All six pipeline stages (initialization, tile access, normalization, gamma correction, 8-bit quantization, JPEG encoding) produce timing samples in each of the four storage/cache contexts
- Relative stage ordering matches the paper's benchmark: tile access dominates when the page cache is flushed (≈25 ms on spinning disks, <1 ms on SSDs)
- Server responds correctly to JTL/Tile IIP commands with JPEG tiles of the requested 256x256 geometry
- MAX_IMAGE_CACHE_SIZE and FILESYSTEM_PREFIX settings are honoured (cache toggling changes repeat-request latency; path traversal such as ../ is filtered)
- Default configuration behaves as documented: JPEG_QUALITY 75, MAX_IMAGE_CACHE_SIZE 10 MB, MAX_CVT 5000

## Limitations

- JPEG2000 decoding via the open source OpenJPEG backend is experimental and much slower than Kakadu; Kakadu is not open source and requires a purchased license
- Uncached tile access on spinning hard disks with the page cache flushed can reach ≈25 ms per 256x256 tile, so storage choice strongly affects uncached latency
- Memcached support requires libmemcached; Kakadu/OpenJPEG require extra configure flags; fcgi-enabled web server is mandatory

## Evidence

- [readme] It is designed to be fast and bandwidth-efficient with low processor and memory requirements. The system can comfortably handle gigapixel size images: [evidence span withheld — anchor status: external_doc]
- [readme] MAX_IMAGE_CACHE_SIZE: Max image cache size to be held in RAM in MB. This is a cache of the compressed JPEG image tiles requested by the client. The default is 10MB.: [evidence span withheld — anchor status: external_doc]
- [readme] IIPImage is able to decode JPEG2000 images via the open source OpenJPEG library ... This is currently experimental and is much slower than Kakadu.: [evidence span withheld — anchor status: external_doc]
- [readme] 1, 8, 16 and 32 bit image support including 32 bit floating point support: [evidence span withheld — anchor status: external_doc]
