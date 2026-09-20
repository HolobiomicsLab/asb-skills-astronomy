# transit-lightcurve-mcmc-comparison-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **prepare_lightcurve** — normalize and detrend transit photometry while preserving transit signals  →  `stellar-lightcurve-detrending`, `orbital-period-measurement-from-transit-epochs`, `exoplanet-transit-photometry-modelling`, `multi-instrument-lightcurve-joint-fitting`
2. **fit_transit_posterior** — jointly fit transit shape and correlated photometric noise  →  `mcmc-transit-light-curve-fitting`, `multi-transit-simultaneous-fitting-with-locked-parameters`, `simultaneous-multi-transit-parameter-sharing`
3. **compare_published_parameters** — quantify agreement between inferred and published transit parameters  →  `cross-survey-transit-parameter-comparison`, `transit-parameter-comparison-across-publications`, `published-transit-parameter-agreement-verification`, `mcmc-posterior-parameter-estimation--625da6f013605ea9`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
