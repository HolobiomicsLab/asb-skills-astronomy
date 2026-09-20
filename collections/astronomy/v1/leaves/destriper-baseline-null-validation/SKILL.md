---
name: destriper-baseline-null-validation
description: Use when a signal-only (noise-free) simulated polarimeter timestream
  is available and you need to verify the destriper adds no spurious offsets — i.e.,
  baselines should solve to ~1e-16 K.
when_to_use_negative:
- The timestream contains added correlated or white noise — baselines are then expected
  to be non-null, since the destriper's baseline term represents the correlated-noise
  timeline
- Systematic effects such as gain variations or spin-synchronous signals are present
  — the paper states destriping can be ineffective or even harmful for these and they
  must be handled separately before map-making
- Data sections near sunrise/sunset are contaminated by thermally induced gain variations
  and have not been masked out
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: dst
  role: Python parallel destriper run serially ('python dst.py ch6_256.cfg') to solve
    the destriping equation for baselines via GMRES (Trilinos Belos)
  repo: http://github.com/zonca/dst
- name: HDF5
  role: Storage format for the input timestream (TIME, PHI, THETA, PSI, FLAG, TEMP,
    Q, U columns) and for the output baselines.h5
- name: HEALPix
  role: Binned map distribution used by the destriper during the solve
- name: Python
  role: Runs the dst pipeline serially and extracts/summarizes the solved baseline
    array
- name: MPI
  role: Optional parallel execution of the destriper ('mpirun -np 3 python dst.py
    ch6_256.cfg')
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:d9a53247d51f2e5fa4295f8d6e5c16dd649c4d166c3c1c237d205ece3fef87e9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HEALPix
  - MPI
  - Python
  - dst
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# destriper-baseline-null-validation

## Summary

A null-baseline validation for the dst destriper: run the solver on a signal-only timestream (no correlated or white noise added) and confirm the solved baseline offsets are null to machine precision (~1e-16 K), since in the destriper's model the baselines represent only the correlated-noise term.

## When to use

Use when a signal-only (noise-free) simulated polarimeter timestream is available and you need to verify the destriper adds no spurious offsets — i.e., baselines should solve to ~1e-16 K.

## When NOT to use

- The timestream contains added correlated or white noise — baselines are then expected to be non-null, since the destriper's baseline term represents the correlated-noise timeline
- Systematic effects such as gain variations or spin-synchronous signals are present — the paper states destriping can be ineffective or even harmful for these and they must be handled separately before map-making
- Data sections near sunrise/sunset are contaminated by thermally induced gain variations and have not been masked out

## Inputs

- Signal-only simulated timestream as a single HDF5 file with columns dtype=[('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')]
- dst configuration file (ch6_256.cfg) with baseline length set to 2000 samples (~60 s at 33.4 Hz)

## Outputs

- Solved baseline array written to baselines.h5 (or .csv)
- Summary statistic: maximum absolute solved baseline amplitude (expected < 1e-15 K)

## How to apply

Load the signal-only timestream from HDF5 (columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) and configure the dst destriper via ch6_256.cfg, setting the baseline length to 60 seconds (2000 samples at 33.4 Hz), slightly shorter than the 70-second B-Machine spin period. Run the destriper serially with 'python dst.py ch6_256.cfg' so GMRES (via Trilinos Belos) solves the destriping equation and HEALPix distributes the binned map. The rationale: in the destriper's model the noise is a separate additive term (white plus correlated noise), distinct from the scanned sky signal, so a timestream containing only signal gives the baseline solver nothing to represent. Extract the solved baseline array, compute its maximum absolute amplitude, and write the array plus summary to baselines.h5 (or .csv); the run is valid only if the max absolute baseline is below 1e-15 K.

## Related tools

- **dst** (Python parallel destriper run serially ('python dst.py ch6_256.cfg') to solve the destriping equation for baselines via GMRES (Trilinos Belos)) — http://github.com/zonca/dst
- **HDF5** (Storage format for the input timestream (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns) and for the output baselines.h5)
- **HEALPix** (Binned map distribution used by the destriper during the solve)
- **Python** (Runs the dst pipeline serially and extracts/summarizes the solved baseline array)
- **MPI** (Optional parallel execution of the destriper ('mpirun -np 3 python dst.py ch6_256.cfg'))

## Examples

```
python dst.py ch6_256.cfg
```

## Evaluation signals

- Maximum absolute solved baseline is below 1e-15 K, consistent with a null solution at machine precision (~1e-16 K)
- The destriper completes the GMRES solve via Trilinos Belos without iteration failures on the signal-only input
- baselines.h5 (or .csv) contains a baseline array whose length matches the configured 2000-sample (60 s) offset segmentation
- Input HDF5 file has the exact dst schema (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) so dst_io.read_data loads it correctly
- The solved map equals the HEALPix-binned input signal (no destriping-induced offsets added) since destriping only affects the noise in the timestream

## Limitations

- A null baseline confirms only the correlated-noise representation is idle on signal-only input; it does not validate destriper performance on data with 1/f correlated noise or systematic effects
- Destriping is ineffective or even harmful for gain variations or spin-synchronous effects, which must be dealt with before the map-making stage
- Destriping requires a scanning strategy with frequent crossing points; a signal-only dataset from a poorly crossing scan would not exercise the solver meaningfully
- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility of the validation

## Evidence

- [intro] Destriping can be thought of as a high-pass filter that only affects the noise in the timestream: [evidence span withheld — anchor status: not_found]
