---
name: gravitational-wave-likelihood-generation
description: Use when a joint multi-messenger analysis (GW + kilonova + GRB afterglow)
  needs the gravitational-wave likelihood precomputed via NMMA before running nmma-analysis;
  triggered by GW strain data for events like GW170817.
when_to_use_negative:
- The analysis targets only the kilonova light curve (AT2017gfo photometry) or only
  the GRB afterglow light curve without any GW data — use lightcurve-analysis or the
  GRB Gaussian jet-model workflow instead
- The GW likelihood products for the event already exist and the task is to run the
  joint inference — proceed directly to nmma-analysis with dynesty
- The problem is EOS post-processing or light-curve simulation (create-svdmodel, combine-EOS,
  lightcurve_simulation tutorial) rather than GW likelihood construction
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3452
tools:
- name: NMMA
  role: Framework whose nmma-generation entry point produces the GW likelihood products;
    its nmma-analysis entry point then consumes them for the joint GW+kilonova+GRB
    posterior
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend for NMMA; the generated GW likelihood objects and downstream inference
    are bilby-compatible
  repo: https://arxiv.org/pdf/1811.02042
- name: dynesty
  role: Nested sampler used to sample the joint posterior built from the generated
    GW likelihood together with kilonova and GRB likelihoods
  repo: https://zenodo.org/records/17268284
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:d2f00d9a655e3dce068b86c403edbb25b06db6b42642c1e6fb17ff5e094605af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gravitational-wave-likelihood-generation

## Summary

Generate the gravitational-wave likelihood products (e.g., for GW170817) with the NMMA framework's nmma-generation entry point so that GW data can be jointly analyzed with kilonova and GRB afterglow data via bilby-backed Bayesian inference.

## When to use

Use when a joint multi-messenger analysis (GW + kilonova + GRB afterglow) needs the gravitational-wave likelihood precomputed via NMMA before running nmma-analysis; triggered by GW strain data for events like GW170817.

## When NOT to use

- The analysis targets only the kilonova light curve (AT2017gfo photometry) or only the GRB afterglow light curve without any GW data — use lightcurve-analysis or the GRB Gaussian jet-model workflow instead
- The GW likelihood products for the event already exist and the task is to run the joint inference — proceed directly to nmma-analysis with dynesty
- The problem is EOS post-processing or light-curve simulation (create-svdmodel, combine-EOS, lightcurve_simulation tutorial) rather than GW likelihood construction

## Inputs

- Gravitational-wave strain data for the event (e.g., GW170817) in a bilby-readable format
- NMMA configuration specifying the GW event, detector network, and prior/EOS settings (e.g., chiral-EFT-based EOS prior sets)
- NMMA installation (github.com/nuclear-multimessenger-astronomy/nmma)

## Outputs

- Gravitational-wave likelihood products consumable by nmma-analysis
- Bilby-compatible likelihood objects forming the GW term of the joint multi-messenger likelihood

## How to apply

Run NMMA's `nmma-generation` entry point to build the gravitational-wave likelihood products for the event of interest (e.g., GW170817 strain data), which are then consumed downstream by `nmma-analysis` together with kilonova (`lightcurve-analysis`) and GRB afterglow likelihoods. NMMA uses bilby as its backend, so the generated likelihood objects are bilby-compatible; the joint posterior over parameters such as the EOS-dependent R(1.4 M⊙) is sampled with the dynesty nested sampler. Rationale: precomputing GW likelihood products separates the expensive GW likelihood construction from the joint sampling stage, enabling the simultaneous GW+kilonova+GRB analysis that yields R_1.4 = 11.98 (+0.35/−0.40) km. Decision point: choose the sampler (dynesty, PyMultiNest, or UltraNest) and cite the corresponding companion paper; if the GW component is used, follow bilby's citation guidelines. Validate by feeding the generated products into the joint analysis and checking that the recovered 90% credible interval for R(1.4 M⊙) reproduces the published value.

## Related tools

- **NMMA** (Framework whose nmma-generation entry point produces the GW likelihood products; its nmma-analysis entry point then consumes them for the joint GW+kilonova+GRB posterior) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend for NMMA; the generated GW likelihood objects and downstream inference are bilby-compatible) — https://arxiv.org/pdf/1811.02042
- **dynesty** (Nested sampler used to sample the joint posterior built from the generated GW likelihood together with kilonova and GRB likelihoods) — https://zenodo.org/records/17268284

## Examples

```
nmma-generation --event GW170817 [then] nmma-analysis --sampler dynesty --eos-priors chiral-EFT  # entry-point names grounded in article; flags illustrative
```

## Evaluation signals

- The generated likelihood products, when used in nmma-analysis with chiral-EFT EOS priors and dynesty, reproduce the 90% credible interval R(1.4 M⊙) = 11.98 (+0.35/−0.40) km
- The likelihood objects load correctly into the bilby backend without format errors
- The joint analysis recovers parameter uncertainties such that adding GRB170817A information gives a 'minimal, yet noticeable' improvement over the GW+kilonova-only analysis, confirming the GW likelihood is not degenerate with the EM likelihoods
- Posterior samples of R(1.4 M⊙) are extractable from the joint posterior file produced by nmma-analysis

## Limitations

- Limited physics input in kilonova and semi-analytic GRB models adds additional uncertainties beyond the GW likelihood construction
- No version number or DOI pinning of the NMMA release is provided in the article's discussion, so reproducibility requires recording the installed NMMA commit/version yourself
- The jet energy E0 imposes no significant EOS constraint because the ratio ξ between E0 and the disk mass mdisk is a free parameter — GW likelihood products alone cannot constrain this degeneracy

## Evidence

- [methods] Entry points include nmma-generation for GW likelihood generation: "Entry points - `nmma-generation` - `nmma-analysis` - `lightcurve-analysis`"
- [methods] NMMA uses bilby as backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [abstract] Joint analysis result for R_1.4: [evidence span withheld — anchor status: not_found]
- [abstract] First simultaneous GW+kilonova+GRB analysis: "Here, we show an extension of the NMMA code as a first attempt of analyzing the gravitational-wave signal, the kilonova, and the gamma-ray burst afterglow simultaneously."
- [readme] README cites dynesty as sampler: "Based on the sampler used for the analysis, please also cite the associated papers: dynesty: Zenodo entry 10.5281/zenodo.3348367"
