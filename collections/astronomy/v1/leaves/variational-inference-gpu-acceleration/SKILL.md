---
name: variational-inference-gpu-acceleration
description: Use when a Bayesian spectral fit with MCMC/NUTS is too slow for high-resolution
  data (many channels, e.g. microcalorimeter spectra), a GPU is available, and approximate
  posteriors within ~10 minutes wall-clock are acceptable.
when_to_use_negative:
- When exact, fully-sampled posteriors are required — VI is approximate; jaxspec's
  no U-turn sampler (MCMCFitter.fit with num_chains/num_warmup/num_samples) is the
  alternative for accurate posteriors
- When the parameter estimates must be unbiased at low count rates under chi-square-style
  fitting — jaxspec's guidance is to fit under a Poisson likelihood (C-stat) regardless,
  and VI adds approximation error on top
- When no GPU is available and the dataset is small enough that CPU MCMC finishes
  quickly — the <10-minute GPU advantage does not apply
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3399
tools:
- name: jaxspec
  role: Loads the OGIP-standard spectrum (ObsConfiguration.from_pha_file with low_energy/high_energy
    band limits), builds the composite spectral model, and exposes differentiable
    likelihoods for VI fitting
  repo: https://github.com/renecotyfanboy/jaxspec
- name: JAX
  role: Provides just-in-time compilation and automatic differentiation so the likelihood
    function is compilable on GPU
  repo: https://github.com/renecotyfanboy/jaxspec
- name: numpyro
  role: Supplies the prior distributions (dist.Uniform, dist.LogUniform), the variational
    inference solver, and the accelerator configuration calls numpyro.set_platform('gpu')
    / numpyro.enable_x64()
  repo: https://github.com/renecotyfanboy/jaxspec
- name: Python 3.12
  role: Runtime environment; jaxspec is installed with pip into a clean Python 3.11/3.12
    environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:661e0a73ca47df379af64f45182fb1071d363e9cec5c8f4d0c0c2bd2ea4b7524
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - Python 3.12
  - jaxspec
  - numpyro
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# variational-inference-gpu-acceleration

## Summary

Solve Bayesian X-ray spectral inference with variational inference (VI) on a GPU-compiled, differentiable likelihood built from jaxspec/JAX/numpyro, replacing slow MCMC sampling when high-resolution data (e.g. Hitomi microcalorimeter spectra) make NUTS too expensive. The paper shows VI produces convincing results on the Hitomi Perseus cluster observation in under 10 minutes on a GPU.

## When to use

Use when a Bayesian spectral fit with MCMC/NUTS is too slow for high-resolution data (many channels, e.g. microcalorimeter spectra), a GPU is available, and approximate posteriors within ~10 minutes wall-clock are acceptable.

## When NOT to use

- When exact, fully-sampled posteriors are required — VI is approximate; jaxspec's no U-turn sampler (MCMCFitter.fit with num_chains/num_warmup/num_samples) is the alternative for accurate posteriors
- When the parameter estimates must be unbiased at low count rates under chi-square-style fitting — jaxspec's guidance is to fit under a Poisson likelihood (C-stat) regardless, and VI adds approximation error on top
- When no GPU is available and the dataset is small enough that CPU MCMC finishes quickly — the <10-minute GPU advantage does not apply

## Inputs

- X-ray spectrum in the OGIP standard (PHA file) with linked ARF/RMF response files (e.g. Hitomi Perseus cluster observation)
- Composite spectral model definition from jaxspec's implemented additive/multiplicative components
- Prior specification dict using numpyro distributions (dist.Uniform, dist.LogUniform), with calibrated parameters fixed as constants
- CUDA-capable GPU configured before JAX initialisation

## Outputs

- Posterior parameter estimates (variational posterior) for the spectral model components
- Wall-clock inference runtime (validated to be < 10 minutes on GPU)
- FitResult object serialised with dill (e.g. result.pickle), plottable via result.plot_corner() and result.plot_ppc()

## How to apply

Build an ObsConfiguration from the PHA file with its ARF/RMF response and band-limit the fit range, then define a composite spectral model from jaxspec's additive/multiplicative components, attaching a prior dict of numpyro distributions (dist.Uniform, dist.LogUniform) and fixing calibrated parameters to constants. Before any JAX code runs, configure the accelerator with numpyro.set_platform('gpu') and numpyro.enable_x64() so the likelihood compiles for the GPU in double precision. Solve the inference problem with variational inference instead of the no U-turn sampler, then compare the fitted posterior parameters against published reference results (here, the Hitomi collaboration's Perseus cluster results) and record the wall-clock runtime against the <10-minute GPU threshold. Serialise the FitResult (e.g. with dill into result.pickle) for later inspection via plot_corner/plot_ppc.

## Related tools

- **jaxspec** (Loads the OGIP-standard spectrum (ObsConfiguration.from_pha_file with low_energy/high_energy band limits), builds the composite spectral model, and exposes differentiable likelihoods for VI fitting) — https://github.com/renecotyfanboy/jaxspec
- **JAX** (Provides just-in-time compilation and automatic differentiation so the likelihood function is compilable on GPU) — https://github.com/renecotyfanboy/jaxspec
- **numpyro** (Supplies the prior distributions (dist.Uniform, dist.LogUniform), the variational inference solver, and the accelerator configuration calls numpyro.set_platform('gpu') / numpyro.enable_x64()) — https://github.com/renecotyfanboy/jaxspec
- **Python 3.12** (Runtime environment; jaxspec is installed with pip into a clean Python 3.11/3.12 environment)

## Examples

```
conda create -n jaxspec python=3.12 && conda activate jaxspec && pip install jaxspec --upgrade
```

## Evaluation signals

- Fitted posterior parameters reproduce the Hitomi collaboration's published Perseus cluster results
- Wall-clock inference runtime is recorded and verified to be below the 10-minute GPU threshold
- GPU platform was configured before any JAX code ran (numpyro.set_platform('gpu')), verified by checking JAX devices after compilation
- 64-bit precision is active (numpyro.enable_x64()) so spectral likelihoods are not degraded by float32 arithmetic
- FitResult serialises cleanly with dill and diagnostic plots (plot_corner, plot_ppc) show converged, well-constrained posteriors

## Limitations

- Variational inference is approximate — the paper claims only 'convincing results', not exact posteriors; use NUTS when precision matters
- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities
- The software version number for jaxspec is not stated in the article (only Python 3.11/3.12 and XSPEC 12.15.1 are referenced), so reproducibility pins must be recorded manually
- VI speed depends on GPU availability; on CPU the compiled-likelihood advantage is reduced
- SubtractedBackground does not propagate dispersion, which is inadequate for a comprehensive error budget on the fitted parameters

## Evidence

- [abstract] Variational inference produces convincing results on high-resolution data in less than 10 minutes on a GPU: "we show that variational inference can produce convincing results even on high-resolution data in less than 10 minutes on a GPU."
- [abstract] jaxspec reproduces the Hitomi collaboration's Perseus cluster results using variational inference on a GPU: "reproducing the results of the Hitomi collaboration on the Perseus cluster, while solving the inference problem using variational inference on a GPU."
- [abstract] jaxspec's JAX-based differentiable likelihoods are GPU-compilable: "Based on the JAX ecosystem, jaxspec allows the generation of differentiable likelihood functions compilable on core or graphical process units (GPUs)"
- [methods] Band-limited observation configuration from a PHA file: "obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)"
- [methods] C-stat / Poisson likelihood required for unbiased parameter estimates: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [readme] Early-release caveat from the README: "**jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities"
