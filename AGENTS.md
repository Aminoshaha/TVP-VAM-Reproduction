# Agent Instructions

## Current Project Phase

Phase 0 is complete when the audit and specification files exist and the draft PR is opened. Do not implement the numerical TVP pipeline in Phase 0.

## Hard Stop Conditions

Stop and ask for review before doing any of the following:

- Installing VAMToolbox, ASTRA, torch or other runtime dependencies.
- Creating `src/` implementation code for the TVP numerical pipeline.
- Modifying VAMToolbox core source.
- Exporting projection sequences and calling them printable.
- Filling unknown physical calibration fields with guessed values.
- Adding hardware-control code.

## Required Scientific Caution

Ideal simulations are allowed only as ideal simulations. Do not claim experimental accuracy until measured calibration data are supplied and validated.

Keep these categories separate:

- Ideal numerical parameters.
- Literature/reference parameters.
- Experimentally measured parameters.
- Unknown parameters.

Unknown values should remain `null`, `unknown`, or explicitly unresolved.

## VAMToolbox Rules

Before implementation work:

1. Inspect the installed VAMToolbox version and commit.
2. Confirm the public API from source and examples.
3. Record the VAMToolbox commit/version in run outputs.
4. Use wrappers around confirmed public APIs rather than editing upstream source.
5. Distinguish `A.forward`, `A.backward`, filtered reconstruction and optimization gradients.

Audited upstream reference for Phase 0:

- Repository: `computed-axial-lithography/VAMToolbox`
- Commit: `c2757e69f91814bfa85e4240e80dc09441173fe2`
- Version reported by source: `3.0.0`

## Coding Standards For Future Phases

- Use type hints and docstrings.
- State array shapes, axes and units in docstrings.
- Use pytest for numerical and artifact tests.
- Use fixed random seed `2026` unless a config overrides it explicitly.
- Keep configuration, computation, plotting, I/O and hardware-control boundaries separate.
- Save floating-point arrays and metadata, not just plots.
- Treat NaN and Inf as failures.
- Enforce nonnegative projection values explicitly.

## First Approved Implementation Target

After review, the first implementation should be a 2D ideal baseline:

- 256 x 256 binary annulus.
- Outer diameter 128 px.
- Inner diameter 64 px.
- 360 angles over `[0, 180)`.
- Nonnegative projections.
- Unfiltered adjoint backprojection.
- Threshold sweep and metrics.
- Saved arrays, plots, CSV metrics, config snapshot, logs and Git commit hash.

## Pull Request Guidance

For Phase 0 PRs, include only audit, specification, plan, README and configuration files. Do not include generated output directories or implementation code.