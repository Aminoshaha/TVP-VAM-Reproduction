# VAMToolbox API Map

## Audited Upstream Reference

Repository: `computed-axial-lithography/VAMToolbox`

Commit: `c2757e69f91814bfa85e4240e80dc09441173fe2`

Reported package version: `3.0.0` from `vamtoolbox/__init__.py`

This map records only objects and functions confirmed from upstream source files or examples. It does not invent interfaces.

## Relevant Official Examples

| Need | Confirmed upstream example |
| --- | --- |
| 2D projection and dose reconstruction | `examples/2Doptimization.py`, documented by `docs/_docs/examples/2Doptimization.rst` |
| 3D target volumes | `examples/3Doptimization.py`, documented by `docs/_docs/examples/3Doptimization.rst` |
| STL voxelization | `examples/voxelizestl.py`, documented by `docs/_docs/examples/voxelizestl.rst` |
| Optimization algorithms | `docs/_docs/examples/optimizationcomparison.rst` and `vamtoolbox/optimize.py` |
| Projection export / image sequences | `examples/imagesequence.py`, `docs/_docs/examples/imagesequence.rst`, and `vamtoolbox/imagesequence.py` |
| High-level GUI-facing pipeline | `examples/gui_integration_example.py` and `vamtoolbox/pipeline.py` |

## Public API Surface Confirmed

Top-level exports in `vamtoolbox/__init__.py` include:

- `imagesequence`
- `metrics`
- `geometry`
- `optimize`
- `projectorconstructor`
- `voxelize`
- `util`
- `display`
- `dlp`
- `optimizer`
- `projector`
- `resources`
- `response`
- `displaygrayscale`
- `pipeline`
- `PrintConfig`
- `VAMPipeline`
- `PrintResult`
- `run_print`

`medium` is imported behind an `ImportError` guard because it depends on optional torch-related functionality.

## API Mapping Table

| Required function | Confirmed VAMToolbox object/function | Source file or example | Input shapes | Output shapes | Units or normalization |
| --- | --- | --- | --- | --- | --- |
| Create target from array | `vamtoolbox.geometry.TargetGeometry(target=array, ...)` | `vamtoolbox/geometry.py` | `array` accepted as NumPy array; upstream wraps with `np.atleast_3d` | `target_geo.array`; `Volume` records `(nY, nX, nZ)` for 3D arrays | Values are not automatically physical units. `clip_to_circle=True` by default. |
| Create target from image | `TargetGeometry(imagefilename=..., pixels=...)` | `geometry.py`, `examples/2Doptimization.py` | Square image after optional padding/resizing; grayscale normalized | `target_geo.array` after `np.atleast_3d`; examples call this a 2D target | Image values normalized to 0 to 1, optionally binarized at 0.5. |
| Create target from STL | `TargetGeometry(stlfilename=..., resolution=...)` | `geometry.py`, `examples/3Doptimization.py`, `examples/voxelizestl.py` | STL path, z resolution | Voxel target array plus optional `insert` and `zero_dose` arrays | STL units are mesh units; `resolution` is number of z slices, not necessarily physical printer resolution. |
| Direct STL voxelization | `vamtoolbox.voxelize.Voxelizer` with `addMeshes()` and `voxelize()` | `vamtoolbox/voxelize.py`, `examples/voxelizestl.py` | Mesh dictionary, `layer_thickness`, `voxel_value`, dtype | `voxel_array` | `layer_thickness` uses mesh-file units. |
| Define projection angles and geometry | `vamtoolbox.geometry.ProjectionGeometry(angles, ray_type="parallel", CUDA=...)` | `geometry.py`, `examples/2Doptimization.py`, `examples/3Doptimization.py` | `angles`: 1D NumPy array in degrees | `proj_geo` with `angles`, `n_angles`, `ray_type`, `CUDA` | Upstream examples use `np.linspace(0, 360 - 360 / n, n)`. Project Phase 1 uses 0 to 180 degrees with endpoint false by specification. |
| Build forward/adjoint projector | `vamtoolbox.projectorconstructor.projectorconstructor(target_geo, proj_geo)` | `vamtoolbox/projectorconstructor.py` | `TargetGeometry`, `ProjectionGeometry` | Projector object with `forward(x)` and `backward(b)` | Actual class depends on dimension, `CUDA`, `ray_type`, ASTRA availability and sparse fallback. |
| Forward projection `A x` | Projector protocol method `forward(x)` | `projectorconstructor.py`, `Projector2DParallel.py`, `Projector3DParallel.py` | 2D path: `(N, N)`; 3D path: `(N, N, Z)` | 2D path: `(N, n_angles)`; 3D path: `(N, n_angles, Z)` | Returns line-integral-like projection values in implementation units. Inputs are clipped to circular support in several projectors. |
| Adjoint backprojection `A^T b` | Projector protocol method `backward(b)` | `projectorconstructor.py`, `Projector2DParallel.py`, `Projector3DParallel.py` | 2D sinogram `(N, n_angles)`; 3D sinogram `(N, n_angles, Z)` | 2D reconstruction `(N, N)`; 3D reconstruction `(N, N, Z)` | Unfiltered backprojection in these projector methods; not the same as filtered reconstruction or optimization gradient. |
| Filtered backprojection initialization/optimizer | `vamtoolbox.optimize.Options(method="FBP", ...)` and `vamtoolbox.optimize.optimize(...)` | `vamtoolbox/optimize.py`, `optimizationcomparison.rst` | `TargetGeometry`, `ProjectionGeometry`, `Options` | Usually `Sinogram`, `Reconstruction`, error depending optimizer | Filter choices include names such as `ram-lak`, `hamming`, etc. FBP is distinct from raw `A^T`. |
| CAL optimization | `Options(method="CAL", ...)`; `optimize()` dispatches to `optimizer.CAL.minimizeCAL` | `optimize.py`, `examples/2Doptimization.py` | Target/projection geometry and optimizer options | Optimized sinogram, reconstruction, error | Options include `d_h`, `d_l`, `learning_rate`, `momentum`, `positivity`, `sigmoid`. |
| OSMO optimization | `Options(method="OSMO", ...)`; `optimize()` dispatches to `optimizer.OSMO.minimizeOSMO` | `optimize.py`, `examples/3Doptimization.py` | Target/projection geometry and optimizer options | Optimized sinogram, reconstruction, error | Examples use `d_h=0.85`, `d_l=0.6`, `filter="hamming"`; do not reuse as physical calibration. |
| BCLP optimization | `Options(method="BCLP", ...)`; `optimize()` dispatches to `optimizer.BCLP.minimizeBCLP` or low-memory variant | `optimize.py`, `README.rst` | Target/projection geometry, `Options`, optional response model | Optimized sinogram/reconstruction/error or packaged output depending `output` | Supports response model, band tolerance `eps`, local `weight`, `p`, `q`, `learning_rate`. |
| High-level print preparation | `vamtoolbox.pipeline.PrintConfig`, `VAMPipeline`, `run_print` | `pipeline.py`, `examples/gui_integration_example.py` | JSON-serializable config; STL or array path depending stage | `PrintResult` with `sinogram`, `reconstruction`, `rebinned_sinogram`, timing, config, quality | Contains many physical defaults; must not be used unreviewed for the 10 mm ideal baseline. |
| Hardware detection | `VAMPipeline.detect_hardware()` and `apply_hardware()` | `pipeline.py`; utility implementation under `vamtoolbox/util/hardware.py` | No array input | hardware info/recommendations dict | Used for runtime selection; Phase 0 did not install or execute VAMToolbox. |
| Fan-beam rebinning | `geometry.rebinFanBeam` is referenced by pipeline rebin stage; exact call path needs installed smoke test | `geometry.py`, `pipeline.py` | Parallel sinogram and optical geometry parameters | Rebinned sinogram | Requires projector/vial geometry; not part of Phase 1 ideal 2D baseline unless explicitly approved. |
| Projection image sequence | `vamtoolbox.imagesequence.ImageConfig`, `ImageSeq` | `imagesequence.py`, `examples/imagesequence.py` | Image dimensions `(width, height)` and sinogram object or array | List of image arrays and `.imgseq` save object | Normalization percentile, intensity scaling, bit depth, offsets, flips and rotation. Export is not printable until calibrated. |
| Projection video export | `ImageSeq.saveAsVideo(...)` and `VAMPipeline.save_video(...)` | `imagesequence.py`, `pipeline.py` | Image sequence or pipeline result, rotation velocity, loop/mode settings | Video file | Requires synchronization assumptions; do not label as printable in this project until verified. |
| Save VAMToolbox geometry object | `Volume.save(name)` | `geometry.py` | Geometry volume object | Dill-serialized file with extension such as `.target` or `.sino` | Project should also save explicit NumPy floating arrays and config snapshots, not only dill or PNG artifacts. |

