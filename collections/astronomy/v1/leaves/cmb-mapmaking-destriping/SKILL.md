---
name: cmb-mapmaking-destriping
description: Use when a CMB polarimeter timestream (e.g. HDF5 with TIME/PHI/THETA/PSI/FLAG/TEMP/Q/U
  columns) contains 1/f correlated noise with knee frequency below the scan rate and
  you need a destriped I/Q sky map with minimal signal impact.
when_to_use_negative:
- The timestream contains systematics destriping cannot handle — e.g. thermally induced
  gain variations near sunrise/sunset (mask those sections first) or spin-synchronous
  effects, which 'need to be dealt with before the map-making stage'.
- The 1/f noise knee frequency is higher than the scan rate, or the scanning strategy
  lacks frequent crossing points, so baseline-offset destriping cannot recover the
  correlated noise.
- The data are not from a polarimeter with Q/U modulation, since the method is specifically
  tailored to QU-only destriping for polarimeter data.
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0593
tools:
- name: dst
  role: Python parallel destriper that solves for 60-second baselines via GMRES and
    produces the destriped HEALPix map
  repo: http://github.com/zonca/dst
- name: HEALPix
  role: Sky pixelization scheme (Nside 512) used for binning, map output, and anafast
    partial-sky spectra
- name: HDF5
  role: Binary machine-independent hierarchical format used to store the input timestream
    (ch6_fill_3d.h5)
- name: Trilinos
  role: PyTrilinos/Belos provides the GMRES iterative solver and MPI communication
    for distributing baselines and maps
- name: MPI
  role: Enables parallel reading/writing and running the destriper as 'mpirun -np
    3 python dst.py ch6_256.cfg'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:57500c7c9de7c74075682f87b84a859c12d49162453106f56e027790c64666d3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HEALPix
  - MPI
  - Python
  - Trilinos
  - dst
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cmb-mapmaking-destriping

## Summary

Map-making skill for CMB polarimeter data that removes low-frequency correlated (1/f) noise by fitting and subtracting 60-second baselines via Maximum Likelihood destriping, implemented in the parallel Python destriper 'dst' with Trilinos/GMRES, validated by reduced chi2 of residual maps against a plain binning baseline.

## When to use

Use when a CMB polarimeter timestream (e.g. HDF5 with TIME/PHI/THETA/PSI/FLAG/TEMP/Q/U columns) contains 1/f correlated noise with knee frequency below the scan rate and you need a destriped I/Q sky map with minimal signal impact.

## When NOT to use

- The timestream contains systematics destriping cannot handle — e.g. thermally induced gain variations near sunrise/sunset (mask those sections first) or spin-synchronous effects, which 'need to be dealt with before the map-making stage'.
- The 1/f noise knee frequency is higher than the scan rate, or the scanning strategy lacks frequent crossing points, so baseline-offset destriping cannot recover the correlated noise.
- The data are not from a polarimeter with Q/U modulation, since the method is specifically tailored to QU-only destriping for polarimeter data.

## Inputs

- Single HDF5 file of polarimeter timestream data (e.g. ch6_fill_3d.h5) with columns dtype=[('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2'), ('TEMP', '>f4'), ('Q', '>f4'), ('U', '>f4')])
- Destriper configuration file (e.g. ch6_256.cfg) specifying baseline length, sampling rate, and solver settings
- Input smoothed signal map (WMAP 7-year Q band smoothed by 1 degree) for residual/chi2 validation

## Outputs

- Destriped HEALPix I/Q map (Nside 512 in the reference test)
- Binned (plain binning) HEALPix I/Q comparison map
- Reduced chi2 values of residual maps for I and Q, binned vs destriped (written to chi2_table.csv)
- Publicly available destriped maps in HEALPix FITS format (figshare DOI 10.6084/m9.figshare.644507 for B-Machine)

## How to apply

Load the simulated or measured timestream from a single HDF5 file (ch6_fill_3d.h5 for the 3-day test set) with columns TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U, filling flagged samples with white noise so baselines have a fixed number of samples. As a baseline comparison, bin the timestream directly onto a HEALPix Nside 512 map (plain binning), then run the dst destriper with 60-second baselines (2000 samples at 33.4 Hz), solving the destriping equation for baselines with GMRES via Trilinos Belos (serial 'python dst.py ch6_256.cfg' or parallel 'mpirun -np 3 python dst.py ch6_256.cfg'). Build the cython extensions in place first with 'python setup.py build_ext -i'. Compute residual maps by subtracting the 1-degree-smoothed input signal from both binned and destriped maps for I and Q, then compute reduced chi2 of each residual: destriping should drive residuals down to the white noise level (binned I=1.052, Q=1.026; destriped I=1.0002, Q=1.0012). Rationale: destriping acts as a high-pass filter affecting only noise in the timestream, and requires a scanning strategy with frequent crossing points; it works when the 1/f knee frequency is lower than the scan rate.

## Related tools

- **dst** (Python parallel destriper that solves for 60-second baselines via GMRES and produces the destriped HEALPix map) — http://github.com/zonca/dst
- **HEALPix** (Sky pixelization scheme (Nside 512) used for binning, map output, and anafast partial-sky spectra)
- **HDF5** (Binary machine-independent hierarchical format used to store the input timestream (ch6_fill_3d.h5))
- **Trilinos** (PyTrilinos/Belos provides the GMRES iterative solver and MPI communication for distributing baselines and maps)
- **MPI** (Enables parallel reading/writing and running the destriper as 'mpirun -np 3 python dst.py ch6_256.cfg')

## Examples

```
python setup.py build_ext -i ; python dst.py ch6_256.cfg  # or in parallel: mpirun -np 3 python dst.py ch6_256.cfg
```

## Evaluation signals

- Reduced chi2 of destriped residual maps falls to the white noise level: I=1.0002 and Q=1.0012, versus binned values of 1.052 (I) and 1.026 (Q), within a small tolerance.
- Destriped map matches the 1-degree-smoothed input signal after subtraction, i.e. residual maps show no large-scale correlated structure.
- Destriped map is closer to the input signal than the plain-binning map, demonstrating correlated noise removal with minimal signal impact.
- The pipeline runs end-to-end: build_ext succeeds, dst.py reads the HDF5 file with the required dtype columns, and the solver converges under GMRES/Belos.
- Output map is in HEALPix FITS format at the configured Nside and can be validated against the published figshare destriped maps.

## Limitations

- Systematic effects such as gain variations or spin-synchronous effects must be handled separately before map-making; destriping can be ineffective or even harmful for them.
- Destriping requires a scanning strategy that assures frequent crossing points to be effective.
- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility; other data formats require modifying dst_io.read_data.
- Scalability to larger datasets (e.g. the upcoming LATTE ~half-billion-sample dataset) is claimed but not demonstrated.

## Evidence

- [abstract] Destriping is a well-established technique for removing low-frequency correlated noise from Cosmic Microwave Background (CMB) survey data: "Destriping is a well-established technique for removing low-frequency correlated noise from Cosmic Microwave Background (CMB) survey data"
- [methods] we chose a baseline length slightly shorter, of about 60 seconds, i.e. 2000 samples at 33.4 Hz.: "we chose a baseline length slightly shorter, of about 60 seconds, i.e. 2000 samples at 33.4 Hz."
- [readme] currently data needs to be in a single HDF5 file with columns: "currently data needs to be in a single HDF5 file with columns: dtype=[('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2'), ('TEMP', '>f4'), ('Q', '>f4'), ('U', '>f4')])"
- [intro] Destriping can be thought of as a high-pass filter that only affects the noise in the timestream: [evidence span withheld — anchor status: not_found]
