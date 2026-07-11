# Specification

## Project Objective

Create a reproducible research repository for an ideal tomographic volumetric printing baseline based on VAMToolbox, then progressively add measured calibration data. Ideal simulations must not be represented as experimentally accurate.

## Scientific Workflow

The intended workflow is:

```text
target geometry -> voxel target -> multi-angle nonnegative projections -> accumulated dose -> thresholded solidification prediction -> quantitative metrics
```

## Mandatory Rules

1. Inspect the actual VAMToolbox repository, README, examples, installed version and public API before implementation.
2. Do not invent VAMToolbox functions or rely on remembered interfaces.
3. Do not modify VAMToolbox core source unless explicitly approved.
4. Keep ideal, literature-reference and experimentally measured parameters separate.
5. Do not silently assign unknown physical parameters.
6. All run parameters must be configuration-driven, not hard-coded.
7. Separate numerical computation, plotting, I/O and future hardware control.
8. Use type hints, docstrings, pytest and fixed random seeds.
9. State array shapes, axis meanings and units or dimensionless status in docstrings.
10. Distinguish forward projection `A`, adjoint backprojection `A^T`, filtered reconstruction and optimization gradients.
11. Numerically test the adjoint relation `<Ax,y> ~= <x,A^T y>`.
12. Enforce nonnegative projection values.
13. Save intermediate floating-point arrays, not only rendered PNG files.
14. Record configuration snapshots, environment versions, logs and Git commit hashes.
15. Do not equate voxel size with physical printer resolution.
16. Do not call an exported projection sequence printable until calibration and synchronization are verified.

## Configuration Contract

- Primary config format: YAML.
- Known physical values must be represented explicitly.
- Unknown physical values must remain `null` or `unknown`.
- Derived values must be marked as derived and must identify their inputs.
- Run outputs must include an exact copy of the config used.

The initial config is `configs/ideal_10mm.yaml`.

## Array Contracts For The First Baseline

Target array:

- Shape: `(height_px, width_px)` for the first 2D baseline before any VAMToolbox adapter conversion.
- Values: binary dimensionless target mask.
- Axes: row/y first, column/x second.

Sinogram array:

- Shape target: `(detector_index, angle_index)` for the 2D baseline.
- Values: nonnegative floating-point projection intensities in normalized simulation units.
- Axes must be verified against the selected VAMToolbox backend.

Accumulated dose array:

- Shape: same spatial shape as target after the project-specific adapter returns from `A^T`.
- Values: normalized floating-point dose.
- Units: dimensionless for the ideal baseline.

Threshold result:

- Shape: same as target.
- Values: binary predicted solidification mask.
- Threshold source: configuration and threshold sweep metadata.

Metrics table:

- One row per threshold candidate or one summary row for a selected threshold.
- Required fields: threshold, IoU, Dice, target under-dose fraction, background over-dose fraction and dose-separation margin.

## Artifact Contract

Each successful numerical run after Phase 0 should write:

- Config snapshot.
- Environment snapshot.
- Git commit hash.
- Target floating array.
- Projection/sinogram floating array.
- Accumulated dose floating array.
- Threshold result array.
- Metrics CSV.
- Human-readable log.
- Rendered figures for inspection.

Rendered PNG files are secondary artifacts; they do not replace floating-point arrays.

## VAMToolbox Boundary

VAMToolbox is an external dependency and operator provider. Project code may wrap confirmed public APIs but must not edit VAMToolbox core source by default.

Confirmed API anchors from Phase 0:

- `vamtoolbox.geometry.TargetGeometry`
- `vamtoolbox.geometry.ProjectionGeometry`
- `vamtoolbox.projectorconstructor.projectorconstructor`
- projector `forward(x)`
- projector `backward(b)`
- `vamtoolbox.optimize.Options`
- `vamtoolbox.optimize.optimize`
- `vamtoolbox.imagesequence.ImageConfig`
- `vamtoolbox.imagesequence.ImageSeq`
- `vamtoolbox.pipeline.PrintConfig`
- `vamtoolbox.pipeline.VAMPipeline`

Any additional API use must be audited in source before adoption.

## Parameter Separation

Ideal parameters:

- Synthetic geometry.
- Synthetic normalized dose threshold.
- Synthetic angle grid.
- Numerical normalization choices.

Known experimental metadata:

- Wavelength 405 nm.
- Vial inner diameter 10 mm.
- Resin record `WDS3532:PEGDA:EDAB:CQ = 2:1:0.003:0.003`.

Unknown measured parameters:

- Projector model, optics, pixel size, irradiance and grayscale response.
- Vial outer diameter, wall thickness, glass type and refractive index.
- Resin absorption coefficient, refractive indices and gel dose.
- Rotation-stage model, RPM, angle-time trace and synchronization.
- PSF and optical blur.

Unknown measured parameters must not be backfilled with convenient defaults.

## Test Requirements

Minimum test categories after Phase 0:

- Configuration loading and unknown preservation.
- Synthetic geometry generation.
- Forward/adjoint shape consistency.
- Numerical adjoint relation.
- Projection nonnegativity.
- NaN/Inf rejection.
- Reproducibility with seed `2026`.
- Artifact existence and metadata completeness.

## Documentation Requirements

Every numerical function should document:

- Input shapes.
- Output shapes.
- Axis meanings.
- Units or dimensionless status.
- Normalization behavior.
- Randomness and seed handling, if applicable.

## Phase Gates

Phase 0:

- Audit and specification only.
- Required files and draft PR.
- No dependency installation.
- No numerical implementation.

Phase 1:

- Implement first 2D ideal baseline after approval.
- Add tests and artifact writers.
- Validate operator contracts before interpreting results.

Later phases:

- Add measured calibration data only when measurements are supplied.
- Add export/hardware-control work only after calibration and synchronization are verified.