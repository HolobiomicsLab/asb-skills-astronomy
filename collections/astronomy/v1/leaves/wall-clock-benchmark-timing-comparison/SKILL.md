---
name: wall-clock-benchmark-timing-comparison
description: Use when you need to benchmark the runtime of vis-r radial-profile fitting
  against an image-based parametric modelling baseline on the same visibility file,
  e.g. to verify the reported 10x-100x speedup.
when_to_use_negative:
- Input visibilities have radially dependent vertical structure requirements beyond
  a fixed vertical scale height — the method assumes a single disk plane with fixed
  scale height
- The goal is posterior accuracy rather than runtime benchmarking — hard --sz u,v
  averaging gives quick but less robust parameter estimates
edam_operation: http://edamontology.org/operation_3861
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: vis-r
  role: Radial-profile visibility fitting implementation whose runtime is benchmarked;
    run with hard u,v averaging via --sz set near the disk size
  repo: https://github.com/drgmk/vis-r
- name: Python
  role: Times both fits with the time module, loads .npy visibilities, and writes
    timing_comparison.csv
- name: ALMA
  role: Source of the high-resolution self-calibrated visibilities being modelled
    in the timing comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:f19e18343cd68a1eea6c2636b2730b0ad5a9c40fa37cd37658970970d131c0b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - ALMA
  - Python
  - vis-r
  edam_operation: http://edamontology.org/operation_3861
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# wall-clock-benchmark-timing-comparison

## Summary

Quantifies the computational advantage of vis-r's radial-profile visibility fitting over conventional image-based parametric modelling of ALMA visibilities by timing both approaches on the same input and recording a wall-clock speedup ratio.

## When to use

Use when you need to benchmark the runtime of vis-r radial-profile fitting against an image-based parametric modelling baseline on the same visibility file, e.g. to verify the reported 10x-100x speedup.

## When NOT to use

- Input visibilities have radially dependent vertical structure requirements beyond a fixed vertical scale height — the method assumes a single disk plane with fixed scale height
- The goal is posterior accuracy rather than runtime benchmarking — hard --sz u,v averaging gives quick but less robust parameter estimates

## Inputs

- Self-calibrated ALMA visibilities exported to a numpy file, e.g. hr4796.selfcal.npy
- Image-based parametric disk model baseline (image generation + Fourier transform to visibility space)
- vis-r radial-profile fitting executable

## Outputs

- timing_comparison.csv containing image-based runtime, vis-r runtime, and the speedup ratio
- Speedup ratio expected to fall between 10x and 100x

## How to apply

Load the self-calibrated visibilities (e.g. hr4796.selfcal.npy) with Python/numpy. Implement the image-based baseline by generating a parametric disk image, Fourier transforming it to visibility space, and timing the full fit wall-clock with Python's time module. Run vis-r on the same visibility file using hard u,v averaging via the --sz parameter set near the disk size, and time that fit's wall-clock. Divide the image-based baseline time by the vis-r time to obtain the speedup ratio, then write both runtimes and the ratio to a CSV (e.g. timing_comparison.csv). Validate that the ratio falls in the 10x-100x range reported for the method, and that the vis-r run completes in seconds-to-minutes on a laptop (the package's minimal example runs in under 10 s on an M2 Mac).

## Related tools

- **vis-r** (Radial-profile visibility fitting implementation whose runtime is benchmarked; run with hard u,v averaging via --sz set near the disk size) — https://github.com/drgmk/vis-r
- **Python** (Times both fits with the time module, loads .npy visibilities, and writes timing_comparison.csv)
- **ALMA** (Source of the high-resolution self-calibrated visibilities being modelled in the timing comparison)

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01 --sz <disk-size>
```

## Evaluation signals

- Speedup ratio (image-based time / vis-r time) falls between 10x and 100x
- vis-r run completes in seconds-to-minutes on a laptop
- Both runtimes and the ratio are recorded in timing_comparison.csv
- Same visibility input file (e.g. hr4796.selfcal.npy) is used for both baseline and vis-r fits

## Limitations

- Speedup figures are wall-clock times on specific hardware; the package's minimal example timing (<10 s) was measured on an M2 Mac, so absolute times vary by machine
- Hard u,v averaging (--sz set near or below the disk size) trades posterior robustness for speed; larger --sz runs are slower but give more robust posterior estimates
- The radial-profile method is one to two orders of magnitude faster but assumes a single disk plane and fixed vertical scale height

## Evidence

- [abstract] Reported 10x-100x speedup claim: "these ingredients yield a method that is one to two orders of magnitude faster than image based methods and can be run on a laptop"
- [readme] Hard u,v averaging instruction: "initially use rather hard u,v averaging with the `--sz` parameter, setting it to something similar to the disk size"
- [results] Minimal example runtime: "minimal example script, this runs in less than 10s on an M2 Mac"
- [readme] Export visibilities to numpy file: "First, export your visibilities from the ms file to a text or numpy file."
