---
name: point-cloud-deviation-statistics
description: Use when a reconstructed 3-D model (e.g., an OpenMVS dense point cloud
  from SfM imagery) must be validated against a known reference surface, and error
  must be reported as Gaussian mean/std deviation statistics.
when_to_use_negative:
- No ground-truth reference surface exists (e.g., target body is unknown before close
  encounter), so point-to-mesh comparison cannot be anchored.
- The reconstruction was not produced from the same reference geometry or registered
  into a common frame — distances would be dominated by misalignment, not reconstruction
  error.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CloudCompare
  role: Used to load both the input shape model and the reconstructed model and compute
    point-to-mesh Gaussian distances between them
- name: OpenMVS
  role: Produces the dense multi-view stereo reconstruction whose deviation from the
    reference model is measured
- name: OpenMVG
  role: Incremental structure-from-motion over the 25-image series yielding camera
    poses and sparse point cloud that feed the dense reconstruction
- name: SISPO
  role: Renders the synthetic 25-image fly-by series (via Blender Cycles) that serves
    as input to the SfM/MVS pipeline being validated
  repo: https://github.com/SISPO-developers/sispo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:dbfae93ef8496f48585ba0943af04ffc026d12616e5e1b6ee3d2774467777030
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - CloudCompare
  - OpenMVG
  - OpenMVS
  - SISPO
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# point-cloud-deviation-statistics

## Summary

Quantify the accuracy of a 3-D reconstruction against a reference shape model by computing point-to-mesh Gaussian distances in CloudCompare and summarizing them as Gaussian mean and standard deviation statistics. In the SISPO SSSB fly-by case this yields a mean error of 14.7 m and standard deviation of 63.9 m.

## When to use

Use when a reconstructed 3-D model (e.g., an OpenMVS dense point cloud from SfM imagery) must be validated against a known reference surface, and error must be reported as Gaussian mean/std deviation statistics.

## When NOT to use

- No ground-truth reference surface exists (e.g., target body is unknown before close encounter), so point-to-mesh comparison cannot be anchored.
- The reconstruction was not produced from the same reference geometry or registered into a common frame — distances would be dominated by misalignment, not reconstruction error.

## Inputs

- Reconstructed 3-D model (dense point cloud from OpenMVS on OpenMVG SfM output)
- Reference/input shape model (e.g., 25143 Itokawa asteroid mesh used in SISPO rendering)
- Fly-by scenario parameters (25-image pinhole-camera series, trajectory, light settings) for context

## Outputs

- Point-to-mesh Gaussian distance values between reconstruction and reference model
- Gaussian error statistics (mean and standard deviation), e.g. mean 14.7 m, std 63.9 m for the fly-by case
- Stats CSV with mean and standard deviation fields
- Flagged worst-case deviations (e.g., edge deviations up to 600 m)

## How to apply

Load both the input reference shape model (e.g., the 25143 Itokawa model) and the reconstructed model (from OpenMVG/OpenMVS) into CloudCompare. Compute point-to-mesh Gaussian distances between the two models so each point of the reconstruction is measured against the reference surface. Summarize the resulting distance distribution as Gaussian statistics (mean and standard deviation) and write them to a stats CSV containing explicit mean and standard deviation fields. Additionally flag worst-case deviations, such as those occurring at model edges, to capture tail behaviour the Gaussian summary alone hides.

## Related tools

- **CloudCompare** (Used to load both the input shape model and the reconstructed model and compute point-to-mesh Gaussian distances between them)
- **OpenMVS** (Produces the dense multi-view stereo reconstruction whose deviation from the reference model is measured)
- **OpenMVG** (Incremental structure-from-motion over the 25-image series yielding camera poses and sparse point cloud that feed the dense reconstruction)
- **SISPO** (Renders the synthetic 25-image fly-by series (via Blender Cycles) that serves as input to the SfM/MVS pipeline being validated) — https://github.com/SISPO-developers/sispo

## Examples

```
cloudcompare.CloudCompare -o reconstructed.ply -o itokawa_reference.obj -computeDist Gaussian
```

## Evaluation signals

- Reported Gaussian mean error should be close to the published fly-by value of 14.7 m for the same 25-image pinhole-camera SSSB case
- Reported Gaussian standard deviation should be close to 63.9 m for the same case
- Stats CSV must contain explicit mean and standard deviation fields
- Worst-case deviations should be identified and flagged (published edge deviations reach up to 600 m)
- Distance distribution should be summarized as Gaussian statistics, not raw maxima alone

## Limitations

- Worst-case deviations concentrate at model edges, where errors can reach up to 600 m — the Gaussian mean/std alone understate tail errors
- Reported accuracy is specific to the 25-image pinhole-camera fly-by scenario; different image counts, camera models, or trajectory configurations will shift the statistics
- SISPO usage requires basic programming skills, limiting reproducibility of the upstream image generation

## Evidence

- [results] Gaussian mean error of 14.7 m and a standard deviation of 63.9 m: [evidence span withheld — anchor status: not_found]
- [other] CloudCompare point-to-mesh Gaussian distances: [evidence span withheld — anchor status: not_found]
- [other] Summarize deviations as Gaussian error statistics: [evidence span withheld — anchor status: not_found]
- [other] Validation criteria from published case: [evidence span withheld — anchor status: not_found]
- [methods] OpenMVS dense reconstruction input: [evidence span withheld — anchor status: not_found]
