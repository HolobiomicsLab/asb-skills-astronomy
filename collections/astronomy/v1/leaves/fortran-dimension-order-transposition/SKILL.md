---
name: fortran-dimension-order-transposition
description: Use when porting an array-based scientific data model with Fortran-order
  dimensions (e.g. HDS primitives) onto HDF5, so multi-dimensional datasets have correctly
  transposed dataspace sizes.
when_to_use_negative:
- When writing purely 1-D or scalar primitives, where Fortran and C order coincide
  and no transposition is needed
- When using the HDF5 Fortran interface directly, which already transposes dimensions
  before calling the C library
- When reading a file that was written by another tool and the intended C/Fortran
  axis convention is unknown — verify with h5dump first rather than assuming
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: HDF5
  role: Target storage layer whose C-order dataspace convention requires transposition
    of HDS Fortran-order dimensions
  repo: https://github.com/Starlink
- name: HDS (Starlink Hierarchical Data System)
  role: Source data model whose primitives and structures use Fortran-order dimensions
    and must be mapped onto HDF5
  repo: https://github.com/Starlink
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:63d2d300bc3719fb8aec412921a4b361850b84f6bca6a35f4574f36e8d42d3fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HDS
  - HDS (Starlink Hierarchical Data System)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fortran-dimension-order-transposition

## Summary

Transposes HDS Fortran-order dimensions into HDF5 C order when defining HDF5 dataspaces, a required step because HDF5 uses C order while HDS uses Fortran order even in its C layer.

## When to use

Use when porting an array-based scientific data model with Fortran-order dimensions (e.g. HDS primitives) onto HDF5, so multi-dimensional datasets have correctly transposed dataspace sizes.

## When NOT to use

- When writing purely 1-D or scalar primitives, where Fortran and C order coincide and no transposition is needed
- When using the HDF5 Fortran interface directly, which already transposes dimensions before calling the C library
- When reading a file that was written by another tool and the intended C/Fortran axis convention is unknown — verify with h5dump first rather than assuming

## Inputs

- HDS primitive with Fortran-order dimensions (hdsdim 32-bit int array)
- HDS structure hierarchy to be written as HDF5 datasets (e.g. /HISTORY group with _CHAR*24 and _INTEGER primitives)
- HDSLoc containing dataset and dataspace identifiers for slice/cell/vector operations

## Outputs

- HDF5 datasets whose dataspaces are specified in C order (transposed from the HDS Fortran-order dimensions)
- h5dump listing (h5dump_listing.txt) showing the structurally mapped hierarchy with correctly shaped primitives

## How to apply

When creating any multi-dimensional HDF5 dataset that represents an HDS primitive, reverse the HDS Fortran-order dimension array before passing it to the HDF5 dataspace-creation call (H5Screate_simple). The hdsdim type in HDS is a 32-bit int while HDF5 uses the unsigned 64-bit hsize_t, so convert dimension types during the transposition. The same rationale applies to slicing and vectorization: datSlice/datCell select subsets of a dataspace, and datVec must present rank-1 vectorized dimensions regardless of the underlying dataset's rank. This mirrors what the HDF5 Fortran interface already does — it transposes dimensions before calling into the C library — and must be replicated inside the HDSv5 wrapper so that datShape and hdsTrace report HDS-syntax-consistent shapes.

## Related tools

- **HDF5** (Target storage layer whose C-order dataspace convention requires transposition of HDS Fortran-order dimensions) — https://github.com/Starlink
- **HDS (Starlink Hierarchical Data System)** (Source data model whose primitives and structures use Fortran-order dimensions and must be mapped onto HDF5) — https://github.com/Starlink

## Examples

```
hdf5_file = h5py.File('history.hdf', 'w'); dset = hdf5_file.create_dataset('/HISTORY/CREATED', data=char24_value, dtype='S24')  # dims transposed to C order via H5Screate_simple
```

## Evaluation signals

- h5dump listing shows /HISTORY with Attr:TYPE=HISTORY, /CREATED (_CHAR*24), /CURRENT_RECORD (_INTEGER), and /RECORDS with HDSCELL(1)..HDSCELL(10) children each with TYPE=HIST_REC and /DATE members
- Shapes reported by datShape match the HDS (Fortran-order) dimensions even though the HDF5 dataspace is stored in C order
- _LOGICAL values are stored as 1-byte bitfields, not 4-byte integers, in the HDF5 file
- hdsTrace converts /HISTORY/RECORDS/HDSCELL(5) to the syntactically correct .HISTORY.RECORDS(5) by removing all occurrences of "/HDSCELL"
- Slicing/vectorized locators (datSlice, datCell, datVec) return dataspace selections consistent with the transposed representation

## Limitations

- HDF5 does not support arrays of structures, so array-of-structure slices and vectorization (heavily used by HDSTRACE) require extra work — contiguous N-D subsets are selected by choosing points from the dataspace
- HDS uses 32-bit hdsdim while HDF5 uses unsigned 64-bit hsize_t; switching types requires all code to properly use hdsdim
- HDS only supports sub-setting, not sampling or scatter-gather, so dataspace selection semantics are narrower than raw HDF5 hyperslab selection

## Evidence

- [other] When creating any multi-dimensional primitive, transpose the HDS Fortran-order dimensions into HDF5 C order when specifying dataspace sizes: [evidence span withheld — anchor status: not_found]
