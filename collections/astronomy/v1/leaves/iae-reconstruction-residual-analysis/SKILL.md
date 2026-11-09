---
name: iae-reconstruction-residual-analysis
description: Use when trained IAE spectral models are available and you need to verify
  their generative reconstruction quality on held-out spectra before using them as
  surrogates in SUSHI unmixing.
when_to_use_negative:
- When the IAE networks have not yet been trained — the README tutorial 'Training
  an IAE.ipynb' must be run first, and the paper notes no trained weights are archived,
  only code
- When you need physical parameters from unmixed spectra — SUSHI's output is fitted
  unmixed spectra and does not directly provide physical parameters; this skill only
  measures surrogate reconstruction error
- When spectra come from a different physical model than the networks were trained
  on — residuals against Table B.1 are only meaningful for the paper's five trained
  networks and their validation splits
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: SUSHI
  role: Provides the trained IAE models, tutorial notebooks, and Output dictionary
    structure (Psi, X_train, X_valid) used for the residual analysis
  repo: https://github.com/jmlascar/SUSHI
- name: IAE (Interpolatory Auto-Encoder)
  role: The generative spectral surrogate being evaluated; encoder E, linear interpolator
    I, and decoder D produce the reconstructions whose residuals are measured
  repo: https://github.com/jbobin/IAE
- name: JAX
  role: Package in which the IAE models were coded; used to run the forward pass through
    E, I, and D
- name: Python 3.10
  role: Runtime environment for the SUSHI/IAE code, installed via the SUSHI_env.yml
    conda environment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:bd1dfadbc8fb5fa581e138ea33e6f77ecac031cb31b22f6c4f3ce38b53cda7ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - IAE (Interpolatory Auto-Encoder)
  - JAX
  - Python
  - Python 3.10
  - SUSHI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# iae-reconstruction-residual-analysis

## Summary

Quantifies the held-out reconstruction accuracy of trained Interpolatory Auto-Encoder (IAE) spectral surrogate models by computing mean element-wise residuals (s_hat - s)/s on the 30% validation split and benchmarking against Table B.1 of the SUSHI paper.

## When to use

Use when trained IAE spectral models are available and you need to verify their generative reconstruction quality on held-out spectra before using them as surrogates in SUSHI unmixing.

## When NOT to use

- When the IAE networks have not yet been trained — the README tutorial 'Training an IAE.ipynb' must be run first, and the paper notes no trained weights are archived, only code
- When you need physical parameters from unmixed spectra — SUSHI's output is fitted unmixed spectra and does not directly provide physical parameters; this skill only measures surrogate reconstruction error
- When spectra come from a different physical model than the networks were trained on — residuals against Table B.1 are only meaningful for the paper's five trained networks and their validation splits

## Inputs

- Five readily trained IAE model files (one per physical component) from the SUSHI repository's 'IAE models' directory
- Held-out testing spectra per network: Output['X_valid'], the 30% validation split of the shuffled training set
- Anchor points Output['Psi'] (boundary-parameter spectra) required by the IAE interpolator
- Table B.1 reference residuals: -0.024%, 0.040%, 0.041%, -0.035%, -0.052%

## Outputs

- iae_reconstruction_residuals.csv: one average residual per IAE network
- Per-network mean reconstruction errors for comparison with Table B.1

## How to apply

Load each of the five readily trained IAE models (one per physical component) from the SUSHI repository's 'IAE models' directory together with its held-out testing spectra, which are Output['X_valid'] — the 30% validation split of the shuffled training set described in 'Training an IAE.ipynb'. Forward-pass every testing spectrum through the encoder E, project onto the anchor points' barycentric span via the linear interpolator I in the code domain, and decode with D to obtain reconstructed spectra. Compute the element-wise relative residual R = (s_hat_i - s_i)/s_i for each spectrum, then average residuals across each network's testing set to yield one mean reconstruction error per IAE. Compare the five averages against the Table B.1 reference values (-0.024%, 0.040%, 0.041%, -0.035%, -0.052%); each should match within a small tolerance and be well under 0.1% in magnitude. Save the results as iae_reconstruction_residuals.csv.

## Related tools

- **SUSHI** (Provides the trained IAE models, tutorial notebooks, and Output dictionary structure (Psi, X_train, X_valid) used for the residual analysis) — https://github.com/jmlascar/SUSHI
- **IAE (Interpolatory Auto-Encoder)** (The generative spectral surrogate being evaluated; encoder E, linear interpolator I, and decoder D produce the reconstructions whose residuals are measured) — https://github.com/jbobin/IAE
- **JAX** (Package in which the IAE models were coded; used to run the forward pass through E, I, and D)
- **Python 3.10** (Runtime environment for the SUSHI/IAE code, installed via the SUSHI_env.yml conda environment)

## Examples

```
conda env create -f SUSHI_env.yml; conda activate SUSHI_env  # then in Jupyter: load 'IAE models', forward-pass Output['X_valid'] through E, I, D and compute mean((s_hat - s)/s)
```

## Evaluation signals

- Each network's computed average residual matches its Table B.1 reference value (-0.024%, 0.040%, 0.041%, -0.035%, -0.052%) within a small tolerance
- All five mean residuals have magnitude well under 0.1%
- Residuals are computed element-wise as R = (s_hat_i - s_i)/s_i on Output['X_valid'], not on training spectra
- The output CSV iae_reconstruction_residuals.csv contains exactly five values, one per physical-component network

## Limitations

- Trained IAE networks/weights are not stated as shared in the paper — only code availability is given, so the five models must come from the repository's 'IAE models' directory or be retrained
- No random seeds or training hyperparameter randomness handling is described for IAE training, so retrained networks may not exactly reproduce Table B.1
- Residuals reflect reconstruction quality only on the 30% validation split of each network's own training set, not generalization to unseen observing conditions

## Evidence

- [abstract] For the spectral model, we used an Interpolatory Auto-Encoder, a generative model: "For the spectral model, we used an Interpolatory Auto-Encoder, a generative model"
- [readme] validation split definition: "Shuffle the rest of the training set. Separate it into a training (70%) and validation (30%) set. Output["X_train"] and Output["X_valid"]."
- [readme] trained models availability: "* *IAE models:* readily trained IAE spectral models for testing SUSHI."
- [discussion] IAE is trained per component: "An IAE model may be trained for each physical component. The code to do so is openly accessible on Git Hub"
- [methods] decoder speed rationale: "a call to D was over 60 times faster than calling the physical model based on look-up tables (XSpec model in Sherpa"
