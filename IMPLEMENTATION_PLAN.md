# Implementation Plan

## Scope Boundary

This file proposes the first implementation after Phase 0 review. It does not implement the numerical TVP pipeline.

The Phase 1 target is an ideal 2D baseline only. It should establish reproducible project scaffolding, array contracts, operator tests and saved artifacts before any experimental calibration claims are made.

## Phase 0 Conclusions Driving Phase 1

- VAMToolbox should be pinned externally at `computed-axial-lithography/VAMToolbox@c2757e69f91814bfa85e4240e80dc09441173fe2` or another explicitly reviewed commit.
- The audited VAMToolbox version reports `3.0.0`.
- The local machine has Python 3.13.1 available through `py -3.13`, but VAMToolbox and ASTRA are not installed.
- Upstream VAMToolbox projectors expose `forward(x)` and `backward(b)` methods through `projectorconstructor()`.
- Upstream array shape conventions must be verified in an installed smoke test before project code depends on them.
- Project configuration must override or avoid VAMToolbox high-level `PrintConfig` physical defaults because this project has a 10 mm vial and many unknown calibration values.

## Phase 1 Baseline Target

Build a 2D ideal baseline with:

- 256 x 256 binary annulus.
- Configurable outer diameter 128 px.
- Configurable inner diameter 64 px.
- 360 uniformly spaced angles from 0 degrees inclusive to 180 degrees exclusive.
- Nonnegative projection values.
- Geometrically consistent unfiltered adjoint backprojection.
- Threshold sweep.
- IoU, Dice, target under-dose fraction, background over-dose fraction and dose-separation margin.
- Saved target, sinogram, accumulated dose, threshold result, histograms, CSV metrics and raw floating-point arrays.
- One-command reproducibility.

## Proposed Package Layout After Approval

```text
src/tvp_vam/
  __init__.py
  config.py
  geometry.py
  projectors.py
  dose.py
  metrics.py
  io.py
  plotting.py
  cli.py
tests/
  test_config.py
  test_geometry.py
  test_projector_contract.py
  test_adjoint.py
  test_metrics.py
  test_reproducibility.py
configs/
  ideal_10mm.yaml
```

## Module Responsibilities

`config.py`

- Load YAML configurations.
- Validate required fields, units and unknown/null preservation.
- Compute derived values such as voxel size only when inputs are sufficient.
- Save exact configuration snapshots into each output run directory.

`geometry.py`

- Generate the 2D annulus target from config.
- State output shape `(height_px, width_px)` and dimensionless binary values in docstrings.
- Avoid physical resolution claims.

`projectors.py`

- Wrap confirmed VAMToolbox projector creation.
- Record actual VAMToolbox commit/version and selected projector class.
- Expose explicit `forward` and `adjoint` methods.
- Enforce and test nonnegative projection outputs after forward/optimization steps.
- Keep filtered reconstruction and optimizer gradients separate from the adjoint.

`dose.py`

- Accumulate dose using the verified adjoint/backprojection operator.
- Apply global normalization and threshold sweeps.
- Save floating-point dose arrays before rendering plots.

`metrics.py`

- Compute IoU.
- Compute Dice.
- Compute target under-dose fraction.
- Compute background over-dose fraction.
- Compute dose-separation margin.
- Treat NaN/Inf as hard failures.

`io.py`

- Write `.npy` or `.npz` arrays.
- Write CSV metrics.
- Write JSON/YAML config snapshots.
- Write environment snapshot and Git commit hash.
- Write structured logs.

`plotting.py`

- Render target, sinogram, accumulated dose, threshold result and histograms.
- Never be required for numerical correctness.

`cli.py`

- Provide the one-command entry point after Phase 1 approval.
- Accept a config path and an output directory.
- Refuse to overwrite an existing run directory unless explicitly requested.

## Required Tests

1. Configuration validation

- Unknown physical parameters remain `null`.
- Known values load exactly from `configs/ideal_10mm.yaml`.
- Angle array is 360 values over `[0, 180)`.

2. Target generation

- Annulus output shape is `(256, 256)`.
- Values are binary and finite.
- Outer and inner diameters match config within pixel-grid conventions documented by the test.

3. Forward/adjoint consistency

- Build the selected VAMToolbox operator in a deterministic test configuration.
- Generate fixed-seed random `x` and `y` arrays.
- Verify `<A x, y> ~= <x, A^T y>` within a tolerance chosen after smoke-testing the backend.
- Run CPU path first; optional CUDA path may have looser tolerance.

4. Nonnegativity

- Confirm final projection arrays are nonnegative.
- Confirm clipping/enforcement is explicit and logged.

5. Shapes and axes

- Assert target, sinogram, dose and threshold array shapes.
- Assert sinogram axes are detector/ray coordinate first and angle second for the 2D baseline.

6. NaN/Inf checks

- Fail if target, sinogram, dose, threshold result, histogram bins or metrics contain NaN or Inf.

7. Reproducibility

- With random seed `2026`, two runs produce identical target and deterministic metrics under the same backend.
- Saved config snapshot and Git commit hash exist in output metadata.

## Execution Plan After Approval

1. Add packaging and test scaffolding without installing dependencies in the PR itself.
2. Add config loader and schema validation for `ideal_10mm.yaml`.
3. Add target generation and tests.
4. Add a thin VAMToolbox projector adapter only after installing/pinning VAMToolbox in a controlled environment.
5. Add forward/adjoint smoke tests and record the selected backend.
6. Add dose accumulation, threshold sweep and metrics.
7. Add output artifact writer for arrays, plots, CSV metrics, config snapshots, logs and environment records.
8. Add CLI entry point and a minimal README run command.
9. Run the full test suite on CPU and, if ASTRA CUDA is installed and verified, on CUDA.

## Non-goals For Phase 1

- No hardware control.
- No DLP projection export labeled as printable.
- No calibration-derived physical accuracy claims.
- No vial refraction, absorption or PSF correction unless measured values are supplied.
- No modification of VAMToolbox core source.

## Review Gate Before Implementation

Before Phase 1 begins, review should approve:

- VAMToolbox inclusion method and exact pin.
- Python and ASTRA installation strategy.
- Whether CPU-only smoke tests are acceptable for the first baseline.
- The tolerance for the adjoint consistency test.
- Output directory naming and artifact retention policy.