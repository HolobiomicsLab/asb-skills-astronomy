---
name: rate-distortion-curve-generation
description: Use when comparing error-bounded lossy compressors on image or volume
  datasets (e.g., FITS solar EUV images) and you need compression-ratio-vs-distortion
  trade-off curves to select or rank a compressor.
when_to_use_negative:
- Data must remain bit-for-bit identical (lossless requirement) — error-bounded lossy
  compression and rate-distortion analysis are inapplicable
- Data are not scientific images/volumes with a meaningful error bound (e.g., unstructured
  text or categorical data), so REL/ABS error-bound sweeps have no defined distortion
  metric
edam_operation: http://edamontology.org/operation_0337
tools:
- name: SolarZip
  role: Compressor under test; run with REL/ABS error bounds at swept bound values
    to generate points on the rate-distortion curve
  repo: https://github.com/CapitalLiu/SolarZip
- name: SolarZip-TestData
  role: Public FSI and HRI_EUV FITS datasets (original + reconstructed files per compressor)
    used to compute curve points and verify visual fidelity
  repo: https://github.com/CapitalLiu/SolarZip-TestData
- name: Numpy
  role: Python dependency for computing compression ratios/distortion metrics and
    assembling rate-distortion plots
- name: Python (3.8 or newer)
  role: Environment for metric computation and plotting the rate-distortion comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202555193
    title: 'SolarZip: An efficient and adaptive compression framework for Solar EUV
      imaging data'
  source_content_hash: sha256:d5357e83e5a548128c6778fc0f5d1d01317c776c94b3b9457f75fc5406b2ad25
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# rate-distortion-curve-generation

## Summary

Generates rate-distortion curves that trade compression ratio against distortion metrics for lossy scientific compressors, as done for SolarZip versus four baseline error-bounded compressors on Solar Orbiter/EUI FSI and HRI_EUV image data.

## When to use

Use when comparing error-bounded lossy compressors on image or volume datasets (e.g., FITS solar EUV images) and you need compression-ratio-vs-distortion trade-off curves to select or rank a compressor.

## When NOT to use

- Data must remain bit-for-bit identical (lossless requirement) — error-bounded lossy compression and rate-distortion analysis are inapplicable
- Data are not scientific images/volumes with a meaningful error bound (e.g., unstructured text or categorical data), so REL/ABS error-bound sweeps have no defined distortion metric

## Inputs

- Solar Orbiter/EUI FSI and HRI_EUV image datasets (FITS files, e.g., solo_L2_eui-hrieuv174-image_20240405T225348216_V01.fits)
- SolarZip-TestData repository datasets (original FITS plus reconstructed FITS per compressor)
- Compressor configurations: error bound mode (REL/ABS), bound values (e.g., 1e-3), dimensionality (e.g., 3)

## Outputs

- Rate-distortion curve plot (compression ratio vs. distortion) comparing SolarZip against four baseline compressors
- Per compressor-dataset-bound compression ratios (e.g., up to 800x FSI, 500x HRI_EUV)
- Reconstructed FITS files at each compression ratio (e.g., SolarZip_cr_831.fits)

## How to apply

For each compressor (SolarZip and the four baseline error-bounded lossy compressors), sweep the error bound values in both relative (REL) and absolute (ABS) modes, invoking the compressor on the same Solar Orbiter/EUI FITS datasets — Full Sun Imager (FSI) and High Resolution Imager (HRI_EUV) — with the CLI 'SolarZip -dim_num -bound_mode -bound_num <dataDir>' (e.g., dimensionality 3 with a 1e-3 bound). Compute compression ratio as uncompressed size / compressed size and distortion metrics for every compressor-dataset-bound setting. Plot rate-distortion curves (compression ratio on one axis, distortion on the other) with Python and NumPy, using reconstructed FITS files such as 'SolarZip_cr_831.fits' to verify data fidelity at each compression ratio. Follow the two-stage evaluation methodology that integrates standard distortion metrics with domain-specific post hoc scientific analyses so that scientific usability, not just numerical distortion, is judged at each point on the curve.

## Related tools

- **SolarZip** (Compressor under test; run with REL/ABS error bounds at swept bound values to generate points on the rate-distortion curve) — https://github.com/CapitalLiu/SolarZip
- **SolarZip-TestData** (Public FSI and HRI_EUV FITS datasets (original + reconstructed files per compressor) used to compute curve points and verify visual fidelity) — https://github.com/CapitalLiu/SolarZip-TestData
- **Numpy** (Python dependency for computing compression ratios/distortion metrics and assembling rate-distortion plots)
- **Python (3.8 or newer)** (Environment for metric computation and plotting the rate-distortion comparison)

## Examples

```
./bin/SolarZip 3 REL 1e-3 /SolarZip/data
```

## Evaluation signals

- SolarZip demonstrates the overall best performance on the rate-distortion curve relative to the four baselines
- Compression ratios reach up to 800x for FSI data and 500x for HRI_EUV data, exceeding traditional algorithms by 3-50x and the second-best algorithm by up to 30%
- Compression ratio is consistently computed as uncompressed size / compressed size for every compressor-dataset pair
- Reconstructed FITS files at high ratios (e.g., SolarZip_cr_831.fits) remain scientifically usable in downstream analyses, per the two-stage evaluation methodology

## Limitations

- Existing error-bounded lossy compressors cannot perfectly handle EUV data, so baseline curve points may not reflect scientifically usable reconstructions at high ratios
- Peak ratios (800x FSI, 500x HRI_EUV) are bounds attained at particular error-bound settings; sweeping different REL/ABS bound values shifts the achievable ratio-distortion trade-off
- The README credits an unrelated elasticmm build badge/doc badge, so repository status badges should not be treated as evidence of SolarZip's own CI state

## Evidence

- [methods] SolarZip demonstrated the overall best performance on the rate-distortion curve.: "SolarZip demonstrated the overall best performance on the rate-distortion curve."
- [abstract] Two-stage evaluation methodology: "a comprehensive two-stage evaluation methodology integrating standard distortion metrics with domain-specific post hoc scientific analyses"
- [readme] CLI syntax and bound modes: "SolarZip -dim_num -bound_mode -bound_num <dataDir>
#compress files in the target directory."
- [abstract] Peak compression ratios: "achieved a data compression ratio of up to 800× for Full Sun Imager (FSI) data and 500× for High Resolution Imager (HRI EUV ) data"
- [readme] Reconstructed file naming convention: "Example: `SolarZip_cr_831.fits`
  - Indicates the reconstructed data from the SolarZip compressor at a compression ratio of 831."
