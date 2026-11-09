---
name: stellar-identifier-resolution-tic-toi-simbad
description: Use when the only available target specification is a TIC number, TOI
  number, or a stellar name resolvable in Simbad, and downstream steps (Gaia query
  within a 200 arcsec search radius, TPF/FFI retrieval) require its sky coordinates.
when_to_use_negative:
- The target already has precise coordinates and you would bypass identifier resolution
  entirely — coordinates can seed the Gaia/TPF steps directly
- The identifier is a name not resolvable in Simbad and no TIC/TOI number is available
  — the tool requires 'Any TIC, TOI, or star's name resoluble in Simbad'
- The goal is contamination analysis for a non-TESS instrument (e.g. Kepler/K2 aperture
  photometry), where the TIC/TOI resolution step does not apply
edam_operation: http://edamontology.org/operation_2422
tools:
- name: TESS-cont
  role: 'Hosts the resolution step: the `target` mandatory config parameter triggers
    identifier resolution to coordinates before the Gaia query and PRF computation'
  repo: https://github.com/castro-gzlz/TESS-cont
- name: Simbad
  role: Fallback resolver for arbitrary star names that are not TIC or TOI numbers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:013863268a92db53a15196d7b55d2dd755dceb22d36bf0a3c19388bc7029045f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castro-gzlz/TESS-cont
  tools:
  - Python
  - Simbad
  - TESS-cont
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Stellar identifier resolution (TIC/TOI/Simbad) to sky coordinates

## Summary

Resolves any TIC, TOI, or Simbad-resolvable target name (e.g. TOI-5005 = TIC 282485660) into the sky coordinates needed to center the Gaia catalogue query and TESS pixel-data download in the TESS-cont contamination workflow.

## When to use

Use when the only available target specification is a TIC number, TOI number, or a stellar name resolvable in Simbad, and downstream steps (Gaia query within a 200 arcsec search radius, TPF/FFI retrieval) require its sky coordinates.

## When NOT to use

- The target already has precise coordinates and you would bypass identifier resolution entirely — coordinates can seed the Gaia/TPF steps directly
- The identifier is a name not resolvable in Simbad and no TIC/TOI number is available — the tool requires 'Any TIC, TOI, or star's name resoluble in Simbad'
- The goal is contamination analysis for a non-TESS instrument (e.g. Kepler/K2 aperture photometry), where the TIC/TOI resolution step does not apply

## Inputs

- Target identifier string in the `target` config parameter: a TIC number (e.g. TIC 282485660), a TOI number, or a Simbad-resolvable star name
- TESS-cont configuration file (.ini, e.g. TOI-5005_S65.ini)

## Outputs

- Resolved sky coordinates of the target star used to center the Gaia search and TPF/FFI retrieval
- Resolved TIC identifier (e.g. TOI-5005 -> TIC 282485660) anchoring the contamination analysis

## How to apply

Accept the target identifier from the [MANDATORY] `target` field of the TESS-cont configuration file — it may be any TIC number, TOI number, or star's name that Simbad can resolve. Resolve it to a unique source and obtain its sky coordinates, which seed the Gaia DR2/DR3 query (default search_radius = 200 arcsec) and the download of the corresponding TESS Target Pixel File (tpf by default, or tesscut with cutout_size 11,11 for FFI targets). A TIC number alone is sufficient, as demonstrated for TOI-5005 (TIC 282485660 in Sector 65); optional parameters such as `target_name` and `sector` only control labeling and data selection, not the resolution itself. The rationale is that contamination analysis must be anchored on a single, unambiguous astrometric position so that Gaia sources are ranked by flux contribution relative to the correct pixel footprint.

## Related tools

- **TESS-cont** (Hosts the resolution step: the `target` mandatory config parameter triggers identifier resolution to coordinates before the Gaia query and PRF computation) — https://github.com/castro-gzlz/TESS-cont
- **Simbad** (Fallback resolver for arbitrary star names that are not TIC or TOI numbers)

## Examples

```
python TESS-cont.py TOI-5005_S65.ini
```

## Evaluation signals

- The resolved identifier matches an expected mapping, e.g. TOI-5005 = TIC 282485660 as shown in the README example
- The resulting Gaia query returns the target among its sources, with the target star ranked as the dominant flux contributor inside the aperture
- The TPF (or tesscut FFI cutout of size 11,11) downloaded for the resolved coordinates covers the target position
- For SPOC PDCSAP workflows, the CROWDSAP/FLFRCSAP by-products in metrics.dat show no major differences with official TESS metrics, indicating the correct target and catalogue (DR2) were anchored

## Limitations

- Only identifiers resolvable via TIC, TOI, or Simbad are accepted; unresolvable names cause the workflow to fail at the mandatory `target` parameter
- Resolution alone is insufficient — the Gaia catalogue choice matters: TIC/SPOC PDCSAP is stacked to Gaia DR2, so DR3 coordinates/catalogue defaults may introduce discrepancies flagged in metrics.dat

## Evidence

- [readme] Mandatory target parameter accepts flexible identifiers: "It only requires the TIC/TOI** number (or Simbad name) of the target star to study"
- [readme] Config table defines the target parameter semantics: "target | Any TIC, TOI, or star's name resoluble in Simbad | Target star identifier"
- [methods] Methods section documents the same resolution contract: "Any TIC, TOI, or star's name resoluble in Simbad | Target star identifier"
- [readme] Concrete identifier mapping demonstrated for TOI-5005: "we specify the **TIC number** of TOI-5005 (TIC 282485660) to quantify the flux contributions to its SPOC photometric aperture in **Sector 65**"
