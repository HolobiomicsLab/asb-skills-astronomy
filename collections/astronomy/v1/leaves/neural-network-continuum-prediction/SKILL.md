---
name: neural-network-continuum-prediction
description: Use when a headerless two-column spectrum (wavelengths in angstroms,
  flux) of an O–G type star needs an automated pseudo-continuum prediction before
  dividing out the continuum or applying spline corrections.
when_to_use_negative:
- Input wavelengths are in nanometers without changing --sampling from the default
  0.05 to 0.005
- Objects showing wide emission lines without switching to --weights emission
- Goal is manual inspection/correction of an already-normalised spectrum — use SUPPNET
  --path, which does not load SUPPNet
tools:
- name: SUPPNet
  role: Fully convolutional neural network (SUPP Network) that performs the pseudo-continuum
    prediction via forward pass with the 'active' weight set
  repo: https://github.com/RozanskiT/suppnet.git
- name: conda
  role: Provides the suppnet-env environment handling all dependencies
- name: pip
  role: Installs suppnet as a regular Python module ('pip install -e .') exposing
    packaged FCN weights and inference code
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202141480
    title: 'SUPPNet: Neural network for stellar spectrum normalisation'
  source_content_hash: sha256:de00fafb7dc12a094400da9762833d3da52447f02be82b496f82cf0292517463
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RozanskiT/suppnet.git
  tools:
  - Python
  - SUPPNet
  - anaconda
  - conda
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# neural-network-continuum-prediction

## Summary

Predict a pseudo-continuum for order-merged stellar spectra using SUPPNet, a fully convolutional neural network that is the core of the SUPPNet normalisation pipeline, prior to any smoothing-spline post-processing.

## When to use

Use when a headerless two-column spectrum (wavelengths in angstroms, flux) of an O–G type star needs an automated pseudo-continuum prediction before dividing out the continuum or applying spline corrections.

## When NOT to use

- Input wavelengths are in nanometers without changing --sampling from the default 0.05 to 0.005
- Objects showing wide emission lines without switching to --weights emission
- Goal is manual inspection/correction of an already-normalised spectrum — use SUPPNET --path, which does not load SUPPNet

## Inputs

- Headerless two-column stellar spectrum: first column wavelengths in angstroms, second column flux (0.005 sampling if nm)
- Order-merged high-resolution O–G type star spectrum

## Outputs

- Raw network pseudo-continuum prediction curve (e.g. pseudo_continuum.dat with wavelengths)
- Continuum to divide flux by prior to spline post-processing

## How to apply

Install suppnet as a Python module with 'pip install -e .' inside the suppnet-env conda environment so the packaged FCN weights and inference code are accessible. Verify the input is headerless and two-column (angstroms, flux); resample onto the network's prediction grid with --sampling 0.05 (use 0.005 if wavelengths are in nanometers). Run the forward pass with the default 'active' weights (--weights active; use 'emission' for wide emission lines) to obtain the raw pseudo-continuum curve. Extract this network output BEFORE the smoothing-spline post-processing stage, since splines with regressed knots are applied only afterwards and are reserved for optional manual corrections.

## Related tools

- **SUPPNet** (Fully convolutional neural network (SUPP Network) that performs the pseudo-continuum prediction via forward pass with the 'active' weight set) — https://github.com/RozanskiT/suppnet.git
- **conda** (Provides the suppnet-env environment handling all dependencies)
- **pip** (Installs suppnet as a regular Python module ('pip install -e .') exposing packaged FCN weights and inference code)

## Examples

```
conda activate suppnet-env && pip install -e . && SUPPNET --quiet --sampling 0.05 --weights active spectrum.txt
```

## Evaluation signals

- Script runs to completion on the input spectrum and writes the continuum together with input wavelengths
- Normalisation RMS of the predicted continuum falls within the reported range: ~0.0128 over 3900–7000 Å and 0.0081 over 4200–7000 Å for high-resolution O–G stars
- Output pseudo-continuum is extracted prior to the smoothing-spline post-processing stage
- Synthetic-spectrum benchmark RMS of the order of 0.0050 as an upper-bound sanity reference
- Result comparable to careful manual normalisation of the same spectrum

## Limitations

- The 'synth' weights are trained only on synthetic spectra and shouldn't be used
- Active learning was applied specifically to deal with biases arising from training with synthetic spectra; biases may remain in non-standard targets
- Default sampling (0.05) assumes wavelengths in angstroms; using it on nm-wavelength spectra requires changing to 0.005

## Evidence

- [abstract] FCN core predicts pseudo-continuum: "The core of the proposed method uses the novel, fully convolutional deep neural network (SUPP Network) that was trained to predict a pseudo-continuum"
- [abstract] splines only post-processing: "The post-processing step uses smoothing splines that give access to regressed knots, which are useful for optional manual corrections"
- [readme] input format: "Spectra that you are working with should't have header: the forst column should contain wavelengths in angstroms"
- [methods] module install: [evidence span withheld — anchor status: not_found]
