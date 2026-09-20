---
name: opencv-geometric-image-transforms
description: Use when a physically rendered scene image (e.g. Blender Cycles output
  of the 25143 Itokawa model) must be distorted with camera intrinsics to emulate
  a real lens, or when an applied distortion must be inverted for validation.
when_to_use_negative:
- The input is already a distorted sensor image with noise/aberrations applied — remapping
  again would double-apply the distortion
- The goal is comet dust/gas coma modelling, which is handled by the ComaCreator plugin
  rather than the camera model
- Only photorealistic path-traced rendering is needed without instrument effects
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: OpenCV
  role: Implements the Brown–Conrady distortion via cv2.initUndistortRectifyMap, cv2.remap
    and validates invertibility with cv2.undistort
- name: SISPO
  role: Full pipeline whose supplementary OASIS camera model wraps the distortion
    and aberration processing of Cycles renders
  repo: https://github.com/SISPO-developers/sispo
- name: OASIS
  role: Supplementary camera model applying distortion, aberration PSFs and sensor
    noise to rendered images
  repo: https://github.com/SISPO-developers/OASIS
- name: Blender
  role: Produces the input scene images with the Cycles path-tracing engine before
    distortion is applied
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:019418e025bbbb933305654f0e127ea243f7339f5925c771812d72badefba0ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - OASIS
  - OpenCV
  - SISPO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# OpenCV geometric image transforms

## Summary

Applies geometric lens distortion to rendered space images with OpenCV's Brown–Conrady model (radial k1–k6, tangential p1–p2) via initUndistortRectifyMap + remap, as used in SISPO's OASIS supplementary camera model to degrade ideal Cycles renders into realistic instrument images.

## When to use

Use when a physically rendered scene image (e.g. Blender Cycles output of the 25143 Itokawa model) must be distorted with camera intrinsics to emulate a real lens, or when an applied distortion must be inverted for validation.

## When NOT to use

- The input is already a distorted sensor image with noise/aberrations applied — remapping again would double-apply the distortion
- The goal is comet dust/gas coma modelling, which is handled by the ComaCreator plugin rather than the camera model
- Only photorealistic path-traced rendering is needed without instrument effects

## Inputs

- Rendered scene image (e.g. Blender Cycles path-traced output, such as a 25143 Itokawa render)
- Camera intrinsic parameters and Brown–Conrady distortion coefficients (k1–k6, p1, p2)

## Outputs

- Geometrically distorted image (e.g. processed_sample.png) with unchanged dimensions
- Undistorted/recovered image from cv2.undistort used for invertibility validation

## How to apply

Load the rendered scene image together with the camera intrinsic parameters, then apply Brown–Conrady geometric distortion in OpenCV using up to six radial coefficients (k1–k6) and two tangential coefficients (p1, p2) by building maps with cv2.initUndistortRectifyMap and resampling with cv2.remap. In the SISPO OASIS pipeline this distortion step is combined with aberration PSFs (tangential/sagittal astigmatism, internal/external coma whose shape varies with field height) and per-channel rescaling for lateral chromatic aberration, but the geometric transform itself is the OpenCV remap. Rationale: renders from Cycles are ideal pinhole images, so geometric distortion must be applied as a post-process to represent instrument optics. Validate by checking cv2.undistort can invert the applied distortion and by comparing images with the S1 supplementary comparison algorithm.

## Related tools

- **OpenCV** (Implements the Brown–Conrady distortion via cv2.initUndistortRectifyMap, cv2.remap and validates invertibility with cv2.undistort)
- **SISPO** (Full pipeline whose supplementary OASIS camera model wraps the distortion and aberration processing of Cycles renders) — https://github.com/SISPO-developers/sispo
- **OASIS** (Supplementary camera model applying distortion, aberration PSFs and sensor noise to rendered images) — https://github.com/SISPO-developers/OASIS
- **Blender** (Produces the input scene images with the Cycles path-tracing engine before distortion is applied)

## Examples

```
map1, map2 = cv2.initUndistortRectifyMap(K, dist, None, K, (w, h), cv2.CV_32FC1); distorted = cv2.remap(img, map1, map2, cv2.INTER_LINEAR); recovered = cv2.undistort(distorted, K, dist)
```

## Evaluation signals

- processed_sample.png exists with dimensions unchanged relative to the input render
- cv2.undistort applied with the same coefficients recovers the original image (distortion is invertible)
- Cross-check the output against the image-comparison algorithm in the manuscript's S1 supplementary script
- Distorted image shows expected field-dependent geometry: straight scene lines curve radially (k1–k6) and shift tangentially (p1, p2) toward image edges

## Limitations

- ComaCreator explicitly warns it 'should be only used for creating pretty realistic looking pictures' and must not substitute for real science; aberration/noise modelling is similarly representational
- SISPO is open access but 'currently requires basic programming skills to be used', and OASIS usage details live in OASIS_instructions.pdf rather than the README
- Only simple geometric distortions plus tangential/sagittal astigmatism and internal/external coma are covered; more exotic optics are out of scope

## Evidence

- [abstract] Aberrations supported: "tools to simulate the most common image aberrations, such as tangential and sagittal astigmatism, internal and external comatic aberration, and simple geometric distortions"
- [intro] Python scripting of rendering: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python"
- [supplementary] S1 image comparison script: "S1 ScriptImage comparison.The file contains python algorithm used to compare images in this manuscript."
- [readme] OASIS docs location: [evidence span withheld — anchor status: external_doc]
