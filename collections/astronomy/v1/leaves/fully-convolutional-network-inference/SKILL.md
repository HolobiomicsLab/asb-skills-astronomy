---
name: fully-convolutional-network-inference
description: Use when a headerless two-column stellar spectrum (wavelengths in angstroms,
  flux in the second column) needs a pseudo-continuum prediction from the SUPP Network
  before spline-based normalisation or manual correction.
when_to_use_negative:
- 'The goal is manual inspection/correction of an already-normalised spectrum with
  SUPPNET --path: SUPPNet will not be loaded, so no network inference occurs'
- 'Weights choice matters beyond the default: ''synth'' weights are trained only on
  synthetic spectra and shouldn''t be used for observed spectra'
- The spectrum has a header or a non-standard column layout — the network expects
  headerless two-column wavelength/flux input
edam_operation: http://edamontology.org/operation_2945
edam_topics: []
tools:
- name: SUPPNet
  role: Provides the fully convolutional neural network (SUPP Network) trained to
    predict a pseudo-continuum; the FCN forward pass is the core of the pipeline
  repo: https://github.com/RozanskiT/suppnet.git
- name: conda
  role: Creates and activates the suppnet-env environment that handles all dependencies
    for network inference
- name: pip
  role: Installs suppnet as a regular Python module ('pip install -e .') so the packaged
    FCN weights and inference code are accessible
- name: anaconda
  role: Python distribution prerequisite for running the network
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202141480
    title: 'SUPPNet: Neural network for stellar spectrum normalisation'
  source_content_hash: sha256:f84a5d550a9db0bff964346436887883504db1a91b2251c8d4cec3df782441be
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# fully-convolutional-network-inference

## Summary

Runs the SUPPNet fully convolutional neural network (FCN) forward pass on a resampled stellar spectrum to predict a pseudo-continuum curve, prior to any smoothing-spline post-processing. This network prediction is the core of SUPPNet's spectrum normalisation method.

## When to use

Use when a headerless two-column stellar spectrum (wavelengths in angstroms, flux in the second column) needs a pseudo-continuum prediction from the SUPP Network before spline-based normalisation or manual correction.

## When NOT to use

- The goal is manual inspection/correction of an already-normalised spectrum with SUPPNET --path: SUPPNet will not be loaded, so no network inference occurs
- Weights choice matters beyond the default: 'synth' weights are trained only on synthetic spectra and shouldn't be used for observed spectra
- The spectrum has a header or a non-standard column layout — the network expects headerless two-column wavelength/flux input

## Inputs

- Headerless two-column spectrum file: first column wavelengths in angstroms, second column flux (wavelengths in nanometers allowed if --sampling is set to 0.005)

## Outputs

- Predicted pseudo-continuum curve from the SUPP Network forward pass (raw network output, before smoothing-spline post-processing)

## How to apply

Activate the suppnet-env conda environment and install suppnet as a Python module with 'pip install -e .' so the packaged FCN weights and inference code are accessible. Verify the input spectrum is headerless and two-column, with wavelengths in angstroms in column 1 and flux in column 2 (if wavelengths are in nanometers, change the sampling value from the default 0.05 to 0.005). Resample the spectrum onto the network's prediction grid with the --sampling option (default 0.05 Å), then run the forward pass with the default 'active' weight set (--weights active; use 'emission' only for objects with wide emission lines, and never 'synth'). Capture the raw network continuum output — this is the pseudo-continuum prediction, and it comes before the smoothing-spline post-processing stage that provides regressed knots for optional manual corrections.

## Related tools

- **SUPPNet** (Provides the fully convolutional neural network (SUPP Network) trained to predict a pseudo-continuum; the FCN forward pass is the core of the pipeline) — https://github.com/RozanskiT/suppnet.git
- **conda** (Creates and activates the suppnet-env environment that handles all dependencies for network inference)
- **pip** (Installs suppnet as a regular Python module ('pip install -e .') so the packaged FCN weights and inference code are accessible)
- **anaconda** (Python distribution prerequisite for running the network)

## Examples

```
SUPPNET --quiet --sampling 0.05 --weights active path_to_spec_1.txt
```

## Evaluation signals

- Script/SUPPNET run completes on the input spectrum without errors
- Normalisation RMS of the predicted continuum falls within the paper's reported range for manual-quality normalisation: RMS ≈ 0.0128 in 3900–7000 Å and 0.0081 in 4200–7000 Å for high-resolution O–G type stars
- Predicted continuum is a smooth curve over the resampled grid at the chosen sampling step (0.05 Å for angstrom wavelengths, 0.005 for nanometers)
- Output is the raw network continuum, i.e. prior to smoothing-spline post-processing (no regressed-knot corrections applied)
- Default 'active' weights were used (not 'synth', which is trained only on synthetic spectra and shouldn't be used)

## Limitations

- Input spectra must have no header and exactly two columns (wavelengths in angstroms or nm, flux); nanometer wavelengths require changing --sampling from 0.05 to 0.005
- The 'synth' weights set was trained only on synthetic spectra and shouldn't be used; 'emission' weights should be reserved for objects showing wide emission lines
- The network prediction is only the first stage: optional manual corrections rely on the downstream smoothing-spline post-processing, not on the FCN output itself

## Evidence

- [abstract] FCN predicts pseudo-continuum before spline post-processing: "The core of the proposed method uses the novel, fully convolutional deep neural network (SUPP Network) that was trained to predict a pseudo-continuum"
- [abstract] Post-processing is a separate later stage: "The post-processing step uses smoothing splines that give access to regressed knots, which are useful for optional manual corrections"
- [readme] Sampling option for the network's resampling: "--sampling`, default=0.05, sampling option enable user to adjust the resampling that the neural network is using for a pseudo-continuum prediction"
- [readme] Input format and nm caveat: "Spectra that you are working with should't have header: the forst column should contain wavelengths in angstroms (nanometers possible but then you need to change the sampling value from default 0.05"
- [readme] Weights selection guidance: "__active__ is a default one, __emission__ should be used for objects that show wide emission lines, __synth__ is a weights trained only on synthetic spectra and shouldn't be used."
- [readme] Install suppnet as Python module: [evidence span withheld — anchor status: not_found]
