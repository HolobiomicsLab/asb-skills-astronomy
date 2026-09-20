---
name: compression-ratio-comparison-analysis
description: Use when evaluating a new error-bounded lossy compressor on solar EUV
  imaging data (FITS from Solar Orbiter EUI FSI/HRI_EUV) and you need compression-ratio
  comparisons versus baseline compressors under identical error bounds.
when_to_use_negative:
- The data are not solar EUV imaging data (e.g. non-imaging time series or non-EUV
  instruments) — SolarZip is specifically optimized for Solar Orbiter EUI data
- The goal is lossless compression or throughput/latency benchmarking rather than
  error-bounded compression-ratio comparison
- Only a single compressor is of interest with no baseline comparison — the skill's
  value is the controlled head-to-head ratio sweep
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3277
tools:
- name: SolarZip
  role: Target compressor under evaluation, run in REL and ABS error-bound modes with
    swept bound values
  repo: https://github.com/CapitalLiu/SolarZip
- name: SolarZip-TestData
  role: Publicly available FSI and HRI_EUV FITS test datasets (original and reconstructed
    files organized by figure number)
  repo: https://github.com/CapitalLiu/SolarZip-TestData
- name: Python (3.8 or newer)
  role: Computes per-file compression ratios, assembles the comparison table, and
    plots rate-distortion curves
- name: Numpy
  role: Numerical support for ratio computation and tabulation in Python
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202555193
    title: 'SolarZip: An efficient and adaptive compression framework for Solar EUV
      imaging data'
  source_content_hash: sha256:1153c10440497a5400a852e6f29551a2f1394bded781a3154340ad4aaeca4e5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CapitalLiu/SolarZip
  tools:
  - Full Sun Imager (FSI)
  - High Resolution Imager (HRI_EUV)
  - Numpy
  - Python
  - Python (3.8 or newer)
  - SolarZip
  - SolarZip-TestData
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3277
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# compression-ratio-comparison-analysis

## Summary

Benchmark SolarZip's error-bounded lossy compression ratios against traditional compressors on Solar Orbiter/EUI FSI and HRI_EUV FITS data, sweeping REL/ABS error bounds to assemble a comparison table and rate-distortion curves.

## When to use

Use when evaluating a new error-bounded lossy compressor on solar EUV imaging data (FITS from Solar Orbiter EUI FSI/HRI_EUV) and you need compression-ratio comparisons versus baseline compressors under identical error bounds.

## When NOT to use

- The data are not solar EUV imaging data (e.g. non-imaging time series or non-EUV instruments) — SolarZip is specifically optimized for Solar Orbiter EUI data
- The goal is lossless compression or throughput/latency benchmarking rather than error-bounded compression-ratio comparison
- Only a single compressor is of interest with no baseline comparison — the skill's value is the controlled head-to-head ratio sweep

## Inputs

- Solar Orbiter/EUI Full Sun Imager (FSI) FITS image datasets from SolarZip-TestData
- Solar Orbiter/EUI High Resolution Imager (HRI_EUV) FITS image datasets from SolarZip-TestData
- SolarZip executable built via CMake/make
- Four baseline error-bounded lossy compressors run under identical error-bound settings

## Outputs

- Per-file compression ratios (uncompressed size / compressed size) for every compressor-dataset pair
- Comparison table of compression ratios across compressors and datasets
- Rate-distortion curves showing SolarZip's overall best performance

## How to apply

Download the SolarZip-TestData deposit containing original and reconstructed FITS files (e.g. solo_L2_eui-hrieuv174-image_20240405T225348216_V01.fits) for FSI and HRI_EUV instruments. Build SolarZip from source with CMake and make, then run it on each dataset in relative (REL) and absolute (ABS) error-bound modes using the syntax 'SolarZip -dim_num -bound_mode -bound_num <dataDir>', sweeping bound values such as 1e-3. Run the four baseline error-bounded lossy compressors on the same datasets under identical error-bound settings to ensure a fair comparison. For each compressor-dataset pair, compute the compression ratio as uncompressed size divided by compressed size, assemble the comparison table, and plot rate-distortion curves. Validate against the paper's reported maxima of 800x for FSI and 500x for HRI_EUV, with SolarZip exceeding traditional algorithms by 3-50x and the second-best by up to 30%.

## Related tools

- **SolarZip** (Target compressor under evaluation, run in REL and ABS error-bound modes with swept bound values) — https://github.com/CapitalLiu/SolarZip
- **SolarZip-TestData** (Publicly available FSI and HRI_EUV FITS test datasets (original and reconstructed files organized by figure number)) — https://github.com/CapitalLiu/SolarZip-TestData
- **Python (3.8 or newer)** (Computes per-file compression ratios, assembles the comparison table, and plots rate-distortion curves)
- **Numpy** (Numerical support for ratio computation and tabulation in Python)

## Examples

```
./bin/SolarZip 3 REL 1e-3 /SolarZip/data
```

## Evaluation signals

- SolarZip's compression ratio reaches up to 800x for FSI data and 500x for HRI_EUV data
- SolarZip's ratio exceeds the four traditional compressors by 3-50x and the second-best algorithm by up to 30%
- SolarZip demonstrates the overall best performance on the rate-distortion curve
- Every compressor-dataset pair is evaluated under identical REL/ABS error-bound settings (e.g. dimensionality 3, bound 1e-3)
- Compression ratio for each file equals uncompressed size divided by compressed size

## Limitations

- Reported maxima are upper bounds on specific FSI/HRI_EUV datasets; other instruments or bound values may yield lower ratios
- Existing error-bounded lossy compressors cannot perfectly handle EUV data, so baseline behavior is instrument-dependent
- Comparison fairness depends on running all compressors under identical error-bound settings; mismatched bounds invalidate the 3-50x and 30% claims

## Evidence

- [abstract] compression ratios vs baselines: "achieved a data compression ratio of up to 800× for Full Sun Imager (FSI) data and 500× for High Resolution Imager (HRI EUV ) data"
- [abstract] advantage over baselines: "achieving 3−50× higher compression ratios than traditional algorithms, surpassing the second-best algorithm by up to 30%"
- [methods] CLI usage: "SolarZip -dim_num -bound_mode -bound_num <dataDir>"
- [readme] test data availability: "We have made the test data publicly available. If you're interested, you can access it here👉: 
https://github.com/CapitalLiu/SolarZip-TestData"
- [readme] build instructions: "cmake ..
make -j8
make install"
