---
name: solar-euv-adaptive-compression-strategy-selection
description: Use when a pipeline must choose between relative- (REL) and absolute-
  (ABS) error-bounded compression strategies per input, e.g. for Solar Orbiter/EUI
  FSI or HRI_EUV FITS images with a given bound (e.g. 1e-3).
when_to_use_negative:
- The input is not solar EUV imaging data (e.g. generic scientific float arrays) —
  SolarZip is specifically optimized for Solar Orbiter EUI data and existing error-bounded
  compressors are noted as imperfect for EUV data
- A fixed, single compression strategy is already specified and no dynamic per-input
  selection is needed
- Lossless compression or scientific-format-preserving transcoding is required rather
  than error-bounded lossy compression
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3812
tools:
- name: SolarZip
  role: Compression framework whose CLI contract (SolarZip -dim_num -bound_mode -bound_num
    <dataDir>) the controller implements and whose REL/ABS error-bounded compressors
    it dispatches to
  repo: github.com/hpdps-group/SolarZip
- name: Python (3.8 or newer)
  role: Language used to implement adaptive_strategy_controller.py
- name: Numpy
  role: Dependency for implementing the controller logic
- name: SolarZip-TestData
  role: Source of FSI and HRI_EUV sample FITS images for running and validating the
    controller
  repo: https://github.com/CapitalLiu/SolarZip-TestData
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202555193
    title: 'SolarZip: An efficient and adaptive compression framework for Solar EUV
      imaging data'
  source_content_hash: sha256:930a9073f553759d8b76026ebb6fb8507e64009831e564772174ad95a4617917
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Numpy
  - Python (3.8 or newer)
  - SolarZip
  - SolarZip-TestData
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3812
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# solar-euv-adaptive-compression-strategy-selection

## Summary

Reconstructs the SolarZip hybrid strategy controller that dynamically selects the optimal error-bounded compression strategy (with optimized interpolation predictors) for each Solar Orbiter/EUI EUV image based on the observational scenario and user requirements, exposing SolarZip's CLI contract.

## When to use

Use when a pipeline must choose between relative- (REL) and absolute- (ABS) error-bounded compression strategies per input, e.g. for Solar Orbiter/EUI FSI or HRI_EUV FITS images with a given bound (e.g. 1e-3).

## When NOT to use

- The input is not solar EUV imaging data (e.g. generic scientific float arrays) — SolarZip is specifically optimized for Solar Orbiter EUI data and existing error-bounded compressors are noted as imperfect for EUV data
- A fixed, single compression strategy is already specified and no dynamic per-input selection is needed
- Lossless compression or scientific-format-preserving transcoding is required rather than error-bounded lossy compression

## Inputs

- Solar Orbiter/EUI FITS images (e.g. solo_L2_eui-hrieuv174-image_20240405T225348216_V01.fits) from the SolarZip-TestData repository
- CLI parameters: dim_num (e.g. 3), bound_mode (REL or ABS), bound_num (e.g. 1e-3), data directory

## Outputs

- adaptive_strategy_controller.py implementing the selection loop
- strategy_selection_report.json with selected strategy and compression ratio per FSI and HRI_EUV file
- Compressed/reconstructed FITS outputs from the dispatched compressor

## How to apply

Implement adaptive_strategy_controller.py in Python (3.8 or newer) with Numpy, exposing the fixed SolarZip CLI contract: dimensionality (dim_num), bound_mode (REL or ABS), and bound_num (e.g. 1e-3). The controller encodes the fixed control loop: read the observational scenario and user requirements, select the adaptive hybrid compression strategy with optimized interpolation predictors, and dispatch to the corresponding relative- or absolute-error-bounded compressor. Run the controller on FSI and HRI_EUV test images from the SolarZip-TestData repository in both REL and ABS modes, recording the selected strategy and compression ratio per file. Qualitatively validate that reported ratios fall within the paper's envelope of up to 800x for FSI and 500x for HRI_EUV.

## Related tools

- **SolarZip** (Compression framework whose CLI contract (SolarZip -dim_num -bound_mode -bound_num <dataDir>) the controller implements and whose REL/ABS error-bounded compressors it dispatches to) — github.com/hpdps-group/SolarZip
- **Python (3.8 or newer)** (Language used to implement adaptive_strategy_controller.py)
- **Numpy** (Dependency for implementing the controller logic)
- **SolarZip-TestData** (Source of FSI and HRI_EUV sample FITS images for running and validating the controller) — https://github.com/CapitalLiu/SolarZip-TestData

## Examples

```
./bin/SolarZip 3 REL 1e-3 /SolarZip/data
```

## Evaluation signals

- Script runs on the documented syntax 'SolarZip -dim_num -bound_mode -bound_num <dataDir>' for both REL and ABS modes (e.g. ./bin/SolarZip 3 REL 1e-3 /SolarZip/data)
- adaptive_strategy_controller.py and strategy_selection_report.json both exist after the run
- strategy_selection_report.json contains strategy entries for both FSI and HRI_EUV instruments
- Compression ratios in the report are consistent with the paper's reported envelope (up to 800x for FSI, up to 500x for HRI_EUV) as a qualitative acceptance check
- Selected strategies cover both relative- and absolute-error-bounded dispatch paths across the REL and ABS test runs

## Limitations

- The paper describes the hybrid strategy controller but does not evaluate it in isolation, so controller behavior can only be validated qualitatively against the paper's compression-ratio envelope
- Existing error-bounded lossy compressors cannot perfectly handle EUV data; ratio ceilings (800x FSI, 500x HRI_EUV) are upper bounds, not guaranteed per-image results
- Requires Python 3.8+ with Numpy (and Astropy/Sunpy for FITS handling); the SolarZip binary itself requires C++ 17 and a CMake build

## Evidence

- [readme] adaptive strategy controller dispatch: "Adaptive Strategy: Employs an adaptive hybrid compression strategy with optimized interpolation predictors."
- [methods] CLI contract: "SolarZip -dim_num -bound_mode -bound_num <dataDir>"
- [abstract] performance envelope: "achieved a data compression ratio of up to 800× for Full Sun Imager (FSI) data and 500× for High Resolution Imager (HRI EUV ) data"
- [other] controller evaluation gap: [evidence span withheld — anchor status: not_found]
- [intro] usability constraint: "Its purpose is to efficiently and adaptively compress the vast amounts of observational data from solar missions while ensuring its scientific usability."
