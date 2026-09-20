# cmb-cosmological-parameter-inference-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **configure_cmb_likelihood** — assemble a callable CMB theory-and-likelihood chain  →  `wmap-likelihood-chain-configuration`, `cmb-likelihood-module-integration`, `likelihood-chain-architecture-implementation`, `plugin-module-registration-pattern`
2. **sample_parameter_posterior** — sample the CMB likelihood over the bounded cosmological parameter space  →  `cosmological-parameter-mcmc-sampling`, `cosmological-parameter-sampler-setup`
3. **determine_burn_in** — derive a defensible burn-in length from the initialisation bias of the chains  →  `burn-in-determination-via-standard-error-comparison`, `monte-carlo-standard-error-analysis`, `cross-sampler-convergence-comparison`
4. **diagnose_autocorrelation** — quantify autocorrelation and effective sampling after the burn-in  →  `mcmc-autocorrelation-time-analysis`, `mcmc-convergence-diagnostic-tabulation`, `sampler-stopping-criterion-computation`
5. **summarize_constraints** — convert converged CMB posterior samples into physical constraints  →  `lambda-cdm-parameter-inference`, `mcmc-ensemble-sampling-cosmological-parameters`, `posterior-summary-statistics-computation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
