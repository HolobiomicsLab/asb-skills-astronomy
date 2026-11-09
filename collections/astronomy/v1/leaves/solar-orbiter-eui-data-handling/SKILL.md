---
name: solar-orbiter-eui-data-handling
description: Use when input data are Solar Orbiter/EUI FITS images (L2 hrieuv174 or
  FSI observing files) and the goal is error-bounded lossy compression, rate-distortion
  evaluation, or transmission-time reduction analysis for deep-space downlink planning.
when_to_use_negative:
- Data are not solar EUV imaging observations (the framework is specifically optimized
  for Solar Orbiter EUI data; existing compressors cannot perfectly handle EUV data,
  so generic benchmarks may not transfer)
- Lossless compression is required, since SolarZip is an error-bounded lossy framework
- The hybrid strategy controller must be evaluated in isolation — the paper describes
  it but does not evaluate it independently
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3391
tools:
- name: SolarZip
  role: Error-bounded lossy compressor executed in REL/ABS bound modes on EUI FITS
    data via 'SolarZip -dim_num -bound_mode -bound_num <dataDir>'
  repo: https://github.com/CapitalLiu/SolarZip
- name: Python (3.8 or newer)
  role: Scripting compression-ratio computation and transmission-time simulation with
    Numpy
- name: Numpy
  role: Array handling for computing per-file compression ratios and simulated downlink
    times
- name: SolarZip-TestData
  role: Publicly available FSI and HRI_EUV FITS test datasets with original and reconstructed
    files organized by figure number
  repo: https://github.com/CapitalLiu/SolarZip-TestData
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202555193
    title: 'SolarZip: An efficient and adaptive compression framework for Solar EUV
      imaging data'
  source_content_hash: sha256:b8d596ae5ec884da3e7dd84a22094da0d844510705d4becfff55e745f8a5642b
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# solar-orbiter-eui-data-handling

## Summary

Method for acquiring, compressing, and evaluating Solar Orbiter/EUI solar EUV imaging data (FSI and HRI_EUV FITS files) using the SolarZip error-bounded lossy compression framework, and verifying compression ratios against the paper's reported envelope (up to 800x FSI, 500x HRI_EUV).

## When to use

Use when input data are Solar Orbiter/EUI FITS images (L2 hrieuv174 or FSI observing files) and the goal is error-bounded lossy compression, rate-distortion evaluation, or transmission-time reduction analysis for deep-space downlink planning.

## When NOT to use

- Data are not solar EUV imaging observations (the framework is specifically optimized for Solar Orbiter EUI data; existing compressors cannot perfectly handle EUV data, so generic benchmarks may not transfer)
- Lossless compression is required, since SolarZip is an error-bounded lossy framework
- The hybrid strategy controller must be evaluated in isolation — the paper describes it but does not evaluate it independently

## Inputs

- Solar Orbiter/EUI FITS image files (Full Sun Imager FSI and High Resolution Imager HRI_EUV datasets from SolarZip-TestData)
- Error-bound parameters (dim_num, bound_mode REL/ABS, bound_num such as 1e-3)
- Fixed deep-space link bandwidth value for transmission-time simulation

## Outputs

- Reconstructed FITS files with compression ratio encoded in filename (e.g. SolarZip_cr_831.fits)
- Compression-ratio table per compressor-dataset pair and rate-distortion curves
- Per-file and aggregated simulated transmission times and reduction factors (transmission_times.csv)

## How to apply

Clone github.com/CapitalLiu/SolarZip and build with CMake and make (requires Python 3.8+, Numpy, Astropy, Sunpy; C++17 compiler), yielding an executable at build/bin. Download the SolarZip-TestData repository (github.com/CapitalLiu/SolarZip-TestData), whose folders are named by figure number and contain original FITS files (e.g. solo_L2_eui-hrieuv174-image_20240405T225348216_V01.fits) plus reconstructed files (e.g. SolarZip_cr_831.fits, where cr encodes the compression ratio). Run compression with the CLI 'SolarZip -dim_num -bound_mode -bound_num <dataDir>', sweeping relative (REL) and absolute (ABS) error bounds (e.g. bound 1e-3, dimensionality 3) across FSI and HRI_EUV datasets. Compute per-file compression ratios as uncompressed size / compressed size, and for transmission analysis divide file sizes by a fixed deep-space link bandwidth to derive simulated downlink times. Validate results against the paper's envelope: up to 800x CR for FSI, 500x for HRI_EUV, 3-50x over traditional compressors, and up to 270x transmission-time reduction.

## Related tools

- **SolarZip** (Error-bounded lossy compressor executed in REL/ABS bound modes on EUI FITS data via 'SolarZip -dim_num -bound_mode -bound_num <dataDir>') — https://github.com/CapitalLiu/SolarZip
- **Python (3.8 or newer)** (Scripting compression-ratio computation and transmission-time simulation with Numpy)
- **Numpy** (Array handling for computing per-file compression ratios and simulated downlink times)
- **SolarZip-TestData** (Publicly available FSI and HRI_EUV FITS test datasets with original and reconstructed files organized by figure number) — https://github.com/CapitalLiu/SolarZip-TestData

## Examples

```
./bin/SolarZip 3 REL 1e-3 /SolarZip/data
```

## Evaluation signals

- Maximum FSI compression ratio falls at or near the paper's envelope of 800x (e.g. reconstructed file SolarZip_cr_831.fits indicates CR ~831)
- Maximum HRI_EUV compression ratio approaches 500x
- SolarZip rate-distortion curve shows 3-50x higher compression ratios than traditional compressors and up to 30% over the second-best algorithm
- Aggregated simulated transmission-time reduction factor falls in the range 1-270 with observed maximum at 270
- CLI runs successfully under documented syntax for both REL and ABS modes (e.g. ./bin/SolarZip 3 REL 1e-3 /SolarZip/data)

## Limitations

- Existing error-bounded lossy compressors cannot perfectly handle EUV data, so baseline results may under-represent achievable quality on this data type
- The hybrid strategy controller is described but not evaluated in isolation, so its selection behavior lacks standalone validation
- Scientific usability is asserted via a two-stage evaluation with post hoc scientific analyses; verification requires domain-specific downstream analysis, not just distortion metrics

## Evidence

- [abstract] achieved a data compression ratio of up to 800× for Full Sun Imager (FSI) data and 500× for High Resolution Imager (HRI EUV ) data: "achieved a data compression ratio of up to 800× for Full Sun Imager (FSI) data and 500× for High Resolution Imager (HRI EUV ) data"
- [readme] Command-line Syntax: "SolarZip -dim_num -bound_mode -bound_num <dataDir>
#compress files in the target directory."
- [readme] clone and build instructions: "git clone https://github.com/CapitalLiu/SolarZip.git
cd SolarZip
mkdir build && cd build
cmake ..
make -j8"
- [readme] test data structure: [evidence span withheld — anchor status: external_doc]
- [abstract] transmission time reduction: "SolarZip can reduce data transmission time by up to 270× while ensuring the preservation of scientific usability"
