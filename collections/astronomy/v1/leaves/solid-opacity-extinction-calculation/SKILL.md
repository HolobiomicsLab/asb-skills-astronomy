---
name: solid-opacity-extinction-calculation
description: Use when you have converged cloud-model output (dominant condensed species
  per atmospheric layer) and need solid opacities and the wavelength-dependent extinction
  of the cloud layer for spectra or retrieval.
when_to_use_negative:
- You need gas-phase or full transmission/emission spectra rather than solid cloud
  opacities — use ExoLyn's petitRADTRANS coupling instead
- Cloud structure has not converged (no per-layer condensed species table), since
  optool coupling requires the converged solid cloud results
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ExoLyn
  role: Produces the converged cloud structure and per-layer condensed species fed
    to the opacity calculation
  repo: github.com/helonghuangastro/exolyn
- name: optool
  role: Calculates solid opacities from the converged solid cloud composition, from
    which the extinction effect is derived
- name: Python 3
  role: Runs the ExoLyn workflow (python3 relaxation.py) whose output is coupled to
    optool
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:6925c72a5b695e95adb31035ce0b76bc6856778f04df8103d60ab1112ee8c544
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExoLyn
  - Python
  - Python 3
  - bash
  - numpy
  - optool
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# solid-opacity-extinction-calculation

## Summary

Couples converged ExoLyn solid cloud results (per-layer condensed species) to optool to compute solid opacities and derive the extinction effect of cloud layers, e.g. Fe clouds with strong extinction on C-rich planets.

## When to use

Use when you have converged cloud-model output (dominant condensed species per atmospheric layer) and need solid opacities and the wavelength-dependent extinction of the cloud layer for spectra or retrieval.

## When NOT to use

- You need gas-phase or full transmission/emission spectra rather than solid cloud opacities — use ExoLyn's petitRADTRANS coupling instead
- Cloud structure has not converged (no per-layer condensed species table), since optool coupling requires the converged solid cloud results

## Inputs

- Converged ExoLyn solid cloud results (condensation rates and per-layer condensed species from relaxation.py)
- parameters.txt specifying bulk elemental composition (e.g. C-rich configuration)
- Optical constants for condensed species consumed by optool

## Outputs

- Composition table per atmospheric layer (dominant condensed species per layer), e.g. c_rich_cloud_profile.csv
- Solid opacities of the condensed material from optool
- Opacity/extinction record of the cloud layer (e.g. Fe cloud layer) demonstrating the extinction effect

## How to apply

After running ExoLyn to convergence (bash build.sh; python3 relaxation.py with parameters.txt set to the desired bulk composition, e.g. the C-rich configuration), export a composition table per atmospheric layer recording the dominant condensed species per layer. Couple these converged solid cloud results to optool to calculate the solid opacities of the condensed material, then derive the extinction effect of the cloud layers from those opacities. Write the composition table together with the opacity/extinction record of the cloud layer (e.g. the Fe layer) to a CSV file such as c_rich_cloud_profile.csv. Judge correctness by checking that the expected condensed species emerges as dominant per composition (Fe for C-rich bulk; MgSiO3/Mg2SiO4 on top of Fe for solar-like hot Jupiter compositions) and that the extinction signature matches expectations (strong for C-rich Fe clouds, 8–10 μm absorption for Mg-silicates).

## Related tools

- **ExoLyn** (Produces the converged cloud structure and per-layer condensed species fed to the opacity calculation) — github.com/helonghuangastro/exolyn
- **optool** (Calculates solid opacities from the converged solid cloud composition, from which the extinction effect is derived)
- **Python 3** (Runs the ExoLyn workflow (python3 relaxation.py) whose output is coupled to optool)

## Examples

```
bash build.sh && python3 relaxation.py  # then couple converged solid cloud results to optool and export c_rich_cloud_profile.csv
```

## Evaluation signals

- Fe emerges as the dominant cloud material per atmospheric layer for a C-rich bulk composition, as verified in c_rich_cloud_profile.csv
- The derived extinction record shows a strong extinction effect for the Fe cloud layer on C-rich planets
- The per-layer composition table covers every atmospheric layer with a dominant condensed species entry
- For solar-composition hot Jupiter runs, Mg-silicate-dominated clouds appear on top of an iron cloud, consistent with layered structure

## Limitations

- The opacity/extinction derivation is only as good as the converged 1D cloud structure; ExoLyn is a 1D model balancing physical consistency with computational efficiency
- Extinction signatures are composition-dependent: MgSiO3 and Mg2SiO4 give a 8–10 μm absorption feature, while C-rich planets yield Fe clouds with strong extinction — misassigning the bulk composition changes the expected dominant species entirely

## Evidence

- [abstract] Exolyn is a standalone, open source package capable of being combined with optool to calculate solid opacities: "Exolyn is a standalone, open source package capable of being combined with optool to calculate solid opacities"
- [abstract] SiO2-dominated clouds form on metal-rich planets and Fe clouds with strong extinction form on C-rich planets: "SiO2-dominated clouds form on metal-rich planets and Fe clouds with a strong extinction effect form on C-rich planets"
- [other] Export composition table and extinction record: [evidence span withheld — anchor status: not_found]
- [readme] README compile/run instructions: "After downloading the code, you can compile the fortran part by running `bash build.sh`"
