---
name: floating-point-subtractive-dithering-quantization
description: Use when compressing floating-point astronomical image or binary-table
  data into HDF5 and you need a dithered quantization filter comparable to FPACK's
  scaled-Rice mode, or when benchmarking HDF5 filters (lzf, gzip, bitshuffle) against
  FITS Rice baselines.
when_to_use_negative:
- When genuinely lossless storage is required — subtractive dithering is quantization-based,
  so output only matches the original within the quantization error, unlike the paper's
  lossless LZF/bitshuffle comparisons
- When the input data are integers already quantized on disk (the paper's baseline
  Rice comparison applied to integer binary-table data); dithering targets floating-point
  pixel streams
- When the goal is only FITS-to-HDF5 format conversion without compression experiments
  — plain fits2hdf/hdf2fits is sufficient
tools:
- name: fits2hdf
  role: Reads FITS into the IDI in-memory interchange format (read_fits) and writes
    compressed HDFITS files (export_hdf); the vehicle for applying the new filter
  repo: https://github.com/telegraphic/fits2hdf
- name: h5py
  role: Python interface used to implement and register the HDF5 dynamic filter and
    manage the filter pipeline (shuffle, fletcher32, compression kwargs)
  repo: http://www.h5py.org/
- name: bitshuffle
  role: Reference implementation of an HDF5 dynamically loaded filter (filter number
    32008) to model filter registration on
  repo: https://github.com/kiyo-masui/bitshuffle
- name: FPACK (fpack)
  role: Source of the subtractive-dithering quantization algorithm and scaled-Rice
    baseline comparator (ascl:1010.002)
- name: CFITSIO
  role: Underlying FITS I/O library used by fits2hdf and the FITS compression baseline
    path
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:4f5e58e015fa4ffcef62fbfb8abccddd8b278a721e12c48d0ab17275fa4dc393
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - CFITSIO
  - FPACK (fpack)
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# floating-point-subtractive-dithering-quantization

## Summary

Port the FPACK (ascl:1010.002) subtractive-dithering quantization algorithm for floating-point astronomical data into HDF5 as a dynamically registered compression filter, filling a capability the HDFITS paper explicitly notes HDF5 lacks, and benchmark its compression ratio against FITS scaled-Rice compression.

## When to use

Use when compressing floating-point astronomical image or binary-table data into HDF5 and you need a dithered quantization filter comparable to FPACK's scaled-Rice mode, or when benchmarking HDF5 filters (lzf, gzip, bitshuffle) against FITS Rice baselines.

## When NOT to use

- When genuinely lossless storage is required — subtractive dithering is quantization-based, so output only matches the original within the quantization error, unlike the paper's lossless LZF/bitshuffle comparisons
- When the input data are integers already quantized on disk (the paper's baseline Rice comparison applied to integer binary-table data); dithering targets floating-point pixel streams
- When the goal is only FITS-to-HDF5 format conversion without compression experiments — plain fits2hdf/hdf2fits is sufficient

## Inputs

- Floating-point astronomical image in FITS format
- FPACK (fpack) source code for the dithering pseudo-random sequence and quantization step
- fits2hdf HDFITS environment (Python, h5py, HDF5) with CFITSIO available for the FITS baseline

## Outputs

- HDF5 dataset compressed with the custom subtractive-dithering dynamic filter (via export_hdf)
- FITS file compressed with fpack scaled-Rice mode as the baseline
- dithering_compression_report.csv with per-method compression ratios and round-trip error metrics

## How to apply

Load a floating-point FITS image via fits2hdf.io.fitsio.read_fits into the in-memory IDI interchange format, then implement an HDF5 dynamic (registered) filter in Python/h5py that reproduces FPACK's subtractive dithering: consult the FPACK source for the pseudo-random dithering sequence and quantization step size, quantize each pixel after adding the dither offset so quantization error averages out. Compress the dataset through fits2hdf.io.hdfio.export_hdf with the new filter, and in parallel with the available HDF5 options (-c None/lzf/gzip/bitshuffle, optionally -S shuffle and -C fletcher32). Produce the baseline by running fpack in scaled-Rice mode on the same image. Report compression ratio (uncompressed bytes / compressed bytes) per method and verify round-trip decompression reproduces pixels within the quantization error; write results to dithering_compression_report.csv. Rationale: the paper states there is 'currently no equivalent to the subtractive dithering' in HDF5, so this must be implemented as a user-level dynamic filter (bitshuffle shows the pattern, HDF5 filter number 32008, integrated via the dynamically loaded filters framework).

## Related tools

- **fits2hdf** (Reads FITS into the IDI in-memory interchange format (read_fits) and writes compressed HDFITS files (export_hdf); the vehicle for applying the new filter) — https://github.com/telegraphic/fits2hdf
- **h5py** (Python interface used to implement and register the HDF5 dynamic filter and manage the filter pipeline (shuffle, fletcher32, compression kwargs)) — http://www.h5py.org/
- **bitshuffle** (Reference implementation of an HDF5 dynamically loaded filter (filter number 32008) to model filter registration on) — https://github.com/kiyo-masui/bitshuffle
- **FPACK (fpack)** (Source of the subtractive-dithering quantization algorithm and scaled-Rice baseline comparator (ascl:1010.002))
- **CFITSIO** (Underlying FITS I/O library used by fits2hdf and the FITS compression baseline path)

## Examples

```
from fits2hdf.io.fitsio import read_fits; from fits2hdf.io.hdfio import export_hdf; idc = read_fits('image.fits'); export_hdf(idc, 'image.h5', compression='lzf')
```

## Evaluation signals

- Round-trip decompression of the dithering-filter dataset reproduces original pixels within the declared quantization error
- Compression ratio (uncompressed bytes / compressed bytes) computed for both the dithering filter and the fpack scaled-Rice baseline is present for every method in dithering_compression_report.csv
- The HDF5 dataset's filter pipeline shows the registered dynamic filter ID alongside any shuffle/fletcher32 stages
- HDF5-side filter results are contextualized against the paper's lossless options (None/lzf/gzip/bitshuffle) so relative ratios are interpretable
- hdf2fits round-trip of the HDFITS file still succeeds, confirming the filter does not break format compatibility

## Limitations

- The paper explicitly did not compare lossy compression algorithms; dithering results cannot be validated against published HDFITS numbers
- HDF5 has no built-in equivalent to FITS subtractive dithering, so correctness depends entirely on a faithful port of the FPACK dithering sequence and quantization step
- HDF5 datasets using a custom dynamic filter are only readable where that filter is registered (the bitshuffle README notes dynamic filters must be available at read time)
- Quantization error is data-dependent: low-SNR or noise-dominated float images may lose meaningful precision if the quantization step is chosen too coarsely

## Evidence

- [discussion] no dithering equivalent in HDF5: [evidence span withheld — anchor status: not_found]
- [readme] dynamic filter registration pattern: [evidence span withheld — anchor status: external_doc]
- [readme] h5py filter pipeline options: [evidence span withheld — anchor status: not_found]
