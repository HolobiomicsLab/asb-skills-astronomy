---
name: pointing-matrix-construction-healpix
description: Use when you need to map polarimeter timeline samples (PHI, THETA, PSI
  angles) onto a HEALPix grid to build the binning operator B or the destriping operator
  for CMB map-making.
when_to_use_negative:
- The timeline contains systematic effects such as thermally induced gain variations
  near sunrise/sunset — these must be masked or handled before map-making, since destriping
  only removes low-frequency correlated noise in the timestream
- The scanning strategy lacks frequent crossing points, which the paper states destriping
  requires to be effective
- The input is not a single HDF5 file with the required columns (TIME, PHI, THETA,
  PSI, FLAG, TEMP, Q, U) and you do not intend to modify dst_io.read_data to support
  another format
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: dst
  role: Python parallel destriper whose Cython extensions build the pointing and binning
    operators and apply them matrix-free during destriping
  repo: http://github.com/zonca/dst
- name: HEALPix
  role: Pixelization scheme on which the Nside 256 sky grid and pixel indices of the
    pointing matrix are defined
- name: h5py
  role: Reads the HDF5 timeline (TIME, PHI, THETA, PSI, FLAG, Q, U columns) from which
    sample pointings are taken
- name: PyTrilinos
  role: Distributes data, baselines and maps among MPI processors; local binned maps
    are reduced to the global map with unique pixels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:e5212e6c3fff8e77ab32e100ddf062c5fa87237ccf1f4ae21ac353fa9be1c907
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HEALPix
  - MPI
  - PyTrilinos
  - Python
  - dst
  - h5py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# pointing-matrix-construction-healpix

## Summary

Constructs the pointing matrix P that maps each polarimeter timestream sample (defined by its PHI, THETA, PSI angles) onto a HEALPix Nside 256 sky pixel. P underpins both the binning operator B and the destriping operator R^T (I - P B^T P^T) R, which are applied via matrix-vector products rather than any dense matrix.

## When to use

Use when you need to map polarimeter timeline samples (PHI, THETA, PSI angles) onto a HEALPix grid to build the binning operator B or the destriping operator for CMB map-making.

## When NOT to use

- The timeline contains systematic effects such as thermally induced gain variations near sunrise/sunset — these must be masked or handled before map-making, since destriping only removes low-frequency correlated noise in the timestream
- The scanning strategy lacks frequent crossing points, which the paper states destriping requires to be effective
- The input is not a single HDF5 file with the required columns (TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U) and you do not intend to modify dst_io.read_data to support another format

## Inputs

- HDF5 timeline file (ch6_fill_3d.h5) with columns dtype=[('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2'), ('TEMP', '>f4'), ('Q', '>f4'), ('U', '>f4')])
- Configuration file ch6_256.cfg specifying the HEALPix Nside 256 grid and destriping parameters

## Outputs

- Implicit sparse pointing matrix P (per-sample HEALPix pixel indices on the Nside 256 grid)
- Binning operator B used to accumulate samples into sky pixels
- Binned and destriped Q and U HEALPix maps (written as FITS files)

## How to apply

Load the timeline from the HDF5 file (e.g. ch6_fill_3d.h5) reading the TIME, PHI, THETA, PSI, FLAG, Q and U columns, and drop samples flagged in the FLAG column. Convert each sample's pointing (PHI, THETA) to a HEALPix pixel index on an Nside 256 grid, as configured in the ch6_256.cfg file; for polarimeters, the PSI polarization angle additionally weights the Q and U contributions of each sample. Represent P implicitly as (sample, pixel) index pairs plus polarization weights instead of forming any dense array, since P has one non-zero block per sample. Build the binning operator B on the same grid so that data can be binned locally and reduced to a global map with unique pixels. Use P and B via matrix-vector products inside the destriping operator R^T (I - P B^T P^T) R, so that neither P nor the dense [n_b, n_b] left-hand matrix is ever materialized.

## Related tools

- **dst** (Python parallel destriper whose Cython extensions build the pointing and binning operators and apply them matrix-free during destriping) — http://github.com/zonca/dst
- **HEALPix** (Pixelization scheme on which the Nside 256 sky grid and pixel indices of the pointing matrix are defined)
- **h5py** (Reads the HDF5 timeline (TIME, PHI, THETA, PSI, FLAG, Q, U columns) from which sample pointings are taken)
- **PyTrilinos** (Distributes data, baselines and maps among MPI processors; local binned maps are reduced to the global map with unique pixels)

## Examples

```
python setup.py build_ext -i; python dst.py ch6_256.cfg
```

## Evaluation signals

- Every unflagged sample maps to exactly one HEALPix pixel on the Nside 256 grid (12*256^2 pixels), with pixel indices consistent with the PHI/THETA angles
- Flagged samples per the FLAG column are excluded from the pointing matrix and hence from the binned map
- The global binned map contains only unique pixels after the distributed sum over processors
- Downstream GMRES solve of the destriping equation converges with the residual reduced by the order of 10^12
- Destriped Q and U outputs exist as HEALPix FITS files with Q and U fields and show reduced correlated-noise striping relative to the binned maps

## Limitations

- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility
- Data must currently reside in a single HDF5 file with the specified columns; other formats require modifying dst_io.read_data
- Scalability to larger datasets (e.g. the upcoming LATTE dataset of about half a billion samples) is claimed but not yet demonstrated
- Destriping can be ineffective or even harmful when systematic effects such as gain variations or spin-synchronous effects are present; these must be dealt with before the map-making stage

## Evidence

- [intro] Pixelization scheme on the HEALPix Nside 256 grid: [evidence span withheld — anchor status: not_found]
- [methods] Distributed binning onto the global map via the pointing/binning operators: [evidence span withheld — anchor status: not_found]
- [methods] HDF5 is the storage format for the timeline from which pointings are read: [evidence span withheld — anchor status: not_found]
