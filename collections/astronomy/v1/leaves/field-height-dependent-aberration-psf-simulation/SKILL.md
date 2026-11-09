---
name: field-height-dependent-aberration-psf-simulation
description: Use when synthetic space-scene images (e.g. Blender Cycles renders of
  the 25143 Itokawa model) must be degraded with realistic lens aberrations and sensor
  noise to test imaging instruments or computer-vision algorithms.
when_to_use_negative:
- The images are real sensor observations rather than synthetic renders from SISPO/Blender
  Cycles — the model parameters and validation script assume the OASIS synthetic pipeline.
- The goal is dust- or gas-environment (coma) optical modelling of comets or active
  asteroids — that is handled by the ComaCreator plugin, not the OASIS camera model.
- A simple geometric distortion without astigmatism, coma, chromatic aberration or
  sensor noise suffices — SISPO's core rendering path covers that without the OASIS
  module.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OpenCV
  role: Implements the Brown–Conrady distortion model via cv2.initUndistortRectifyMap,
    cv2.remap and cv2.undistort
- name: OASIS
  role: Supplementary camera model of SISPO that performs the aberration-PSF and sensor-noise
    simulation
  repo: https://github.com/SISPO-developers/OASIS
- name: SISPO
  role: Produces the rendered scene images that OASIS processes
  repo: github.com/SISPO-developers/sispo
- name: Blender
  role: 3-D visualisation system used to generate the input scene renders
- name: Cycles
  role: Physically based path-tracing rendering engine producing the images fed to
    the OASIS model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:acf70eaf39a8ed7574b3056478e7033bde3387bbcf40d7fe00408faa3362a031
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/OASIS
  tools:
  - Blender
  - Cycles
  - OASIS
  - OpenCV
  - SISPO
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Field-height-dependent aberration PSF simulation

## Summary

Simulates optical imaging artefacts — Brown–Conrady geometric distortion, field-height-dependent astigmatism and coma PSFs, lateral chromatic aberration and sensor noise — on synthetic SISPO/Blender Cycles renders via the OASIS supplementary camera model implemented in OpenCV.

## When to use

Use when synthetic space-scene images (e.g. Blender Cycles renders of the 25143 Itokawa model) must be degraded with realistic lens aberrations and sensor noise to test imaging instruments or computer-vision algorithms.

## When NOT to use

- The images are real sensor observations rather than synthetic renders from SISPO/Blender Cycles — the model parameters and validation script assume the OASIS synthetic pipeline.
- The goal is dust- or gas-environment (coma) optical modelling of comets or active asteroids — that is handled by the ComaCreator plugin, not the OASIS camera model.
- A simple geometric distortion without astigmatism, coma, chromatic aberration or sensor noise suffices — SISPO's core rendering path covers that without the OASIS module.

## Inputs

- Rendered scene image produced by the Blender Cycles path-tracing pipeline (e.g. of the 25143 Itokawa asteroid model)
- Camera intrinsic parameters (intrinsic matrix K)
- Distortion coefficients (k1–k6 radial, p1–p2 tangential)
- Per-channel chromatic aberration scale factors for R, G, B

## Outputs

- Distorted/aberrated sensor image (e.g. processed_sample.png) with unchanged pixel dimensions relative to the input render
- Invertible distortion map reproducible with cv2.undistort

## How to apply

Load the rendered scene image together with its camera intrinsic parameters, then apply Brown–Conrady distortion in OpenCV using up to six radial coefficients (k1–k6) and two tangential coefficients (p1, p2) via cv2.initUndistortRectifyMap followed by cv2.remap. Next, simulate tangential and sagittal astigmatism and internal/external comatic aberration by convolving the image with PSFs whose shape and orientation vary with field height (distance from the optical axis). Model lateral chromatic aberration by rescaling the R, G and B channels with slightly different per-channel scale factors and recombining them. Finally, add sensor noise composed of folded-normal distributed dark current, Gaussian readout noise, and Poisson shot noise. The rationale is to inject physically motivated, spatially varying degradation so that downstream 3-D reconstruction and navigation algorithms are validated under realistic instrument behaviour.

## Related tools

- **OpenCV** (Implements the Brown–Conrady distortion model via cv2.initUndistortRectifyMap, cv2.remap and cv2.undistort)
- **OASIS** (Supplementary camera model of SISPO that performs the aberration-PSF and sensor-noise simulation) — https://github.com/SISPO-developers/OASIS
- **SISPO** (Produces the rendered scene images that OASIS processes) — github.com/SISPO-developers/sispo
- **Blender** (3-D visualisation system used to generate the input scene renders)
- **Cycles** (Physically based path-tracing rendering engine producing the images fed to the OASIS model)

## Examples

```
map1, map2 = cv2.initUndistortRectifyMap(K, dist, None, K, (w, h), cv2.CV_32FC1); img_out = cv2.remap(img_in, map1, map2, cv2.INTER_LINEAR)
```

## Evaluation signals

- processed_sample.png exists after running the module end-to-end on a sample rendered image
- Output image dimensions are unchanged relative to the input rendered scene
- Applied distortion is invertible with OpenCV's cv2.undistort
- Cross-check the processed result against the image-comparison algorithm provided in the manuscript's S1 supplementary script
- Noise statistics match the specified composition: folded-normal dark current, Gaussian readout noise, and Poisson shot noise

## Limitations

- OASIS's repository README contains no usage detail; full installation and usage instructions live in a separate OASIS_instructions.pdf, which must be consulted.
- SISPO and its modules are open access but require basic programming skills to use.
- PSF shape and orientation vary with field height, so aberrations are spatially varying — results are only valid for the camera intrinsics and field heights actually configured.

## Evidence

- [full_text] Aberration PSFs vary with field height: [evidence span withheld — anchor status: not_found]
- [full_text] Brown-Conrady distortion in OpenCV: [evidence span withheld — anchor status: not_found]
- [full_text] Chromatic aberration modelling: [evidence span withheld — anchor status: not_found]
- [full_text] Sensor noise model: [evidence span withheld — anchor status: not_found]
- [abstract] SISPO aberration capabilities: "tools to simulate the most common image aberrations, such as tangential and sagittal astigmatism, internal and external comatic aberration, and simple geometric distortions"
- [readme] OASIS documentation location: [evidence span withheld — anchor status: external_doc]
