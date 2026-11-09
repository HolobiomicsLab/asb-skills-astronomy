---
name: camera-sensor-noise-modeling
description: Use when rendered space-scene images (e.g. Blender/Cycles renders of
  the 25143 Itokawa model) must be degraded with realistic lens and sensor artifacts
  to characterise imaging-instrument performance or validate computer-vision navigation
  algorithms.
when_to_use_negative:
- The input is already real sensor data containing native lens distortion and noise
  — this skill simulates degradation of synthetic renders, not calibration of measured
  imagery
- Only scene geometry, lighting, or comet dust/gas environments are needed — use SISPO/ComaCreator
  rendering without the OASIS camera-degradation stage
- The task requires high-precision photometric radiometry rather than instrument-performance
  characterisation for mission planning or computer-vision algorithm development
edam_operation: http://edamontology.org/operation_3443
tools:
- name: OASIS
  role: Supplementary SISPO camera model that implements the distortion, aberration-PSF,
    chromatic-aberration and sensor-noise pipeline
  repo: https://github.com/SISPO-developers/OASIS
- name: OpenCV
  role: Implements the Brown–Conrady distortion via cv2.initUndistortRectifyMap/cv2.remap
    and provides cv2.undistort for invertibility validation
- name: Blender
  role: Produces the input rendered scene images through its Cycles path-tracing engine
  repo: https://github.com/SISPO-developers/sispo
- name: SISPO
  role: Host pipeline from simulated imagery to data products into which the OASIS
    camera model feeds
  repo: https://github.com/SISPO-developers/sispo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:2ce5d697d24064a138a40d9660a0a4776c1a90736549eb1fc95b5d04aa9fdf1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/OASIS
  tools:
  - Blender
  - OASIS
  - OpenCV
  - SISPO
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# camera-sensor-noise-modeling

## Summary

Simulates realistic camera artifacts on rendered space-scene images by applying Brown–Conrady lens distortion, field-height-dependent aberration PSFs, lateral chromatic aberration, and a physical sensor-noise model (folded-normal dark current, Gaussian readout, Poisson shot noise), as implemented in SISPO's OASIS supplementary camera model.

## When to use

Use when rendered space-scene images (e.g. Blender/Cycles renders of the 25143 Itokawa model) must be degraded with realistic lens and sensor artifacts to characterise imaging-instrument performance or validate computer-vision navigation algorithms.

## When NOT to use

- The input is already real sensor data containing native lens distortion and noise — this skill simulates degradation of synthetic renders, not calibration of measured imagery
- Only scene geometry, lighting, or comet dust/gas environments are needed — use SISPO/ComaCreator rendering without the OASIS camera-degradation stage
- The task requires high-precision photometric radiometry rather than instrument-performance characterisation for mission planning or computer-vision algorithm development

## Inputs

- Rendered scene image from Blender/Cycles path tracing (e.g. of the 25143 Itokawa asteroid model)
- Camera intrinsic parameters (intrinsic matrix, distortion coefficients k1–k6, p1, p2)
- PSF definitions for astigmatism and coma as a function of field height
- Per-channel scale factors for lateral chromatic aberration
- Sensor-noise parameters (dark-current level, readout-noise sigma, shot-noise gain)

## Outputs

- Distorted, aberrated, noise-degraded image (e.g. processed_sample.png) with unchanged dimensions
- Invertible distortion maps usable with cv2.undistort

## How to apply

Load a rendered scene image produced by the Blender Cycles path-tracing pipeline together with the camera intrinsic parameters. Apply Brown–Conrady geometric distortion in OpenCV via cv2.initUndistortRectifyMap followed by cv2.remap, using up to six radial coefficients (k1–k6) and two tangential coefficients (p1, p2). Simulate tangential and sagittal astigmatism and internal/external comatic aberration by convolving the image with PSFs whose shape and orientation vary with field height (distance from the optical axis). Model lateral chromatic aberration by rescaling the R, G, and B channels with slightly different per-channel scale factors and recombining them. Add sensor noise composed of folded-normal distributed dark current (folded because physical dark current cannot be negative), Gaussian readout noise, and Poisson shot noise. Validate by confirming the output (e.g. processed_sample.png) exists with unchanged dimensions and that the applied distortion is invertible with cv2.undistort, cross-checking against the image-comparison algorithm in the manuscript's S1 supplementary script.

## Related tools

- **OASIS** (Supplementary SISPO camera model that implements the distortion, aberration-PSF, chromatic-aberration and sensor-noise pipeline) — https://github.com/SISPO-developers/OASIS
- **OpenCV** (Implements the Brown–Conrady distortion via cv2.initUndistortRectifyMap/cv2.remap and provides cv2.undistort for invertibility validation)
- **Blender** (Produces the input rendered scene images through its Cycles path-tracing engine) — https://github.com/SISPO-developers/sispo
- **SISPO** (Host pipeline from simulated imagery to data products into which the OASIS camera model feeds) — https://github.com/SISPO-developers/sispo

## Examples

```
map1, map2 = cv2.initUndistortRectifyMap(K, dist, np.eye(3), K, img.shape[1::-1], cv2.CV_32FC1); img_dist = cv2.remap(img, map1, map2, cv2.INTER_LINEAR)
```

## Evaluation signals

- Output file (processed_sample.png) exists and has dimensions identical to the input rendered image
- Applied Brown–Conrady distortion is invertible: cv2.undistort recovers the original image geometry
- Processed image matches expectations when run through the manuscript's S1 supplementary image-comparison algorithm
- PSF blur varies with field height (stronger astigmatism/coma away from the optical axis) rather than being a uniform convolution
- Noise statistics separate into three components: Poisson shot noise scaling with signal, Gaussian readout noise, and non-negative folded-normal dark current

## Limitations

- OASIS's repository README does not contain usage details — full installation and usage instructions are only in OASIS_instructions.pdf
- SISPO is open access but requires basic programming skills to use, which limits accessibility
- Distortion support is limited to the Brown–Conrady form (up to k1–k6 radial, p1–p2 tangential); more complex unmodelled lens effects are not covered

## Evidence

- [abstract] aberrations and distortion scope: "tools to simulate the most common image aberrations, such as tangential and sagittal astigmatism, internal and external comatic aberration, and simple geometric distortions"
- [full_text] sensor noise composition: [evidence span withheld — anchor status: not_found]
- [intro] rendering backend: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python"
- [readme] OASIS docs pointer: [evidence span withheld — anchor status: external_doc]
