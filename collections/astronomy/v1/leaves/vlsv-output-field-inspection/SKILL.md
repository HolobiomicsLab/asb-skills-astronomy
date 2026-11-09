---
name: vlsv-output-field-inspection
description: 'Use when a Vlasiator run has finished and you must verify a .vlsv output
  file: checking that requested per-species moment fields were written, that values
  are nonzero, and that their spatial pattern matches the physics (e.g., O+ localised
  near Earth).'
when_to_use_negative:
- 'The run is proton-only (single-population): there is no second-species field to
  verify, only standard proton moments.'
- You need to modify simulation physics or run configuration rather than inspect existing
  output — this skill reads .vlsv files, it does not change or re-run simulations.
- The input is not a VLSV file (e.g., MHD NetCDF or observational data); Analysator's
  readers target Vlasiator's VLSV format specifically.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: analysator
  role: Python toolkit used to read and analyze VLSV output files and plot inspected
    fields (VlsvReader, plot_colormap)
  repo: https://github.com/fmihpc/analysator
- name: vlsv
  role: VLSV file-format I/O library underlying Vlasiator output reading
- name: Vlasiator
  role: Producer of the .vlsv output files being inspected; hybrid-Vlasov magnetosphere
    simulation with multi-ion-population support
  repo: https://github.com/fmihpc/vlasiator
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:e381db10887edf8522703ff3216d61c13ad88cb62ed23402a5ebf65840d6bafa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/analysator
  tools:
  - Vlasiator
  - analysator
  - vlsv
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# vlsv-output-field-inspection

## Summary

Inspect Vlasiator simulation output files in the VLSV format to confirm that per-population fields (e.g., O+ density and bulk velocity alongside proton density and bulk velocity) exist, are nonzero, and show the expected spatial localisation. Uses the fmihpc/analysator Python tooling built on top of the fmihpc/vlsv reader library.

## When to use

Use when a Vlasiator run has finished and you must verify a .vlsv output file: checking that requested per-species moment fields were written, that values are nonzero, and that their spatial pattern matches the physics (e.g., O+ localised near Earth).

## When NOT to use

- The run is proton-only (single-population): there is no second-species field to verify, only standard proton moments.
- You need to modify simulation physics or run configuration rather than inspect existing output — this skill reads .vlsv files, it does not change or re-run simulations.
- The input is not a VLSV file (e.g., MHD NetCDF or observational data); Analysator's readers target Vlasiator's VLSV format specifically.

## Inputs

- Vlasiator .vlsv output file(s) from a multi-population (proton + heavy-ion) run
- Expected field checklist: per-species density and bulk velocity for each ion population

## Outputs

- Verification report that O+ (or other heavy-ion) density and bulk-velocity fields exist in the .vlsv file
- Confirmation that the heavy-ion density is nonzero and spatially localised to the near-Earth (ionospheric) region
- Optional colormap visualisations of the inspected fields

## How to apply

After executing a multi-population Vlasiator run, load the produced .vlsv file with Analysator's VlsvReader (imported as `import analysator as pt`, giving access to pt.vlsvfile.VlsvReader and plotting helpers like pt.plot.plot_colormap). First enumerate the file's variable names to confirm the per-species moment outputs were enabled in the run configuration — density and bulk velocity must be present separately for the proton population and for the added heavy-ion population (e.g., O+ flowing in from the ionospheric boundary). Then check the heavy-ion density array is nonzero overall and that its spatial distribution is localised to the near-Earth (ionospheric) region rather than spread through the solar wind, which flows in protons/helium from the upstream boundary. Use the PTNONINTERACTIVE=1 environment variable for batch/headless inspection, and plot_colormap for visual field checks.

## Related tools

- **analysator** (Python toolkit used to read and analyze VLSV output files and plot inspected fields (VlsvReader, plot_colormap)) — https://github.com/fmihpc/analysator
- **vlsv** (VLSV file-format I/O library underlying Vlasiator output reading)
- **Vlasiator** (Producer of the .vlsv output files being inspected; hybrid-Vlasov magnetosphere simulation with multi-ion-population support) — https://github.com/fmihpc/vlasiator

## Examples

```
export PTNONINTERACTIVE=1; python -c "import analysator as pt; vlsv=pt.vlsvfile.VlsvReader('bulk.0000000.vlsv'); print(vlsv.read_variable_names())"
```

## Evaluation signals

- The .vlsv variable list contains separate density and bulk-velocity entries for both the proton population and the O+ population (per-species moments enabled).
- The O+ density array is nonzero — an all-zero array indicates the ionospheric boundary inflow or sparse velocity-space initialisation failed.
- O+ density maxima are spatially confined to the near-Earth/ionospheric region, not distributed through the solar-wind inflow region.
- Proton fields remain intact and comparable to the task_004 proton-only baseline run, confirming the added population did not corrupt existing outputs.
- Visual inspection via plot_colormap shows physically plausible field structures rather than numerical noise (Vlasiator's Vlasov representation is expected to be numerically noiseless).

## Limitations

- Vlasiator uses a sparse velocity-space grid and immense computational demands, so output files can be large and per-species moments must have been explicitly enabled in the run configuration before execution — they cannot be recovered afterwards.
- The verification is limited to the near-Earth localisation heuristic; the skill cannot distinguish a physically correct O+ outflow from a misconfigured boundary condition without comparison to reference runs.
- Analysator is developed at the University of Helsinki for Vlasiator's VLSV format only; it does not read other simulation formats.

## Evidence

- [readme] Analysator is a set of Python tools and scripts used to read and analyze Vlasiator output files (VLSV).: [evidence span withheld — anchor status: not_found]
- [other] Workflow verification step for the two-population run: [evidence span withheld — anchor status: not_found]
- [other] Multi-population boundary conditions from the source card finding: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract identifying Vlasiator as the simulation producing these outputs: "A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
