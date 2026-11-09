---
name: momentum-target-network-update
description: 'Use when training BYOL directly as a torch.nn.Module (not via ByolTrainer)
  on astronomical images: after each optimization step you must sync the target network
  with the online network via an exponential moving average.'
when_to_use_negative:
- You are using the `ByolTrainer` class or the `pipeline_01_training.py` script, which
  already call the update internally — manual invocation would be redundant.
- You are running inference only (`pipeline_02_inference.py`) on an already trained
  network; no optimization steps occur and no moving-average update applies.
- use_momentum is set to false, in which case the target network is not maintained
  as an exponential moving average of the online network.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: astromorph
  role: Provides the `BYOL` class exposing the `update_moving_average()` method that
    must be called after every optimization step
  repo: github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework in which `BYOL` is implemented as a torch.nn.Module; optimizer steps
    trigger the EMA update
- name: Python 3.12
  role: Runtime environment for the training loop that interleaves optimizer steps
    with update_moving_average() calls
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:38c46a4fb4a33726e976a37207dc205ee4c4e4cbddf2e0d26f05aa18518890b4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - Python 3.12
  - astromorph
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# momentum-target-network-update

## Summary

Implements the BYOL target-network update from astromorph: after every optimizer step, the target network is replaced by an exponential moving average of the online network's weights. Calling `update_moving_average()` after each step is mandatory when training the raw `BYOL` PyTorch module.

## When to use

Use when training BYOL directly as a torch.nn.Module (not via ByolTrainer) on astronomical images: after each optimization step you must sync the target network with the online network via an exponential moving average.

## When NOT to use

- You are using the `ByolTrainer` class or the `pipeline_01_training.py` script, which already call the update internally — manual invocation would be redundant.
- You are running inference only (`pipeline_02_inference.py`) on an already trained network; no optimization steps occur and no moving-average update applies.
- use_momentum is set to false, in which case the target network is not maintained as an exponential moving average of the online network.

## Inputs

- BYOL module instance (torch.nn.Module) with online and target encoders (e.g. AstroMorphologyModel or reduced ResNet18 truncated at a chosen last_layer such as 'layer2')
- Two augmented views per image from a PyTorch DataLoader over FITS files (e.g. via FitsFilelistDataset)
- Optimizer state after each gradient step

## Outputs

- Target network parameters updated to the exponential moving average of the online network parameters
- Trained BYOL encoder whose representations are inspected downstream via the inference script / embeddings

## How to apply

Instantiate the `BYOL` class (a subclass of `torch.nn.Module`) with an online encoder, a target encoder, and the projector (defaults: representation_size=128, projection_size=16, projection_hidden_size=512, use_momentum=true). During training, feed two independently augmented views of each image through the online and target branches, compute the BYOL loss, and run the optimizer step. Immediately after every optimization step, call the `update_moving_average()` method so the target network becomes the exponential moving average of the online network, as prescribed by the original BYOL paper (arXiv:2006.07733). If you omit this call, the target network drifts out of sync and the self-supervised training objective degrades; the README explicitly warns about this. Alternatively, use the `ByolTrainer` wrapper or the training script (`pipeline_01_training.py`), which handle this update internally.

## Related tools

- **astromorph** (Provides the `BYOL` class exposing the `update_moving_average()` method that must be called after every optimization step) — github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework in which `BYOL` is implemented as a torch.nn.Module; optimizer steps trigger the EMA update)
- **Python 3.12** (Runtime environment for the training loop that interleaves optimizer steps with update_moving_average() calls)

## Examples

```
loss = byol(view1, view2); loss.backward(); optimizer.step(); byol.update_moving_average()
```

## Evaluation signals

- A smoke test instantiates `BYOL`, runs a forward pass, and calls `update_moving_average()` without error.
- After an optimizer step, target-network parameters change (EMA applied); without calling update_moving_average(), target weights remain frozen between steps.
- With use_momentum=true, target parameters are a convex combination favoring the previous target weights (exponential moving average), not copies of the online weights.
- Downstream check: resulting embeddings cluster/augment meaningfully in TensorBoard visualization and export_to_csv output, indicating the BYOL objective was optimized correctly.
- Training reproduces expected behavior described in the original BYOL paper (arXiv:2006.07733), where the target network is the exponential MA of the online network.

## Limitations

- The README warns: 'NB: remember to call the `update_moving_average()` method after every optimization step' — omitting it silently breaks training since no exception is raised.
- Using the raw `BYOL` class 'requires some experience with setting up a training routine in PyTorch'; less experienced users should prefer `ByolTrainer` or the training script.
- The project was developed for Python 3.12; lower versions are not guaranteed and anything below Python 3.9 will not work.

## Evidence

- [readme] NB: remember to call the `update_moving_average()` method after every optimization step. If you do not know why you have to do this, please have a look at the original paper: "NB: remember to call the `update_moving_average()` method after every optimization step. If you do not know why you have to do this, please have a look at the original paper"
- [methods] the `BYOL` class as a PyTorch implementation of the BYOL framework;: "the `BYOL` class as a PyTorch implementation of the BYOL framework;"
- [abstract] The package is built with usability in mind, and offers streamlined pipeline scripts for ease of use as well as deeper customization options via PyTorch-based classes.: "The package is built with usability in mind, and offers streamlined pipeline scripts for ease of use as well as deeper customization options via PyTorch-based classes."
- [readme] use_momentum = true # Target network is exponential MA of online network.: "use_momentum = true # Target network is exponential MA of online network."
