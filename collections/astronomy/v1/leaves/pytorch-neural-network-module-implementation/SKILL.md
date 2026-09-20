---
name: pytorch-neural-network-module-implementation
description: Use when you need a customizable PyTorch module for self-supervised BYOL
  training on unlabeled astronomical images (e.g. FITS files) and want direct control
  over the optimizer, augmentations, and target-network update rather than a wrapped
  trainer.
when_to_use_negative:
- You lack PyTorch training-loop experience and want an easy interface — use the `ByolTrainer`
  wrapper or the configurable training script instead
- The task requires labeled supervision rather than self-supervised representation
  learning
- Input is not image data amenable to augmentation-based contrastive-style pretraining
  (e.g. already-extracted feature tables)
edam_operation: http://edamontology.org/operation_3983
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0003
tools:
- name: astromorph
  role: Provides the `BYOL` class (PyTorch implementation of the BYOL framework),
    `ByolTrainer` wrapper, FitsFilelistDataset, and AstroMorphologyModel backbone
  repo: https://github.com/onsala-space-observatory/astromorph
- name: PyTorch
  role: Framework in which the BYOL class is implemented as a subclass of `torch.nn.Module`
    and used like any other PyTorch module
- name: Python 3.12
  role: Runtime environment for implementing and running the BYOL module
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557638
    title: 'astromorph: self-supervised machine learning pipeline for astronomical
      morphology analysis'
  source_content_hash: sha256:7ce76fec19211ecc87967cad4b63bd6d1fde678c8ec2521133d3ba399b2f3980
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/onsala-space-observatory/astromorph
  tools:
  - PyTorch
  - Python
  - Python 3.12
  - astromorph
  edam_operation: http://edamontology.org/operation_3983
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pytorch-neural-network-module-implementation

## Summary

Implements the Bootstrap Your Own Latents (BYOL) self-supervised framework as a `BYOL` class subclassing `torch.nn.Module`, tailored to astronomical imaging in the astromorph package, giving PyTorch users full flexibility over the training loop and BYOL components.

## When to use

Use when you need a customizable PyTorch module for self-supervised BYOL training on unlabeled astronomical images (e.g. FITS files) and want direct control over the optimizer, augmentations, and target-network update rather than a wrapped trainer.

## When NOT to use

- You lack PyTorch training-loop experience and want an easy interface — use the `ByolTrainer` wrapper or the configurable training script instead
- The task requires labeled supervision rather than self-supervised representation learning
- Input is not image data amenable to augmentation-based contrastive-style pretraining (e.g. already-extracted feature tables)

## Inputs

- Unlabeled astronomical images as FITS files (via FitsFilelistDataset / PyTorch DataLoader)
- A backbone network (e.g. AstroMorphologyModel or ResNet18 truncated at last_layer)
- BYOL settings: representation_size, projection_size, projection_hidden_size, use_momentum

## Outputs

- A trained BYOL module producing image embeddings of dimensionality representation_size (default 128)
- An updated target network after each call to update_moving_average()

## How to apply

Subclass `torch.nn.Module` and wrap a configurable backbone (e.g. the package's `AstroMorphologyModel` or a reduced ResNet18 truncated at a chosen `last_layer` such as 'layer2') as an online encoder with a projector MLP configured via `representation_size` (default 128), `projection_size` (16), `projection_hidden_size` (512), and a `use_momentum` flag making the target network the exponential moving average of the online network. Feed two independently augmented views of each image through the online and target branches and train with the symmetric augmentation-based BYOL loss, which needs no negative pairs. After every optimization step, call `update_moving_average()` to refresh the target network, as required by the BYOL procedure (arXiv:2006.07733). Validate with a smoke test that instantiates `BYOL`, runs a forward pass, and calls `update_moving_average()` without error; this class requires prior experience setting up PyTorch training routines, otherwise prefer `ByolTrainer`.

## Related tools

- **astromorph** (Provides the `BYOL` class (PyTorch implementation of the BYOL framework), `ByolTrainer` wrapper, FitsFilelistDataset, and AstroMorphologyModel backbone) — https://github.com/onsala-space-observatory/astromorph
- **PyTorch** (Framework in which the BYOL class is implemented as a subclass of `torch.nn.Module` and used like any other PyTorch module)
- **Python 3.12** (Runtime environment for implementing and running the BYOL module)

## Examples

```
from astromorph import BYOL; model = BYOL(AstroMorphologyModel(), representation_size=128); loss = model(view1, view2); loss.backward(); optimizer.step(); model.update_moving_average()
```

## Evaluation signals

- Smoke test: instantiate `BYOL`, run a forward pass on augmented views, and call `update_moving_average()` without error
- Confirm `update_moving_average()` is called after every optimization step during training
- Check that default BYOL component dimensions match configuration: representation_size=128, projection_size=16, projection_hidden_size=512, use_momentum=true
- Verify the target network parameters equal an exponential moving average of the online network parameters
- Downstream check: resulting embeddings should support clustering, anomaly detection, and similarity-based exploration of astronomical images

## Limitations

- Using the raw `BYOL` class 'provides the most flexibility, but requires some experience with setting up a training routine in PyTorch'
- Forgetting to call `update_moving_average()` after every optimization step breaks the BYOL procedure (target network must be the exponential moving average of the online network)
- Python 3.12 is the developed-for version; anything below Python 3.9 will definitely not work

## Evidence

- [methods] BYOL class as a PyTorch implementation: "the `BYOL` class as a PyTorch implementation of the BYOL framework;"
- [methods] update_moving_average after every step: "**NB: remember to call the `update_moving_average()` method after every optimization step.**"
- [readme] BYOL class is a nn.Module subclass: "The `BYOL` class is a subclass of `pytorch.nn.Module`, and can therefore be used like any other PyTorch module."
- [readme] BYOL class requires PyTorch experience: "The `BYOL` class provides the most flexibility, but requires some experience with setting up a training routine in PyTorch."
- [abstract] deeper customization via PyTorch classes: "offers streamlined pipeline scripts for ease of use as well as deeper customization options via PyTorch-based classes"
