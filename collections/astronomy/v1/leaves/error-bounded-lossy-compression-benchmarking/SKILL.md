---
name: error-bounded-lossy-compression-benchmarking
description: Use when comparing compression ratios of an error-bounded lossy compressor
  against baselines on scientific imaging data (e.g. Solar EUV FITS from FSI or HRI_EUV)
  under identical error-bound settings.
when_to_use_negative:
- The input is not solar EUV imaging data (e.g. non-imaging time series or non-FITS
  formats) — SolarZip is specifically optimized for Solar Orbiter/EUI data
- The goal is lossless compression or exact data preservation — error-bounded lossy
  compression trades reconstruction fidelity for ratios
- Downstream scientific analysis requires bit-exact reproducibility of the raw observational
  data rather than scientifically usable reconstructions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: SolarZip
  role: Adaptive error-bounded lossy compressor being benchmarked; built from source
    with CMake and run via 'SolarZip -dim_num -bound_mode -bound_num <dataDir>'
  repo: github.com/CapitalLiu/SolarZip
- name: SolarZip-TestData
  role: Publicly available FSI and HRI_EUV FITS test datasets with original and reconstructed
    files used as benchmark inputs
  repo: github.com/CapitalLiu/SolarZip-TestData
- name: Python/Numpy
  role: Used to compute compression ratios for every compressor-dataset pair and assemble
    comparison tables and rate-distortion plots
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202555193
    title: 'SolarZip: An efficient and adaptive compression framework for Solar EUV
      imaging data'
  source_content_hash: sha256:22b40cb22d75586cbb5d2a5ff2ec28e723035fa5a4b0e84093153af6121fa7be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Full Sun Imager (FSI)
  - High Resolution Imager (HRI_EUV)
  - Numpy
  - Python
  - Python (3.8 or newer)
  - Python/Numpy
  - SolarZip
  - SolarZip-TestData
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# error-bounded-lossy-compression-benchmarking

## Summary

Benchmarks SolarZip against traditional error-bounded lossy compressors on Solar Orbiter/EUI FITS images by sweeping REL and ABS error bounds (e.g. 1e-3) and comparing compression ratios and rate-distortion curves.

## When to use

Use when comparing compression ratios of an error-bounded lossy compressor against baselines on scientific imaging data (e.g. Solar EUV FITS from FSI or HRI_EUV) under identical error-bound settings.

## When NOT to use

- The input is not solar EUV imaging data (e.g. non-imaging time series or non-FITS formats) — SolarZip is specifically optimized for Solar Orbiter/EUI data
- The goal is lossless compression or exact data preservation — error-bounded lossy compression trades reconstruction fidelity for ratios
- Downstream scientific analysis requires bit-exact reproducibility of the raw observational data rather than scientifically usable reconstructions

## Inputs

- Solar Orbiter/EUI FSI and HRI_EUV FITS image datasets from SolarZip-TestData (50 months of observations)
- Baseline error-bounded lossy compressors run under identical error-bound settings

## Outputs

- Compression ratio table (uncompressed size / compressed size) for every compressor-dataset pair
- Rate-distortion curves comparing SolarZip against the baseline compressors
- Reconstructed FITS files named e.g. SolarZip_cr_831.fits indicating compressor and compression ratio

## How to apply

Clone the SolarZip repository and build it with CMake and make; obtain the FSI and HRI_EUV test datasets from the SolarZip-TestData repository (original and reconstructed FITS files, e.g. solo_L2_eui-hrieuv174-image_20240405T225348216_V01.fits). Run SolarZip on each dataset using the syntax 'SolarZip -dim_num -bound_mode -bound_num <dataDir>' in both REL and ABS modes (e.g. dimensionality 3 with bound 1e-3), sweeping the bound values. Run the baseline error-bounded lossy compressors on the same datasets under identical error-bound settings, then compute compression ratios as uncompressed size / compressed size for every compressor-dataset pair and plot rate-distortion curves. Judge success by checking that SolarZip attains up to 800x reduction on FSI and 500x on HRI_EUV data, exceeding traditional algorithms by 3-50x and the second-best algorithm by up to 30%, and by verifying it shows the best overall performance on the rate-distortion curve.

## Related tools

- **SolarZip** (Adaptive error-bounded lossy compressor being benchmarked; built from source with CMake and run via 'SolarZip -dim_num -bound_mode -bound_num <dataDir>') — github.com/CapitalLiu/SolarZip
- **SolarZip-TestData** (Publicly available FSI and HRI_EUV FITS test datasets with original and reconstructed files used as benchmark inputs) — github.com/CapitalLiu/SolarZip-TestData
- **Python/Numpy** (Used to compute compression ratios for every compressor-dataset pair and assemble comparison tables and rate-distortion plots)

## Examples

```
./bin/SolarZip 3 REL 1e-3 /SolarZip/data
```

## Evaluation signals

- SolarZip reaches up to 800x compression ratio on FSI data and 500x on HRI_EUV data
- SolarZip exceeds traditional algorithms by 3-50x and the second-best algorithm by up to 30%
- SolarZip demonstrates overall best performance on the rate-distortion curve
- Reconstructed FITS filenames encode compressor and ratio (e.g. SolarZip_cr_831.fits) and can be cross-checked against computed ratios
- Baseline compressors were run under identical REL/ABS error-bound settings so ratios are directly comparable

## Limitations

- Existing error-bounded lossy compressors cannot perfectly handle EUV data, so baseline ratios are inherently lower
- Peak ratios (800x/500x) are upper bounds tied to specific bound sweeps; other bound values yield different ratios
- Scientific usability must be verified with a two-stage evaluation (distortion metrics plus domain-specific post hoc scientific analyses) before accepting high ratios

## Evidence

- [abstract] achieved a data compression ratio of up to 800× for Full Sun Imager (FSI) data and 500× for High Resolution Imager (HRI EUV ) data: "achieved a data compression ratio of up to 800× for Full Sun Imager (FSI) data and 500× for High Resolution Imager (HRI EUV ) data"
- [abstract] achieving 3-50x higher ratios: "achieving 3−50× higher compression ratios than traditional algorithms, surpassing the second-best algorithm by up to 30%"
- [abstract] two-stage evaluation methodology: "a comprehensive two-stage evaluation methodology integrating standard distortion metrics with domain-specific post hoc scientific analyses"
- [methods] command-line syntax: "SolarZip -dim_num -bound_mode -bound_num <dataDir>"
- [readme] build and clone instructions: "git clone https://github.com/CapitalLiu/SolarZip.git"
- [readme] rate-distortion result: "SolarZip demonstrated the overall best performance on the rate-distortion curve."
