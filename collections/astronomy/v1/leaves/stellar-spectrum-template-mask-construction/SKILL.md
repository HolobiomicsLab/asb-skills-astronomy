---
name: stellar-spectrum-template-mask-construction
description: Use when you have a reduced stellar spectrum template (e.g. a SERVAL
  template FITS for a CARMENES VIS/NIR, HARPS, HARPN, or EXPRES target) and need a
  line-list mask for CCF-based RV computation.
when_to_use_negative:
- When a suitable pre-computed mask already exists for the target's spectral type
  — for CARMENES M dwarfs (M0.0 V–M7.0 V, vsini <= 2.9 km/s) raccoon ships default
  masks (e.g. `J12123+544Sdefault`) that can be passed directly to `raccoonccf`
- When you need RVs from an already-computed CCF — use the `raccoonccf`/CCF-fitting
  step instead of mask construction
- When the spectra are not echelle spectrograph observations compatible with raccoon's
  supported instruments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: raccoon
  role: Open source Python/Fortran package implementing the CCF method; mask construction
    is one of its three capabilities
  repo: github.com/mlafarga/raccoon
- name: raccoonmask
  role: Entry-point script that builds weighted binary masks from a stellar spectrum
    template with continuum filtering and line-selection thresholds
  repo: github.com/mlafarga/raccoon
- name: Python
  role: Implementation language of raccoon; fortran subroutines are compiled with
    numpy.f2py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:33d985ff5d42cdbdf69eab72b9bdd101c2ece7747d31d32e3d72d39ae10e0dd0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - raccoon
  - raccoonccf
  - raccoonmask
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# stellar-spectrum-template-mask-construction

## Summary

Build weighted binary spectral masks from a stellar spectrum template using the `raccoonmask` entry point of the raccoon package, so the resulting mask can be cross-correlated with observed spectra to derive radial velocities and activity indicators.

## When to use

Use when you have a reduced stellar spectrum template (e.g. a SERVAL template FITS for a CARMENES VIS/NIR, HARPS, HARPN, or EXPRES target) and need a line-list mask for CCF-based RV computation.

## When NOT to use

- When a suitable pre-computed mask already exists for the target's spectral type — for CARMENES M dwarfs (M0.0 V–M7.0 V, vsini <= 2.9 km/s) raccoon ships default masks (e.g. `J12123+544Sdefault`) that can be passed directly to `raccoonccf`
- When you need RVs from an already-computed CCF — use the `raccoonccf`/CCF-fitting step instead of mask construction
- When the spectra are not echelle spectrograph observations compatible with raccoon's supported instruments

## Inputs

- Stellar spectrum template FITS file (e.g. SERVAL template like J07446+035.fits)
- Template type identifier (e.g. `serval`)
- Target/object name (e.g. J07446+035)
- Instrument identifier: `CARM_VIS`, `CARM_NIR`, `HARPS`, `HARPN`, or `EXPRES`
- Optional telluric mask file or the `default` mask

## Outputs

- Weighted binary mask file in `.mas` format (e.g. `TPL.mas` with line positions, depths/weights, and widths)

## How to apply

Run `raccoonmask` with the template FITS file as the first argument, the template type (e.g. `serval`), the target name, and the instrument (`--inst CARM_VIS`, `CARM_NIR`, `HARPS`, `HARPN`, or `EXPRES`); set `--tplrv` to the target's approximate absolute RV (e.g. 10 km/s) so template lines land at rest wavelengths. Apply continuum filtering (`--cont poly` with `--contfiltmed 1`, `--contfiltmax 400`, `--contpolyord 2`) and select lines via the FWHM window `--line_fwhmmin 2.00` to `--line_fwhmmax 30.00` and a minimum contrast `--line_contrastminmin 0.06`; weight lines with `--line_depthw_percentdeepest 0.10` and `--line_depthw_depthmaxquantile 0.6`. Optionally provide a telluric mask (`--filtell default --tellbroadendv 1`) to reject telluric-contaminated regions. The script writes a `.mas` weighted binary mask (e.g. `TPL.mas`); alternatively, for CARMENES M dwarfs (M0.0 V–M7.0 V), reuse a pre-built mask such as `J07274+052default` rather than building one.

## Related tools

- **raccoon** (Open source Python/Fortran package implementing the CCF method; mask construction is one of its three capabilities) — github.com/mlafarga/raccoon
- **raccoonmask** (Entry-point script that builds weighted binary masks from a stellar spectrum template with continuum filtering and line-selection thresholds) — github.com/mlafarga/raccoon
- **Python** (Implementation language of raccoon; fortran subroutines are compiled with numpy.f2py)

## Examples

```
raccoonmask PATH/TO/TPL.fits serval J07446+035 --inst CARM_VIS --tplrv 10. --cont poly --contfiltmed 1 --contfiltmax 400 --contpolyord 2 --line_fwhmmin 2.00 --line_fwhmmax 30.00 --line_contrastminmin 0.06 --line_depthw_percentdeepest 0.10 --line_depthw_depthmaxquantile 0.6 --filtell default --tellbroadendv 1 --verbose
```

## Evaluation signals

- Running `raccoonmask -h` displays all available options, confirming a working installation
- A `.mas` mask file is produced with a filename encoding the applied thresholds (e.g. serval_tellbervmax_fwhm2.00-30.00_contrminmin0.06_depthwq0.60_contrastmeanfwhm-1.mas), so the expected name itself encodes fwhm 2.00–30.00, contrastminmin 0.06, and depthw q0.60
- The resulting mask, when used with `raccoonccf`, yields a CCF whose minimum recovers the target's absolute RV (given `--rvabs`/`--tplrv` near the true value)
- Mask line widths and contrasts fall inside the selection window (FWHM 2.00–30.00 pixels, contrast >= 0.06) as encoded in the mask filename

## Limitations

- raccoon uses fortran subroutines compiled with numpy.f2py, so a compiler/toolchain is required; if installed from source, leave the source directory before testing `raccoonccf -h`
- Telluric contamination must be handled — default telluric masks are invoked with `--filtell default`, and uncorrected barycentric/drift shifts require `--rvshift` (mask building should match the same telluric treatment, e.g. `--tellbroadendv 1`)
- Mask is built from one stellar template, so it is optimal only for targets near that template's spectral type and rotation (vsini); masks ship with an associated SpT and vsini limit (e.g. M3.5 V, vsini <= 2.0 km/s)

## Evidence

- [intro] raccoonmask example: "build weighted binary masks from a stellar spectrum template"
- [methods] entry points: "### Entry points
- `raccoonccf`
- `raccoonmask`"
- [readme] raccoonmask invocation: [evidence span withheld — anchor status: external_doc]
- [readme] available masks table: [evidence span withheld — anchor status: external_doc]
- [readme] implementation note: [evidence span withheld — anchor status: external_doc]
