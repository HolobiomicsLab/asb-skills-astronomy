---
name: cmb-timestream-destriping
description: Use when a polarimeter timestream (HDF5 with TIME, PHI, THETA, PSI, FLAG,
  TEMP, Q, U columns) contains 1/f correlated noise below the scan rate that must
  be removed before map-making, e.g. for B-Machine 37.5 GHz or LATTE-like data.
when_to_use_negative:
- When systematic effects such as thermally induced gain variations (e.g. sections
  near sunrise/sunset) or spin-synchronous effects dominate — the article warns destriping
  can be ineffective or even harmful and these must be handled before the map-making
  stage
- When the correlated noise 1/f knee frequency is higher than the scan rate, or the
  scanning strategy lacks frequent crossing points, destriping cannot efficiently
  remove the noise
- When the goal is to remove signal components rather than noise — destriping is a
  high-pass filter that only affects the noise in the timestream
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3315
tools:
- name: dst
  role: Python parallel destriper that solves the destriping equation for baselines
    (GMRES via Trilinos Belos) and produces binned HEALPix maps
  repo: http://github.com/zonca/dst
- name: Python
  role: Implementation language; run dst serially or under MPI
  repo: http://github.com/zonca/dst
- name: HDF5
  role: Binary machine-independent hierarchical data storage for input timestreams
    read via dst_io.read_data
- name: MPI
  role: Parallel reading/writing and distribution of data, baselines and maps among
    processors
- name: HEALPix
  role: Pixelization scheme for the binned map distribution of the destriped data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:f21ae23372b73d40d9da95ea23fdf3d3c07e2658d8dc2b3e551ba7759eb85e64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HEALPix
  - MPI
  - Python
  - dst
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# cmb-timestream-destriping

## Summary

Maximum Likelihood destriping of CMB polarimeter timestreams: solve the destriping equation for baseline offsets (60 s baselines, 2000 samples at 33.4 Hz) with GMRES via Trilinos Belos using the dst Python parallel destriper, removing low-frequency correlated noise while leaving sky signal intact.

## When to use

Use when a polarimeter timestream (HDF5 with TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns) contains 1/f correlated noise below the scan rate that must be removed before map-making, e.g. for B-Machine 37.5 GHz or LATTE-like data.

## When NOT to use

- When systematic effects such as thermally induced gain variations (e.g. sections near sunrise/sunset) or spin-synchronous effects dominate — the article warns destriping can be ineffective or even harmful and these must be handled before the map-making stage
- When the correlated noise 1/f knee frequency is higher than the scan rate, or the scanning strategy lacks frequent crossing points, destriping cannot efficiently remove the noise
- When the goal is to remove signal components rather than noise — destriping is a high-pass filter that only affects the noise in the timestream

## Inputs

- Single HDF5 timestream file with columns [('TIME','>f8'),('PHI','>f8'),('THETA','>f8'),('PSI','>f8'),('FLAG','>i2'),('TEMP','>f4'),('Q','>f4'),('U','>f4')]
- Test dataset ch6_fill_3d.h5 (3 days, from figshare) placed in the h5/ folder
- dst configuration file ch6_256.cfg specifying baseline length and solver settings

## Outputs

- Solved baseline offsets array (per 60 s baseline segment) for the timestream
- Destriped HEALPix sky map (binned Q/U map distribution via Trilinos)

## How to apply

Load a single HDF5 file whose columns are TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U (fill flagged/missing samples with white noise so baselines contain a fixed number of samples and FFTs are possible). Configure dst via the ch6_256.cfg file, setting the baseline length slightly shorter than the spin period — about 60 seconds, i.e. 2000 samples at 33.4 Hz for B-Machine — and choose QU-only destriping, appropriate for polarimeters. Build the cython extensions with 'python setup.py build_ext -i', then run serially ('python dst.py ch6_256.cfg') or in parallel ('mpirun -np 3 python dst.py ch6_256.cfg'); dst solves the destriping equation for the baselines with GMRES through Trilinos Belos, bins data locally on each process, and reduces to a global HEALPix binned map via Trilinos. Rationale: destriping acts as a high-pass filter that only affects the noise in the timestream, requiring a scanning strategy with frequent crossing points; it works when the 1/f knee frequency is lower than the scan rate.

## Related tools

- **dst** (Python parallel destriper that solves the destriping equation for baselines (GMRES via Trilinos Belos) and produces binned HEALPix maps) — http://github.com/zonca/dst
- **Python** (Implementation language; run dst serially or under MPI) — http://github.com/zonca/dst
- **HDF5** (Binary machine-independent hierarchical data storage for input timestreams read via dst_io.read_data)
- **MPI** (Parallel reading/writing and distribution of data, baselines and maps among processors)
- **HEALPix** (Pixelization scheme for the binned map distribution of the destriped data)

## Examples

```
python setup.py build_ext -i && python dst.py ch6_256.cfg  # or parallel: mpirun -np 3 python dst.py ch6_256.cfg
```

## Evaluation signals

- For a signal-only input timestream (no correlated or white noise added), the solved baseline array must be null to machine precision: max |baseline| < ~1e-15 K (reported ~1e-16 K)
- The input must be a single HDF5 file with exactly the columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U in the specified dtypes, or dst_io.read_data must be modified
- Baseline length must be a fixed number of samples (e.g. 2000 samples at 33.4 Hz ≈ 60 s); flagged/missing samples must be filled with white noise before destriping
- For the 3-day test dataset, the run with 'python dst.py ch6_256.cfg' should complete with the GMRES (Trilinos Belos) solver converging and producing a HEALPix binned map
- Destriping impact should be evaluated in map, frequency and angular power spectrum domains (partial-sky anafast spectra corrected by sky fraction)

## Limitations

- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility
- Systematic effects like gain variations or spin-synchronous effects must be handled separately before map-making; destriping can be ineffective or even harmful on them
- Flagged samples must be removed and gaps filled with white noise; other input data formats require modifying dst_io.read_data

## Evidence

- [intro] Destriping can be thought of as a high-pass filter that only affects the noise in the timestream: [evidence span withheld — anchor status: not_found]
- [intro] destriping can efficiently remove 1 / f noise with a knee frequency lower than the scan rate by removing a sequence of offsets: [evidence span withheld — anchor status: not_found]
- [readme] try running the destriping serially:

        python dst.py ch6_256.cfg: "try running the destriping serially:

        python dst.py ch6_256.cfg"
