# Environment Audit

## Scope

This Phase 0 audit documents the starting repository state, the expected runtime environment, and the verified upstream VAMToolbox reference. It intentionally does not install dependencies, run VAMToolbox, or implement the numerical TVP pipeline.

## Repository State

Repository: `Aminoshaha/TVP-VAM-Reproduction`

Default branch at audit start: `main`

Observed repository contents on `main` before Phase 0 outputs:

- `README.md` with a one-line project description.

No package layout, test suite, configuration directory, data schema, or numerical implementation existed before this Phase 0 documentation branch.

## Proposed Initial Directory Structure

```text
configs/
  ideal_10mm.yaml          Known ideal baseline metadata and unknown calibration fields
src/tvp_vam/
  config.py                Future typed configuration loading and validation
  geometry.py              Future target generation and shape utilities
  projectors.py            Future wrapper around confirmed forward and adjoint operators
  dose.py                  Future dose accumulation and thresholding utilities
  metrics.py               Future IoU, Dice, dose-separation and failure metrics
  io.py                    Future array/config/log/environment artifact writers
  plotting.py              Future visualization-only helpers
tests/
  test_adjoint.py          Future <Ax,y> ~= <x,A^T y> tests
  test_nonnegativity.py    Future projection constraints
  test_shapes.py           Future array shape and axis-contract tests
  test_reproducibility.py  Future fixed-seed rerun tests
outputs/                  Generated artifacts; should be ignored by git
docs/                     Future calibration notes and references
```

Phase 0 creates only the required documentation and `configs/ideal_10mm.yaml`.

## VAMToolbox Source Audited

Canonical upstream repository identified by GitHub search:

- `computed-axial-lithography/VAMToolbox`
- Default branch: `main`
- Commit audited: `c2757e69f91814bfa85e4240e80dc09441173fe2`
- Recent commit message: `Merge pull request #27 from computed-axial-lithography/tomo-edits`
- Package version reported in `vamtoolbox/__init__.py`: `3.0.0`

Important audited upstream files:

- `README.rst`
- `setup.py`
- `requirements-py313.txt`
- `install.ps1`
- `vamtoolbox/__init__.py`
- `vamtoolbox/geometry.py`
- `vamtoolbox/projectorconstructor.py`
- `vamtoolbox/projector/Projector2DParallel.py`
- `vamtoolbox/projector/Projector3DParallel.py`
- `vamtoolbox/optimize.py`
- `vamtoolbox/pipeline.py`
- `vamtoolbox/imagesequence.py`
- `vamtoolbox/voxelize.py`
- `examples/2Doptimization.py`
- `examples/3Doptimization.py`
- `examples/voxelizestl.py`
- `examples/imagesequence.py`
- `examples/gui_integration_example.py`

## Recommended VAMToolbox Inclusion

Recommended approach: keep VAMToolbox external and pinned.

Use one of these reproducible forms after review:

1. `requirements.txt` or lock file entry pointing at the audited Git commit.
2. Sibling checkout documented by path and commit, for local development only.
3. Git submodule only if the project needs persistent source inspection from this repository.

Do not copy VAMToolbox core source into this repository. Do not modify VAMToolbox core unless explicitly approved in a later issue.

## Upstream Installation Expectations

From upstream `README.rst` at the audited commit:

- Supported operating system: Windows.
- Supported Python: 3.13.
- Conda path: deprecated.
- Supported install path: native Python virtual environment plus `install.ps1`.
- ASTRA backend: standalone CUDA-bundled ASTRA Toolbox wheel from the ASTRA download site, not the ordinary Windows PyPI path.
- `torch`: optional, needed for PyTorch ray-tracing or algebraic propagators, not necessarily for the ASTRA/sparse OSMO/BCLP path.

The upstream packaging file `setup.py` uses `vamtoolbox/__init__.py` to read the version. `requirements-py313.txt` includes pinned scientific, image, visualization and optional torch dependencies. The file content appears UTF-16-like when fetched through the connector, so future local use should verify its encoding before tooling assumes UTF-8.

## Local Audit Snapshot

Audit machine date: 2026-07-11

Observed without installing dependencies:

| Item | Observed value |
| --- | --- |
| OS | Microsoft Windows NT 10.0.26200.0 |
| Default `python` | Python 3.8.20 |
| `py` launcher default | Python 3.13.1 |
| `py -3.13` | Python 3.13.1 |
| GPU | NVIDIA GeForce RTX 4060 Laptop GPU |
| NVIDIA driver | 566.26 |
| GPU memory | 8188 MiB |
| `nvcc` | Not found on PATH |
| `astra` import under Python 3.8 | Not found |
| `vamtoolbox` import under Python 3.8 | Not found |
| `astra` import under Python 3.13 | Not found |
| `vamtoolbox` import under Python 3.13 | Not found |
| `git` | 2.54.0.windows.1 |
| GitHub CLI `gh` | Not found on PATH |

Because dependencies were not installed, this audit does not verify `astra.use_cuda()`, VAMToolbox importability, projector behavior, or optimizer behavior.

## Known Experimental Metadata

Known values are captured in `configs/ideal_10mm.yaml`:

- Projection wavelength: 405 nm.
- Cylindrical glass vial inner diameter: 10 mm.
- Resin record: `WDS3532:PEGDA:EDAB:CQ = 2:1:0.003:0.003`.
- WDS3532: aliphatic polyurethane acrylate resin from Wuxi Weidusi Electronic Materials.

Unknowns are preserved as `null` or `unknown` rather than filled with guesses.

## Compatibility Risks

- Local default `python` is 3.8.20, while VAMToolbox 3.0.0 documents Python 3.13.
- Python 3.13 exists through the `py` launcher but has neither ASTRA nor VAMToolbox installed.
- CUDA runtime availability is not established by `nvidia-smi`; `nvcc` is absent, and ASTRA CUDA must still be verified after installation.
- Upstream VAMToolbox currently documents Windows-only compatibility.
- Upstream examples commonly use 0 to almost 360 degree angles, while the requested first ideal baseline uses 0 to 180 degrees with 360 angles.
- VAMToolbox's high-level `PrintConfig` contains physical defaults such as vial radius and voxel pitch that do not match this project's known 10 mm vial metadata; project configs must override or avoid those defaults.
- `ProjectionGeometry` accepts physical parameters in centimeters in upstream docstrings, while this project config stores many known dimensions in millimeters. Unit conversion must be explicit.
- Some upstream paths fall back between ASTRA, CUDA and skimage projectors. Tests must record the actual projector selected.
- Export APIs can produce image sequences or video, but this project must not call exported projections printable until calibration and synchronization are verified.

## Unresolved Assumptions

- Ratio basis for the resin record is unknown.
- PEGDA molecular weight is unknown.
- Projector model, lens, projected pixel size, irradiance, grayscale response and spatial uniformity are unknown.
- Vial outer diameter, wall thickness, glass type and refractive index are unknown.
- Resin refractive indices, absorption coefficient and gel dose are unknown.
- Rotation-stage model, RPM, angle timing and synchronization are unknown.
- PSF and optical blur are unknown.
- Whether the future baseline should call VAMToolbox projectors directly or wrap them behind project-specific adapters should be decided after an installed smoke test.

## Phase 0 Stop Condition

Stop after these audit/specification files and the draft PR. Do not install dependencies, modify VAMToolbox core source, or implement the TVP numerical pipeline in Phase 0.