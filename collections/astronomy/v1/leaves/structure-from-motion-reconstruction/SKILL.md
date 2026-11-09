---
name: structure-from-motion-reconstruction
description: Use when synthetic or real imagery of a small-body target is available
  from multiple known/unrecoverable viewing angles and the goal is to recover camera
  poses, a point cloud, and a fully reconstructed 3-D model whose accuracy must be
  quantified against a reference mesh.
when_to_use_negative:
- Input is a single image or too few viewing angles — structure-from-motion will not
  converge to a reliable pose/point-cloud solution
- The target already has a high-fidelity reference model and no reconstruction accuracy
  study is needed — direct simulation via SISPO alone suffices
- The scene involves cometary dust/gas activity rendering (ComaCreator) rather than
  solid-body 3-D reconstruction — the SfM/OpenMVS pipeline does not address that
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0593
tools:
- name: SISPO
  role: Orchestrates the pipeline from simulated imagery to final 3-D data products;
    renders the orbiting scenario images
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: 3-D visualisation host in which the target model is loaded and rendered, programmable
    in Python
  repo: https://github.com/SISPO-developers/sispo
- name: Cycles
  role: Physically based path-tracing rendering engine used to generate photorealistic
    images from the input 3-D model
- name: OpenMVG
  role: 'Structure-from-motion: recovers camera poses and a sparse point cloud from
    the rendered image set'
- name: OpenMVS
  role: 'Multi-view stereo: densifies the sparse point cloud and meshes it into the
    fully reconstructed 3-D model'
- name: sispo_docker
  role: Docker image recommended for the easiest deployment of SISPO and its environment
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:c990c29db49fba6f3a6e51daf80e88922dd242caf4115fbc1e835f26f742f3c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - Cycles
  - OpenMVG
  - OpenMVS
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# structure-from-motion-reconstruction

## Summary

Full pipeline that renders synthetic images of a 3-D model (e.g., the 25143 Itokawa SSSB) with Blender/Cycles via SISPO, then recovers a 3-D model from those images using OpenMVG structure-from-motion and OpenMVS multi-view stereo densification/meshing, and quantifies reconstruction deviation against the input reference model.

## When to use

Use when synthetic or real imagery of a small-body target is available from multiple known/unrecoverable viewing angles and the goal is to recover camera poses, a point cloud, and a fully reconstructed 3-D model whose accuracy must be quantified against a reference mesh.

## When NOT to use

- Input is a single image or too few viewing angles — structure-from-motion will not converge to a reliable pose/point-cloud solution
- The target already has a high-fidelity reference model and no reconstruction accuracy study is needed — direct simulation via SISPO alone suffices
- The scene involves cometary dust/gas activity rendering (ComaCreator) rather than solid-body 3-D reconstruction — the SfM/OpenMVS pipeline does not address that

## Inputs

- Input 3-D model (e.g., 25143 Itokawa asteroid shape model) in a mesh format loadable by SISPO/Blender (e.g., .obj)
- Scenario definition file specifying trajectory, light parameters, and camera intrinsic parameters (pinhole camera, viewing-angle geometry)
- Rendered image set from multiple viewing angles (e.g., 53 orbiting images)

## Outputs

- Recovered camera poses and sparse point cloud (from OpenMVG structure-from-motion)
- Dense point cloud, meshed and fully reconstructed 3-D model (from OpenMVS)
- Point-deviation statistics comparing the reconstructed model to the input reference model (mean ~1.2 m, std ~6.6 m for the orbiting SSSB case; 60–89.7 m at crater spots)

## How to apply

Load the 25143 Itokawa asteroid 3-D model into SISPO and configure the orbiting (or fly-by) scenario; render images with a pinhole-camera model from different viewing angles using Blender's Cycles path-tracing engine (the orbiting case study used 53 images). Run structure-from-motion on the rendered image set with OpenMVG to recover camera poses and a sparse point cloud, then run multi-view stereo densification and meshing with OpenMVS to obtain the fully reconstructed 3-D model. Finally, use the image-comparison Python algorithm from supplementary script S1 to compute point-wise deviation statistics of the reconstruction against the input reference model. Decision points: prefer Docker installation via sispo_docker for environment reproducibility, and consider adding more images and/or higher resolution if crater-spot deviations exceed acceptable bounds (reported 60–89.7 m at crater spots vs. Gaussian mean 1.2 m, std 6.6 m for the orbiting case).

## Related tools

- **SISPO** (Orchestrates the pipeline from simulated imagery to final 3-D data products; renders the orbiting scenario images) — https://github.com/SISPO-developers/sispo
- **Blender** (3-D visualisation host in which the target model is loaded and rendered, programmable in Python) — https://github.com/SISPO-developers/sispo
- **Cycles** (Physically based path-tracing rendering engine used to generate photorealistic images from the input 3-D model)
- **OpenMVG** (Structure-from-motion: recovers camera poses and a sparse point cloud from the rendered image set)
- **OpenMVS** (Multi-view stereo: densifies the sparse point cloud and meshes it into the fully reconstructed 3-D model)
- **sispo_docker** (Docker image recommended for the easiest deployment of SISPO and its environment) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
pip install sispo; sispo --with-render --with-sim
```

## Evaluation signals

- Computed Gaussian mean error of model deviations reproduces ~1.2 m for the orbiting SSSB case with 53 pinhole images
- Computed standard deviation of model deviations reproduces ~6.6 m
- Deviation outliers at crater spots fall in the reported 60–89.7 m range
- OpenMVG output yields a coherent sparse point cloud and consistent camera poses before OpenMVS densification is run
- Reconstruction accuracy improves (error decreases) when more images and higher resolution are supplied, as stated in the case study

## Limitations

- Large local deviations (60–89.7 m) occur at crater spots even when global mean error is low
- Accuracy depends on number of images and resolution; fewer/lower-resolution images degrade the reconstruction
- Using SISPO and the pipeline currently requires basic programming skills, limiting accessibility
- SISPO installation can be simplified via Docker, but the VM must be given sufficient disk and cores (default 10 GB is not enough)

## Evidence

- [intro] SISPO uses Cycles rendering with the Blender software package: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python"
- [discussion] The tool generates physically based, photorealistic images from an input 3-D model via path-tracing rendering: [evidence span withheld — anchor status: not_found]
- [supplementary] S1 Script image comparison algorithm: "The file contains python algorithm used to compare images in this manuscript."
- [readme] Docker installation recommended: [evidence span withheld — anchor status: external_doc]
- [intro] 25143 Itokawa model used as input: "The 25143 Itokawa asteroid model by the [14] was used."