## Shape and Axis Notes

- `Volume` records target/reconstruction arrays as `(nY, nX)` for 2D and `(nY, nX, nZ)` for 3D.
- `Volume` records sinograms as `(nR, nTheta)` for 2D and `(nR, nTheta, nZ)` for 3D.
- The 3D parallel projectors process independent z slices and return sinograms shaped `(detector_pixels, n_angles, z_slices)`.
- VAMToolbox's `TargetGeometry` wraps raw targets with `np.atleast_3d`; the future 2D baseline must smoke-test whether the selected operator path treats a 2D annulus as `(N, N)` or `(N, N, 1)` and document the chosen wrapper behavior.
- Upstream docstrings use centimeters for `projector_pixel_size`, `absorption_coeff` and `container_radius` in `ProjectionGeometry`. This project config uses millimeters for known vial metadata, so conversion must be explicit.

## Forward, Adjoint, Reconstruction and Gradient Boundaries

The project must keep these separate:

- `A`: projector `forward(x)`.
- `A^T`: projector `backward(b)`.
- Filtered reconstruction: FBP-style reconstruction through optimization/FBP routines, not equivalent to raw adjoint.
- Optimization gradient: algorithm-specific update in CAL/OSMO/BCLP, not assumed to equal `A^T` unless verified in source and tests.

## Phase 1 Smoke Tests Required Before Implementation Trust

- Instantiate `TargetGeometry` from a synthetic annulus and verify the exact stored array rank.
- Instantiate `ProjectionGeometry` with `np.linspace(0, 180, 360, endpoint=False)` and confirm angle handling.
- Build the projector through `projectorconstructor()` with explicit `CUDA=False` first, then optional CUDA.
- Verify actual selected projector class and record it in logs.
- Check `A.forward` output shape and dtype.
- Check `A.backward` output shape and dtype.
- Numerically test `<A x, y> ~= <x, A^T y>` for fixed random arrays.
- Confirm projection nonnegativity after project-specific enforcement.
- Confirm no NaN or Inf values in target, sinogram, accumulated dose and metrics.
- Confirm fixed-seed reproducibility for generated test arrays and full Phase 1 outputs.