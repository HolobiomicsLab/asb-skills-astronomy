---
name: stellar-spectrum-normalisation
description: Use when headerless two-column stellar spectra (wavelengths in angstroms,
  flux) need continuum normalisation without supervision, or when automating pseudo-continuum
  prediction across a batch of O–G star spectra.
when_to_use_negative:
- Spectra with headers or non-conforming column order — the tool expects strictly
  headerless two-column (wavelengths in angstroms, flux) files
- Objects dominated by wide emission lines when using default weights — switch to
  --weights emission rather than relying on the default 'active' set
- When a truly continuum-normalised reference already exists and only arithmetic rescaling
  is needed — the network pseudo-continuum prediction and spline post-processing are
  unnecessary overhead
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3570
tools:
- name: SUPPNet
  role: Fully convolutional network predicting the pseudo-continuum; SUPPNET.sh provides
    interactive, quiet batch, and --path correction modes
  repo: https://github.com/RozanskiT/suppnet.git
- name: conda
  role: Creates and activates the 'suppnet-env' environment holding all dependencies
    from environment.yml
- name: anaconda
  role: Python distribution prerequisite for installing the suppnet environment
- name: pip
  role: Installs suppnet as a regular Python module ('pip install -e .') for notebook-based
    usage
- name: Python
  role: Runtime for the suppnet module and the SUPPNET.sh entry script
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202141480
    title: 'SUPPNet: Neural network for stellar spectrum normalisation'
  source_content_hash: sha256:a5f7ba86d22a62d5ed2eaf44ed84288636c12d546598196877867d759c4734d3
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
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3570
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# stellar-spectrum-normalisation

## Summary

Normalise order-merged stellar spectra (O–G type) with SUPPNet, a fully convolutional neural network that predicts a pseudo-continuum followed by smoothing-spline post-processing with regressed knots for optional manual corrections. Achieves RMS ≈ 0.0128 (3900–7000 Å) and 0.0081 (4200–7000 Å) on test stars, comparable to careful manual normalisation.

## When to use

Use when headerless two-column stellar spectra (wavelengths in angstroms, flux) need continuum normalisation without supervision, or when automating pseudo-continuum prediction across a batch of O–G star spectra.

## When NOT to use

- Spectra with headers or non-conforming column order — the tool expects strictly headerless two-column (wavelengths in angstroms, flux) files
- Objects dominated by wide emission lines when using default weights — switch to --weights emission rather than relying on the default 'active' set
- When a truly continuum-normalised reference already exists and only arithmetic rescaling is needed — the network pseudo-continuum prediction and spline post-processing are unnecessary overhead

## Inputs

- Headerless two-column text spectra: column 1 = wavelengths in angstroms, column 2 = flux (nm allowed with --sampling 0.005)
- Optionally, a path_to_processing_results.all file for manual inspection/correction of a previously normalised spectrum
- Reference continua (for RMS-error validation against the paper's benchmarks)

## Outputs

- Normalised spectrum (flux divided by the spline-fitted pseudo-continuum), headerless two-column format
- Processing results file (path_to_processing_results.all) containing the spline-normalised spectrum and regressed spline knots
- Per-spectrum and window-averaged RMS error values over 3900–7000 Å and 4200–7000 Å

## How to apply

Install SUPPNet from github.com/RozanskiT/suppnet inside the conda environment 'suppnet-env' created from environment.yml, optionally installing suppnet as a Python module with 'pip install -e .' and linking SUPPNET.sh into ~/bin as 'SUPPNET'. Ensure inputs are headerless two-column files with wavelengths in angstroms in column 1 and flux in column 2 (use --sampling 0.005 if wavelengths are in nanometers). Run 'SUPPNET --quiet --sampling 0.05 --weights active' for unsupervised batch normalisation, letting the fully convolutional SUPP Network predict the pseudo-continuum; the active weights use an active-learning scheme to counter biases from training on synthetic spectra. A smoothing-spline post-processing stage is then fitted to the predicted pseudo-continuum, and flux is divided by it to yield the normalised spectrum; the regressed spline knots can be reloaded via 'SUPPNET --path path_to_processing_results.all' for interactive manual correction. Quality is judged by RMS error against reference continua: 0.0128 over 3900–7000 Å and 0.0081 over 4200–7000 Å for high-resolution O–G star spectra, and ~0.0050 on synthetic spectra.

## Related tools

- **SUPPNet** (Fully convolutional network predicting the pseudo-continuum; SUPPNET.sh provides interactive, quiet batch, and --path correction modes) — https://github.com/RozanskiT/suppnet.git
- **conda** (Creates and activates the 'suppnet-env' environment holding all dependencies from environment.yml)
- **anaconda** (Python distribution prerequisite for installing the suppnet environment)
- **pip** (Installs suppnet as a regular Python module ('pip install -e .') for notebook-based usage)
- **Python** (Runtime for the suppnet module and the SUPPNET.sh entry script)

## Examples

```
SUPPNET --quiet --sampling 0.05 --weights active path_to_spec_1.txt
```

## Evaluation signals

- Computed RMS over the test stars must equal 0.0128 in 3900–7000 Å and 0.0081 in 4200–7000 Å; flag any discrepancy
- RMS on synthetic spectra must be of the order of 0.0050
- Output normalised spectrum exists as a headerless two-column file with normalised flux hovering around 1 in continuum regions
- processing_results.all reloads successfully through 'SUPPNET --path' with regressed knots available for manual correction
- Symbolic link check 'ls -l ~/bin/SUPPNET' shows SUPPNET.sh correctly linked and bare 'SUPPNET' runs to completion

## Limitations

- The 'synth' weight set is trained only on synthetic spectra and shouldn't be used; prefer 'active' weights to avoid synthetic-training biases (active learning applied for this reason)
- If wavelengths are in nanometers the --sampling value must be changed from the default 0.05 to 0.005, otherwise resampling is wrong
- 'emission' weights should be used for objects showing wide emission lines; default 'active' weights may underperform on such targets

## Evidence

- [abstract] RMS error on test stars: "gives a root mean squared (RMS) error over the set of test stars equal to 0.0128 in the spectral range from 3900 Å to 7000 Å and 0.0081 in the range from 4200 Å to 7000 Å"
- [abstract] Network core: "The core of the proposed method uses the novel, fully convolutional deep neural network (SUPP Network) that was trained to predict a pseudo-continuum"
- [abstract] Post-processing splines: "The post-processing step uses smoothing splines that give access to regressed knots, which are useful for optional manual corrections"
- [readme] Input format (readme): "Spectra that you are working with should't have header: the forst column should contain wavelengths in angstroms (nanometers possible but then you need to change the sampling value from default 0.05"
- [abstract] Active learning: "The active learning technique was applied to deal with possible biases that may arise from training with synthetic spectra"
