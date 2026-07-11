# TVP-VAMToolbox Reproduction

This repository is being bootstrapped as a reproducible research project for tomographic volumetric printing (TVP) simulations based on VAMToolbox.

Phase 0 is documentation-only. It audits the starting repository, the expected environment, and the confirmed VAMToolbox API surface before any numerical TVP pipeline is implemented.

## Current Status

- Phase: `0` audit and specification
- Numerical pipeline: not implemented
- VAMToolbox core source: not modified
- Upstream reference audited: `computed-axial-lithography/VAMToolbox@c2757e69f91814bfa85e4240e80dc09441173fe2`
- Upstream package version reported by `vamtoolbox/__init__.py`: `3.0.0`

## Phase 0 Outputs

- `ENVIRONMENT_AUDIT.md` records repository, local environment, dependency and compatibility findings.
- `VAMTOOLBOX_API_MAP.md` maps required TVP functions to confirmed VAMToolbox objects, files and examples.
- `IMPLEMENTATION_PLAN.md` proposes the first approved 2D ideal baseline without implementing it.
- `SPEC.md` defines project rules, data contracts, artifacts and validation expectations.
- `AGENTS.md` gives future agent instructions and stop conditions.
- `configs/ideal_10mm.yaml` stores known ideal configuration metadata while preserving unknown physical values as `null`.

## Proposed Initial Directory Structure

```text
configs/              Configuration snapshots for ideal and calibrated runs
docs/                 Future notes, figures and calibration references
src/tvp_vam/          Future project package, after Phase 0 approval
tests/                Future pytest suite for numerical and reproducibility checks
outputs/              Ignored generated arrays, metrics, logs and figures
```

Only `configs/` is created in Phase 0 because no numerical implementation has been approved yet.

## Scientific Boundary

The first implementation target after review is an ideal 2D baseline: target geometry, voxel target, nonnegative projections, accumulated dose, thresholded solidification prediction and quantitative metrics. Ideal results must not be described as experimentally accurate until projector, vial, resin, rotation and synchronization calibration are measured.

## VAMToolbox Inclusion Strategy

Use VAMToolbox as a pinned external dependency or sibling checkout during development, not as copied source in this repository. The Phase 0 audit recommends pinning the upstream commit above and recording the resolved commit, environment, configuration snapshot and project commit in every run artifact.