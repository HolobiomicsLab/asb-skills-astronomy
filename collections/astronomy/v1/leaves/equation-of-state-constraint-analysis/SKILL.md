---
name: equation-of-state-constraint-analysis
description: Use when a binary-neutron-star event has gravitational-wave data plus
  kilonova photometry and/or a GRB afterglow light curve, and the goal is a joint
  EOS/neutron-star-radius posterior rather than per-messenger analyses combined a
  posteriori.
when_to_use_negative:
- The event has only gravitational-wave data with no kilonova photometry or GRB afterglow
  — the joint analysis is designed to exploit the multi-messenger nature of compact-binary
  mergers, and the workflow steps (lightcurve-analysis, jet-model GRB fit) would be
  inapplicable.
- You want the GRB jet energy E0 itself to constrain the EOS — the article states
  no significant constraint is imposed by E0 because the ratio ξ between E0 and the
  disk mass mdisk is treated as a free parameter; the EOS constraint instead flows
  via inclination and kilonova ejecta masses.
- The target is a parameter-estimation run for a new astrophysical transient model
  rather than reusing the existing NMMA inference infrastructure — the article notes
  reliable interpretation requires advancing the models themselves, not just the parameter-estimation
  machinery.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: NMMA
  role: Hosts the entry points nmma-generation, lightcurve-analysis, and nmma-analysis
    used to build the GW likelihood, fit the kilonova light curve, and run the simultaneous
    GW+kilonova+GRB joint inference with EOS priors.
  repo: github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Inference backend used by NMMA, e.g. backing the lightcurve-analysis entry
    point for AT2017gfo photometry.
- name: dynesty
  role: Nested sampler used to sample the joint GW+kilonova+GRB posterior from which
    the R_1.4 distribution is extracted.
  repo: https://zenodo.org/records/17268284
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:e02cf61ff6ca44e49463d81af96f9e48f526ab9f33fbc00c4c2f09b0e209b272
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# equation-of-state-constraint-analysis

## Summary

Reproduces the simultaneous multi-messenger inference of GW170817, kilonova AT2017gfo, and GRB170817A afterglow data within the NMMA framework, using chiral-EFT equation-of-state prior sets and the dynesty nested sampler, to constrain neutron-star matter (R of a 1.4 M⊙ star = 11.98 +0.35/-0.40 km).

## When to use

Use when a binary-neutron-star event has gravitational-wave data plus kilonova photometry and/or a GRB afterglow light curve, and the goal is a joint EOS/neutron-star-radius posterior rather than per-messenger analyses combined a posteriori.

## When NOT to use

- The event has only gravitational-wave data with no kilonova photometry or GRB afterglow — the joint analysis is designed to exploit the multi-messenger nature of compact-binary mergers, and the workflow steps (lightcurve-analysis, jet-model GRB fit) would be inapplicable.
- You want the GRB jet energy E0 itself to constrain the EOS — the article states no significant constraint is imposed by E0 because the ratio ξ between E0 and the disk mass mdisk is treated as a free parameter; the EOS constraint instead flows via inclination and kilonova ejecta masses.
- The target is a parameter-estimation run for a new astrophysical transient model rather than reusing the existing NMMA inference infrastructure — the article notes reliable interpretation requires advancing the models themselves, not just the parameter-estimation machinery.

## Inputs

- Gravitational-wave likelihood products for GW170817 (generated via nmma-generation)
- AT2017gfo kilonova photometry (multi-band light-curve data points)
- GRB170817A afterglow light-curve data set (Troja et al.)
- Chiral-EFT-based EOS prior sets (optionally with radio/X-ray measurements of isolated neutron stars)
- Synthetic Gaussian jet-model light curve for the GRB afterglow

## Outputs

- Joint posterior samples of GW/kilonova/GRB parameters (incl. masses, inclination, ejecta masses, EOS parameters)
- Posterior distribution of the radius of a 1.4 M⊙ neutron star (target: R = 11.98 +0.35/-0.40 km, 90% credible interval)
- Fitted kilonova AT2017gfo and GRB170817A afterglow light curves

## How to apply

First run the `nmma-generation` entry point to build the gravitational-wave likelihood products for the GW170817 signal, then run `lightcurve-analysis` (backed by bilby) on the AT2017gfo photometry for the kilonova model. Analyze the GRB170817A afterglow data set (the Troja et al. collection) with the synthetic Gaussian jet-model light curve. Execute the joint GW + kilonova + GRB analysis with `nmma-analysis`, incorporating chiral-EFT-based EOS prior sets (and radio/X-ray constraints on isolated neutron stars where available), and sample the joint posterior with the dynesty nested sampler. Extract the posterior distribution of the radius of a 1.4 M⊙ neutron star from the joint samples and check that both the AT2017gfo kilonova and GRB170817A afterglow light curves are well described by the fitted model. The EOS constraint enters through the joint posterior: the GRB's inclination measurement imposes a constraint on the EOS via the kilonova ejecta mass measurements.

## Related tools

- **NMMA** (Hosts the entry points nmma-generation, lightcurve-analysis, and nmma-analysis used to build the GW likelihood, fit the kilonova light curve, and run the simultaneous GW+kilonova+GRB joint inference with EOS priors.) — github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Inference backend used by NMMA, e.g. backing the lightcurve-analysis entry point for AT2017gfo photometry.)
- **dynesty** (Nested sampler used to sample the joint GW+kilonova+GRB posterior from which the R_1.4 distribution is extracted.) — https://zenodo.org/records/17268284

## Examples

```
nmma-generation && nmma-analysis --sampler dynesty
```

## Evaluation signals

- The recovered 90% credible interval for the radius of a 1.4 M⊙ neutron star reproduces R = 11.98 (+0.35/-0.40) km from the joint posterior samples.
- Both the kilonova AT2017gfo and the GRB afterglow GRB170817A light curves are well described by the fitted models (visual/statistical light-curve fit check).
- Adding GRB170817A information to the GW170817 + AT2017gfo analysis yields a minimal yet noticeable improvement in parameter uncertainties, driven by the GRB inclination constraint on the EOS.
- The EOS constraint is traceable to the kilonova ejecta mass measurements via the GRB inclination measurement, not to the free-parameter jet energy E0/ξ relation.
- The posterior sampling completed with the dynesty nested sampler over the chiral-EFT EOS prior sets without evidence of non-convergence.

## Limitations

- Additional uncertainties arise from limited physics input in the kilonova models and in the semi-analytic GRB models.
- No significant EOS constraint is imposed by the jet energy E0, since the ratio ξ between E0 and the disk mass mdisk is a free parameter.
- The improvement in parameter uncertainties from adding GRB170817A is minimal (though noticeable) compared with the GW + kilonova analysis alone.

## Evidence

- [abstract] Target R_1.4 from the joint analysis: [evidence span withheld — anchor status: not_found]
- [discussion] Simultaneous GW + EM analysis capability of the framework: "This framework allows for the simultaneous analysis of GW and EM signals such as kilonovae and GRB afterglows."
- [results] GRB afterglow analyzed with the Gaussian jet model: "The GRB afterglow light-curve data are analyzed with the synthetic Gaussian jet-model lightcurve described before"
- [results] Light-curve fits are validated: "We find that both the kilonova AT2017gfo and the GRB afterglow GRB170817A are well described in our analysis."
- [methods] bilby is the inference backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [readme] NMMA entry points used in the workflow: "When using this code for a publication, kindly make a reference to the package by its name, NMMA"
