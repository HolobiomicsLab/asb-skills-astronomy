---
name: error-bound-auto-tuning-per-level
description: Use when implementing or extending error-bounded lossy compressors for
  multi-level interpolation predictors (e.g., SolarZip on Solar Orbiter EUI FITS data)
  and you must guarantee the user-specified REL/ABS bound is honored despite error
  accumulation across interpolation levels.
when_to_use_negative:
- The data is not error-bounded lossy compression of multi-resolution image grids
  (e.g., it is a generic lossless compression task)
- You need a predictor whose compression performance is independently validated by
  reported metrics — this component is not separately validated in the paper
- The target compressor does not use multi-level interpolation prediction, so per-level
  error accumulation is not a concern
edam_operation: http://edamontology.org/operation_0282
tools:
- name: SolarZip
  role: compression framework whose enhanced spline interpolation predictor incorporates
    level-wise error bound auto-tuning
  repo: github.com/CapitalLiu/SolarZip
- name: C++ 17
  role: implementation language for the enhanced spline predictor layer with level-wise
    error bound auto-tuning
- name: CMake
  role: build system used to compile and install SolarZip with the new predictor layer
    (cmake .., make -j8, make install)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202555193
    title: 'SolarZip: An efficient and adaptive compression framework for Solar EUV
      imaging data'
  source_content_hash: sha256:ac8880168739d249cb2fb30eb87eb0913762884e7af6ef5056b69c5089a70585
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++ 17
  - CMake
  - SolarZip
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# error-bound-auto-tuning-per-level

## Summary

A design feature of SolarZip's enhanced spline interpolation predictor that automatically tunes the error bound level-by-level so accumulated prediction error respects the user-specified bound (REL or ABS). It prevents error accumulation across multi-level interpolation in solar EUV image compression.

## When to use

Use when implementing or extending error-bounded lossy compressors for multi-level interpolation predictors (e.g., SolarZip on Solar Orbiter EUI FITS data) and you must guarantee the user-specified REL/ABS bound is honored despite error accumulation across interpolation levels.

## When NOT to use

- The data is not error-bounded lossy compression of multi-resolution image grids (e.g., it is a generic lossless compression task)
- You need a predictor whose compression performance is independently validated by reported metrics — this component is not separately validated in the paper
- The target compressor does not use multi-level interpolation prediction, so per-level error accumulation is not a concern

## Inputs

- Solar Orbiter EUI solar EUV imaging data (FITS files, e.g., solo_L2_eui-hrieuv174-image_20240405T225348216_V01.fits)
- User-specified error bound mode (REL or ABS) and bound value (e.g., 1e-3)
- SolarZip C++17 compression core source tree

## Outputs

- Compressed SolarZip output files honoring the REL/ABS error bound
- Reconstructed FITS files (e.g., SolarZip_cr_831.fits) after decompression
- SolarZip executable in build/bin

## How to apply

Locate the interpolation predictor structure in the SolarZip C++ compression core, then implement the enhanced spline interpolation predictor layer in C++17, adding level-wise error bound auto-tuning alongside grid-wise anchor points so accumulated prediction error respects the user-specified bound. Wire the predictor layer into SolarZip's adaptive hybrid compression strategy and register it in the CMake build configuration; compile with cmake, make -j8, and make install so the executable appears in build/bin. Exercise the layer by compressing the SolarZip-TestData corpus (solo_L2_eui-hrieuv174-image FITS files) in both relative and absolute bound modes at 1e-3. The component is a method component of the adaptive framework rather than a separately validated result, so correctness is judged by runtime completion in both bound modes, not by reported compression metrics.

## Related tools

- **SolarZip** (compression framework whose enhanced spline interpolation predictor incorporates level-wise error bound auto-tuning) — github.com/CapitalLiu/SolarZip
- **C++ 17** (implementation language for the enhanced spline predictor layer with level-wise error bound auto-tuning)
- **CMake** (build system used to compile and install SolarZip with the new predictor layer (cmake .., make -j8, make install))

## Examples

```
./bin/SolarZip 3 REL 1e-3 /SolarZip/data
```

## Evaluation signals

- SolarZip executable exists in build/bin after cmake .., make -j8, make install
- Compression of the SolarZip-TestData directory completes in REL mode (./bin/SolarZip 3 REL 1e-3 <dataDir>) without runtime failure
- Compression completes in ABS mode (./bin/SolarZip 3 ABS 1e-3 <dataDir>) without runtime failure
- Decompressed FITS output follows the SolarZip_cr_<ratio>.fits naming convention and respects the user-specified error bound

## Limitations

- The enhanced spline interpolation predictor with level-wise error bound auto-tuning is a method component of the adaptive framework, not a separately validated result with reported compression metrics
- Existing error-bounded lossy compressors cannot perfectly handle the EUV data, which motivates but also limits this tuning approach
- Optimized for Solar Orbiter EUI data (FSI and HRI_EUV); applicability to other instruments is not established

## Evidence

- [other] level-wise error bound auto-tuning: [evidence span withheld — anchor status: not_found]
- [other] it is a method component of the adaptive framework rather than a separately validated result: [evidence span withheld — anchor status: not_found]
- [readme] compile and run commands: "cmake ..
make -j8
make install
#The executable will now appear in the build/bin"
- [readme] adaptive strategy: "Employs an adaptive hybrid compression strategy with optimized interpolation predictors."
