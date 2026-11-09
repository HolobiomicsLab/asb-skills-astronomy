---
name: lateral-chromatic-aberration-channel-rescaling
description: Use when generating synthetic space-mission imagery that must include
  realistic chromatic aberration — e.g. after Cycles path-traced rendering and Brown–Conrady
  distortion, before sensor-noise injection — to stress-test visual-navigation or
  3-D reconstruction algorithms.
when_to_use_negative:
- The goal is calibration/correction of real chromatic aberration in an existing image
  rather than its simulation for rendered synthetic imagery
- The pipeline step requires geometric (radial/tangential) distortion modelling —
  use the Brown–Conrady/OpenCV undistort-remap workflow instead of channel rescaling
- Rendering is done outside SISPO/OASIS and no aberration simulation is required for
  the downstream algorithm
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: OpenCV
  role: Used to implement the OASIS camera model, including the Brown–Conrady distortion
    stage whose output the per-channel rescaling operates on; also used to verify
    distortion is invertible via cv2.undistort
- name: Blender
  role: Produces the rendered scene images that OASIS processes with chromatic aberration
    simulation
  repo: github.com/SISPO-developers/sispo
- name: Cycles
  role: Path-tracing rendering engine within Blender generating the physically based
    input images
  repo: github.com/SISPO-developers/sispo
- name: SISPO
  role: Hosts the OASIS supplementary camera model that simulates image aberrations
    including lateral chromatic aberration
  repo: https://github.com/SISPO-developers/sispo
- name: OASIS
  role: Supplementary camera model module implementing the per-channel rescaling for
    lateral chromatic aberration
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:e7181fd847ac6912037a9cdffd2560ed69ec61e622084d17124eb18cb6c52b09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Blender
  - Cycles
  - OASIS
  - OpenCV
  - SISPO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Lateral chromatic aberration channel rescaling

## Summary

A camera-model artefact simulation step from SISPO's OASIS supplementary imaging module that emulates lateral chromatic aberration by rescaling the R, G, and B channels of a rendered image with slightly different per-channel scale factors and recombining them, so downstream computer-vision algorithms are tested on imagery with realistic colour-fringing.

## When to use

Use when generating synthetic space-mission imagery that must include realistic chromatic aberration — e.g. after Cycles path-traced rendering and Brown–Conrady distortion, before sensor-noise injection — to stress-test visual-navigation or 3-D reconstruction algorithms.

## When NOT to use

- The goal is calibration/correction of real chromatic aberration in an existing image rather than its simulation for rendered synthetic imagery
- The pipeline step requires geometric (radial/tangential) distortion modelling — use the Brown–Conrady/OpenCV undistort-remap workflow instead of channel rescaling
- Rendering is done outside SISPO/OASIS and no aberration simulation is required for the downstream algorithm

## Inputs

- Rendered RGB scene image from the Blender Cycles path-tracing pipeline (e.g. of the 25143 Itokawa asteroid model)
- Camera intrinsic parameters (after Brown–Conrady distortion with k1–k6 radial and p1, p2 tangential coefficients)

## Outputs

- RGB image with laterally chromatic-aberration-simulated per-channel rescaling, unchanged in dimensions (e.g. processed_sample.png)

## How to apply

Take the rendered scene image (produced by Blender's Cycles path-tracing pipeline, e.g. of the 25143 Itokawa model) after geometric Brown–Conrady distortion has been applied in OpenCV. Decompose the image into its R, G, and B channels and rescale each channel with a slightly different per-channel scale factor — small deviations around 1.0 that mimic the wavelength-dependent focal-length shift of lateral chromatic aberration. Recombine the rescaled channels into a single RGB image, keeping dimensions unchanged. This step is performed as part of OASIS's full aberration chain (alongside field-height-dependent astigmatism/coma PSF convolution) and precedes the addition of folded-normal dark current, Gaussian readout, and Poisson shot noise. Validate by confirming the processed image keeps its original dimensions and by cross-checking against the image-comparison algorithm in the manuscript's S1 supplementary script.

## Related tools

- **OpenCV** (Used to implement the OASIS camera model, including the Brown–Conrady distortion stage whose output the per-channel rescaling operates on; also used to verify distortion is invertible via cv2.undistort)
- **Blender** (Produces the rendered scene images that OASIS processes with chromatic aberration simulation) — github.com/SISPO-developers/sispo
- **Cycles** (Path-tracing rendering engine within Blender generating the physically based input images) — github.com/SISPO-developers/sispo
- **SISPO** (Hosts the OASIS supplementary camera model that simulates image aberrations including lateral chromatic aberration) — https://github.com/SISPO-developers/sispo
- **OASIS** (Supplementary camera model module implementing the per-channel rescaling for lateral chromatic aberration)

## Examples

```
import cv2, numpy as np; img = cv2.imread('rendered.png').astype(np.float32); img[...,0] *= 1.002; img[...,1] *= 1.0005; img[...,2] *= 0.999; cv2.imwrite('processed_sample.png', np.clip(img,0,255).astype(np.uint8))
```

## Evaluation signals

- The processed output image (e.g. processed_sample.png) exists and has unchanged dimensions relative to the input rendered image
- Per-channel scale factors are close to 1.0 — only slight differences among R, G, B — so the effect mimics lateral (not gross) chromatic shift
- Recombined RGB channels show wavelength-dependent edge fringing consistent with simulated chromatic aberration, invertible/verifiable against the S1 image-comparison script
- The end-to-end OASIS run completes with distortion still invertible via OpenCV's cv2.undistort
- Downstream steps (PSF convolution for astigmatism/coma, then sensor noise) are applied in the documented order around this step

## Limitations

- Lateral chromatic aberration is approximated by simple per-channel rescaling rather than a full wavelength-dependent optical model, so it captures geometric colour fringing but not detailed spectral PSF differences
- The SISPO tool requires basic programming skills to use, limiting accessibility
- OASIS usage instructions are only available in OASIS_instructions.pdf; the repository README provides no parameter details

## Evidence

- [methods] Lateral chromatic aberration simulated via per-channel rescaling: [evidence span withheld — anchor status: not_found]
- [abstract] SISPO simulates common image aberrations: "tools to simulate the most common image aberrations, such as tangential and sagittal astigmatism, internal and external comatic aberration, and simple geometric distortions"
- [discussion] Rendered images from Cycles path tracing: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [readme] OASIS repository documentation is minimal: [evidence span withheld — anchor status: external_doc]
