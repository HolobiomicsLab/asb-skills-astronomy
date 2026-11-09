---
name: brown-conrady-radial-tangential-distortion-remapping
description: Use when a rendered or synthetic image must be degraded with realistic
  lens distortion before downstream computer-vision or 3-D reconstruction tasks, and
  camera intrinsics plus Brown–Conrady k/p coefficients are available.
when_to_use_negative:
- The image is already undistorted/rectified or you need to remove distortion rather
  than apply it (use cv2.undistort directly)
- No camera intrinsics or Brown–Conrady coefficients are available and the distortion
  model cannot be characterised
- The goal is generating dust/gas comet environments or rendering new scenes rather
  than degrading existing renders (use ComaCreator or the Cycles rendering stage instead)
tools:
- name: OpenCV
  role: Implements the Brown–Conrady distortion via cv2.initUndistortRectifyMap and
    cv2.remap; cv2.undistort is used for invertibility validation
- name: Blender
  role: Produces the rendered scene images that the distortion model is applied to
  repo: https://github.com/SISPO-developers/sispo
- name: Cycles
  role: Physically based path-tracing rendering engine generating the input imagery
  repo: https://github.com/SISPO-developers/sispo
- name: OASIS
  role: SISPO's supplementary camera-model module implementing distortion, aberration
    PSFs and sensor noise
  repo: https://github.com/SISPO-developers/OASIS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:620a2fe0459f48ff3ae95a5c1fbd445901fb242e5723f540793f5ae40e8ef7b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - Cycles
  - OASIS
  - OpenCV
  - SISPO
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# Brown-Conrady radial/tangential distortion remapping

## Summary

Simulates camera lens distortion on rendered images (e.g. Blender/Cycles renders from SISPO of the 25143 Itokawa model) using the Brown–Conrady model with up to six radial (k1–k6) and two tangential (p1–p2) coefficients, implemented in OpenCV via map-based remapping, followed by aberration-PSF and sensor-noise stages as in SISPO's OASIS camera model.

## When to use

Use when a rendered or synthetic image must be degraded with realistic lens distortion before downstream computer-vision or 3-D reconstruction tasks, and camera intrinsics plus Brown–Conrady k/p coefficients are available.

## When NOT to use

- The image is already undistorted/rectified or you need to remove distortion rather than apply it (use cv2.undistort directly)
- No camera intrinsics or Brown–Conrady coefficients are available and the distortion model cannot be characterised
- The goal is generating dust/gas comet environments or rendering new scenes rather than degrading existing renders (use ComaCreator or the Cycles rendering stage instead)

## Inputs

- Rendered scene image (e.g. Cycles path-traced render of the 25143 Itokawa asteroid model)
- Camera intrinsic parameter matrix (K)
- Distortion coefficient vector: radial k1–k6, tangential p1, p2

## Outputs

- Geometrically distorted image (e.g. processed_sample.png) with unchanged dimensions
- Remapping tables (mapx, mapy) from cv2.initUndistortRectifyMap for reproducible/invertible distortion
- Optionally: PSF-convolved and chromatic-aberration-corrected image, plus sensor-noise-added image

## How to apply

Load the rendered scene image (produced by Blender's Cycles path-tracing pipeline) together with the camera intrinsic parameters. Build the OpenCV distortion vector with radial coefficients k1–k6 and tangential p1, p2, then apply the geometric distortion with cv2.initUndistortRectifyMap followed by cv2.remap. Optionally continue the OASIS pipeline: convolve with astigmatism/comatic PSFs whose shape and orientation vary with field height, rescale R/G/B channels for lateral chromatic aberration, and add folded-normal dark current, Gaussian readout and Poisson shot noise. Validate by checking that the applied distortion is invertible with cv2.undistort and that output dimensions are unchanged.

## Related tools

- **OpenCV** (Implements the Brown–Conrady distortion via cv2.initUndistortRectifyMap and cv2.remap; cv2.undistort is used for invertibility validation)
- **Blender** (Produces the rendered scene images that the distortion model is applied to) — https://github.com/SISPO-developers/sispo
- **Cycles** (Physically based path-tracing rendering engine generating the input imagery) — https://github.com/SISPO-developers/sispo
- **OASIS** (SISPO's supplementary camera-model module implementing distortion, aberration PSFs and sensor noise) — https://github.com/SISPO-developers/OASIS

## Examples

```
mapx, mapy = cv2.initUndistortRectifyMap(K, np.array([k1,k2,p1,p2,k3]), np.eye(3), K, img.shape[1::-1], cv2.CV_32FC1); distorted = cv2.remap(img, mapx, mapy, cv2.INTER_LINEAR)
```

## Evaluation signals

- processed_sample.png exists after the end-to-end run and has the same pixel dimensions as the input render
- The applied distortion is invertible: cv2.undistort recovers the original image from the distorted output
- Distortion vector contains the expected Brown–Conrady layout (up to k1–k6 radial plus p1, p2 tangential) and remap tables cover the full frame without out-of-map regions
- Cross-check processed vs. reference images with the image-comparison algorithm in the manuscript's S1 supplementary script
- If downstream 3-D reconstruction is the goal, distorted and ground-truth imagery should yield consistent reconstruction error bounds in the SISPO case-study pipeline

## Limitations

- SISPO/OASIS is open access but currently requires basic programming skills to be used
- Full installation and usage instructions for OASIS live in OASIS_instructions.pdf rather than the README
- The distortion simulation is described as 'simple geometric distortions' relative to the full aberration set, so extreme real-lens effects may not be captured

## Evidence

- [full_text] OASIS simulates lens distortion with a Brown–Conrady model implemented in the OpenCV computer-vision library, supporting up to six radial distortion coefficients (k1 to k6) and two tangential distortion coefficients (p1 and p2).: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract lists the aberration capabilities the distortion stage belongs to.: "tools to simulate the most common image aberrations, such as tangential and sagittal astigmatism, internal and external comatic aberration, and simple geometric distortions"
- [intro] The rendering pipeline that supplies the input imagery is Blender/Cycles with Python programming.: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python"
- [full_text] The workflow relies on OpenCV remap operations and invertibility checks.: [evidence span withheld — anchor status: not_found]
- [supplementary] S1 supplementary script provides the image-comparison algorithm used for validation.: "The file contains python algorithm used to compare images in this manuscript."
