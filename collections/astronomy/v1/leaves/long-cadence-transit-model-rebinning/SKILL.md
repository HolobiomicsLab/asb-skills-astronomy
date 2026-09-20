---
name: long-cadence-transit-model-rebinning
description: Use when fitting transit light curves with long integration times (e.g.
  Kepler 29.4-minute long-cadence data), where unbinned model evaluation would distort
  fitted parameters such as Rp/Rs and a/Rs.
when_to_use_negative:
- Short-cadence or ground-based photometry where integration smearing is negligible
  — the finer-scale model computation and rebinning add cost without benefit for well-sampled
  light curves
- Analyses that require phase folding or rebinning multiple light curves into a single
  fold — TAP is explicitly designed to simultaneously analyze multiple transits of
  varying cadence without phase folding and rebinning
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: Transit Analysis Package (TAP)
  role: IDL GUI that computes transit models on a finer temporal scale and integrates
    them to the observed long cadence during MCMC fitting
  repo: https://github.com/zgazak/TAP
- name: IDL
  role: Language environment in which TAP runs the rebinning workflow
- name: EXOFAST
  role: Provides the Mandel & Agol (2002) analytic transit model evaluated at the
    finer temporal scale before rebinning
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:981485af94d662266c7f2b9b9b7c6967c6928d14d9cbf57642a2f8e41ab40ccc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# long-cadence-transit-model-rebinning

## Summary

Corrects long-integration smearing in space-based transit photometry (e.g. 29.4-minute Kepler long cadence) by computing the Mandel & Agol (2002) analytic transit model on a 10x finer temporal scale and rebinning it to the observed cadence before MCMC fitting, as implemented in TAP.

## When to use

Use when fitting transit light curves with long integration times (e.g. Kepler 29.4-minute long-cadence data), where unbinned model evaluation would distort fitted parameters such as Rp/Rs and a/Rs.

## When NOT to use

- Short-cadence or ground-based photometry where integration smearing is negligible — the finer-scale model computation and rebinning add cost without benefit for well-sampled light curves
- Analyses that require phase folding or rebinning multiple light curves into a single fold — TAP is explicitly designed to simultaneously analyze multiple transits of varying cadence without phase folding and rebinning

## Inputs

- Kepler long-cadence photometry (29.4-minute cadence) retrieved from the MAST archive for targets such as Kepler-4b through Kepler-8b
- TAP configuration with circular orbits locked, limb-darkening parameters left free, and the 10x finer model-cadence setting enabled

## Outputs

- Rebinned transit models matched to the observed 29.4-minute sampling at each MCMC step
- Per-planet posterior summaries (median and 1-sigma) of Period, Inclination, a/Rs, and Rp/Rs (e.g. tap_kepler_4_8_parameters.csv)

## How to apply

When the input photometry comes from long-integration observations such as Kepler long-cadence (29.4 min) light curves, configure TAP (in IDL) to calculate the EXOFAST implementation of the Mandel & Agol (2002) analytic transit model on a finer temporal scale (here 10x finer) and then integrate/rebin it to the observed sampling before each MCMC likelihood evaluation. This removes the light-curve distortion caused by intra-cadence smearing so that the Carter & Winn (2009) wavelet likelihood can robustly parameterize white and red noise while Period, Inclination, a/Rs, and Rp/Rs are sampled. The decision point is the ratio of model time step to observed cadence: the finer scale must be an integer multiple of the observed cadence so rebinned models align with the 29.4-minute sampling. Validate by checking fitted parameters against published values and computing the fractional difference versus Kipping and Bakos (2010), confirming |FD| < 1 for Kepler-4b through Kepler-7b.

## Related tools

- **Transit Analysis Package (TAP)** (IDL GUI that computes transit models on a finer temporal scale and integrates them to the observed long cadence during MCMC fitting) — https://github.com/zgazak/TAP
- **IDL** (Language environment in which TAP runs the rebinning workflow)
- **EXOFAST** (Provides the Mandel & Agol (2002) analytic transit model evaluated at the finer temporal scale before rebinning)

## Evaluation signals

- Fitted parameters fall within the quoted 1-sigma of the TAP column of Table 5 for each planet
- Fractional difference (FD, Eq. 5) versus Kipping and Bakos (2010) 'A.c' values satisfies |FD| < 1 for Kepler-4b through Kepler-7b
- Kepler-8b period difference is flagged as significant while other parameters remain in agreement, reproducing res_kepler8_discrepancy
- Rebinned model values align with the 29.4-minute sampling grid of the Kepler long-cadence data
- MCMC uncertainties reflect wavelet-parameterized white and red noise rather than classic chi-squared errors

## Limitations

- Derived transit parameters vary between fitting methods, and confidence depends critically on the treatment of noise, stellar limb darkening, and cadence
- The technique targets long-integration observations specifically; without integration smearing the finer-scale computation is unnecessary
- Agreement with prior publications is statistical (|FD| < 1), not exact, and individual parameters can still show significant discrepancies (as with the Kepler-8b period)

## Evidence

- [intro] Long-integration smearing correction: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
- [intro] Model implementation via EXOFAST: "The TAP software package uses EXOFAST [23], an improved implementation of the [15] analytic transit model"
- [abstract] Wavelet likelihood motivation: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [abstract] Agreement with Kipping and Bakos 2010: "and space-based Kepler 4b–8b (Kipping and Bakos 2010) show good agreement between TAP and those publications"
